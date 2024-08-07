# JAX/Flax

🤗 Diffusers يدعم Flax للحصول على استنتاج سريع للغاية على Google TPUs، مثل تلك المتوفرة في Colab أو Kaggle أو Google Cloud Platform. يُظهر لك هذا الدليل كيفية تشغيل الاستدلال باستخدام Stable Diffusion باستخدام JAX/Flax.

قبل أن تبدأ، تأكد من تثبيت المكتبات اللازمة:

```py
# قم بإلغاء التعليق لتثبيت المكتبات اللازمة في Colab
#! pip install -q jax==0.3.25 jaxlib==0.3.25 flax transformers ftfy
#! pip install -q diffusers
```

يجب أيضًا التأكد من استخدامك لخلفية TPU. في حين أن JAX لا يعمل حصريًا على وحدات TPU، فستحصل على أفضل أداء على وحدة TPU لأن كل خادم يحتوي على 8 مسرعات TPU تعمل بالتوازي.

إذا كنت تشغل هذا الدليل في Colab، فحدد *Runtime* في القائمة أعلاه، ثم حدد خيار *Change runtime type*، ثم حدد *TPU* ضمن إعداد *Hardware accelerator*. قم باستيراد JAX والتحقق بسرعة مما إذا كنت تستخدم وحدة TPU:

```python
import jax
import jax.tools.colab_tpu
jax.tools.colab_tpu.setup_tpu()

num_devices = jax.device_count()
device_type = jax.devices()[0].device_kind

print(f"Found {num_devices} JAX devices of type {device_type}.")
assert (
    "TPU" in device_type,
    "Available device is not a TPU, please select TPU from Runtime > Change runtime type > Hardware accelerator"
)
# Found 8 JAX devices of type Cloud TPU.
```


رائع، الآن يمكنك استيراد بقية التبعيات التي ستحتاجها:

```python
import jax.numpy as jnp
from jax import pmap
from flax.jax_utils import replicate
from flax.training.common_utils import shard

from diffusers import FlaxStableDiffusionPipeline
```


## تحميل نموذج

Flax هو إطار عمل وظيفي، لذلك فإن النماذج عديمة الحالة والبارامترات مخزنة خارجها. تحميل خط أنابيب Flax المعلم مسبقًا يعيد *كلاً* من خط الأنابيب ووزن النموذج (أو المعلمات). في هذا الدليل، ستستخدم `bfloat16`، وهو نوع نصف عائم أكثر كفاءة تدعمه وحدات TPU (يمكنك أيضًا استخدام `float32` للدقة الكاملة إذا كنت تريد).

```python
dtype = jnp.bfloat16
pipeline, params = FlaxStableDiffusionPipeline.from_pretrained(
    "CompVis/stable-diffusion-v1-4",
    revision="bf16",
    dtype=dtype,
)
```

## الاستدلال

عادةً ما تحتوي وحدات TPU على 8 أجهزة تعمل بالتوازي، لذا دعنا نستخدم نفس المحفز لكل جهاز. وهذا يعني أنه يمكنك إجراء الاستدلال على 8 أجهزة في نفس الوقت، مع قيام كل جهاز بتوليد صورة واحدة. ونتيجة لذلك، ستحصل على 8 صور في نفس الوقت الذي يستغرقه الشريحة لتوليد صورة واحدة!

<Tip>
تعرف على المزيد من التفاصيل في قسم [كيف تعمل الموازاة؟](#how-does-parallelization-work).
</Tip>

بعد استنساخ المحفز، احصل على معرفات النص المعلم بالاتصال بوظيفة `prepare_inputs` على خط الأنابيب. تم تعيين طول النص المعلم على 77 رمزًا كما هو مطلوب من خلال تكوين نموذج CLIP النصي الأساسي.

```python
prompt = "A cinematic film still of Morgan Freeman starring as Jimi Hendrix, portrait, 40mm lens, shallow depth of field, close up, split lighting, cinematic"
prompt = [prompt] * jax.device_count()
prompt_ids = pipeline.prepare_inputs(prompt)
prompt_ids.shape
# (8, 77)
```

يجب تكرار معلمات النموذج والمدخلات عبر الأجهزة المتوازية 8. يتم تكرار قاموس المعلمات باستخدام [`flax.jax_utils.replicate`](https://flax.readthedocs.io/en/latest/api_reference/flax.jax_utils.html#flax.jax_utils.replicate) الذي يتنقل في القاموس ويغير شكل الأوزان بحيث يتم تكرارها 8 مرات. يتم تكرار المصفوفات باستخدام `shard`.

```python
# parameters
p_params = replicate(params)

# arrays
prompt_ids = shard(prompt_ids)
prompt_ids.shape
# (8, 1, 77)
```

يعني هذا الشكل أن كل جهاز من الأجهزة الثمانية يستقبل كإدخال مصفوفة `jnp` ذات الشكل `(1، 77)`، حيث `1` هو حجم الدفعة لكل جهاز. في وحدات TPU ذات الذاكرة الكافية، يمكن أن يكون حجم الدفعة أكبر من `1` إذا كنت تريد إنشاء صور متعددة (للشريحة) في نفس الوقت.

بعد ذلك، قم بإنشاء مولد رقم عشوائي لتمريره إلى وظيفة التوليد. هذا هو الإجراء القياسي في Flax، والذي يكون جادًا ومتعجرفًا بشأن الأرقام العشوائية. من المتوقع أن تتلقى جميع الوظائف التي تتعامل مع الأرقام العشوائية مولدًا لضمان إمكانية إعادة الإنتاج، حتى عند التدريب عبر أجهزة موزعة متعددة.

تستخدم وظيفة المساعدة أدناه بذرة لتهيئة مولد رقم عشوائي. طالما أنك تستخدم نفس البذرة، فستحصل على نفس النتائج بالضبط. لا تتردد في استخدام بذور مختلفة عند استكشاف النتائج لاحقًا في الدليل.

```python
def create_key(seed=0):
    return jax.random.PRNGKey(seed)
```

يتم تقسيم وظيفة المساعدة، أو `rng`، 8 مرات بحيث يتلقى كل جهاز مولدًا مختلفًا وينشئ صورة مختلفة.

```python
rng = create_key(0)
rng = jax.random.split(rng, jax.device_count())
```

لاستغلال سرعة JAX المحسنة على وحدة TPU، قم بتمرير `jit=True` إلى خط الأنابيب لتجميع رمز JAX إلى تمثيل فعال وضمان تشغيل النموذج بالتوازي عبر الأجهزة الثمانية.

<Tip warning={true}>
يجب التأكد من أن جميع مدخلاتك لها نفس الشكل في المكالمات اللاحقة، وإلا فسيتعين على JAX إعادة تجميع الرمز وهو أبطأ.
</Tip>

تستغرق أول عملية استدلال وقتًا أطول لأنها تحتاج إلى تجميع الرمز، ولكن المكالمات اللاحقة (حتى مع مدخلات مختلفة) تكون أسرع بكثير. على سبيل المثال، استغرق الأمر أكثر من دقيقة لتجميع على وحدة TPU v2-8، لكنه يستغرق حوالي **7 ثوانٍ** في عملية استدلال مستقبلية!

```py
%%time
images = pipeline(prompt_ids, p_params, rng, jit=True)[0]

# CPU times: user 56.2 s, sys: 42.5 s, total: 1min 38s
# Wall time: 1min 29s
```

يكون للمصفوفة التي تمت إعادتها الشكل `(8، 1، 512، 512، 3)` والذي يجب إعادة تشكيله لإزالة البعد الثاني والحصول على 8 صور بحجم `512 × 512 × 3`. بعد ذلك، يمكنك استخدام وظيفة [`~utils.numpy_to_pil`] لتحويل المصفوفات إلى صور.

```python
from diffusers.utils import make_image_grid

images = images.reshape((images.shape[0] * images.shape[1],) + images.shape[-3:])
images = pipeline.numpy_to_pil(images)
make_image_grid(images, rows=2, cols=4)
```

![img](https://huggingface.co/datasets/YiYiXu/test-doc-assets/resolve/main/stable_diffusion_jax_how_to_cell_38_output_0.jpeg)

## استخدام محفزات مختلفة

أنت لست مضطرًا لاستخدام نفس المحفز على جميع الأجهزة. على سبيل المثال، لإنشاء 8 محفزات مختلفة:

```python
prompts = [
    "Labrador in the style of Hokusai",
    "Painting of a squirrel skating in New York",
    "HAL-9000 in the style of Van Gogh",
    "Times Square under water, with fish and a dolphin swimming around",
    "Ancient Roman fresco showing a man working on his laptop",
    "Close-up photograph of young black woman against urban background, high quality, bokeh",
    "Armchair in the shape of an avocado",
    "Clown astronaut in space, with Earth in the background",
]

prompt_ids = pipeline.prepare_inputs(prompts)
prompt_ids = shard(prompt_ids)

images = pipeline(prompt_ids, p_params, rng, jit=True).images
images = images.reshape((images.shape[0] * images.shape[1],) + images.shape[-3:])
images = pipeline.numpy_to_pil(images)

make_image_grid(images, 2, 4)
```

![img](https://huggingface.co/datasets/YiYiXu/test-doc-assets/resolve/main/stable_diffusion_jax_how_to_cell_43_output_0.jpeg)

## كيف تعمل الموازاة؟ How does parallelization work?

يقوم خط أنابيب Flax في 🤗 Diffusers بتجميع النموذج وتشغيله تلقائيًا بالتوازي على جميع الأجهزة المتاحة. دعنا نلقي نظرة فاحصة على كيفية عمل هذه العملية.

يمكن إجراء موازاة JAX بعدة طرق. أسهلها يدور حول استخدام وظيفة [`jax.pmap`](https://jax.readthedocs.io/en/latest/_autosummary/jax.pmap.html) لتحقيق موازاة البرنامج الفردي والبيانات المتعددة (SPMD). وهذا يعني تشغيل عدة نسخ من نفس الرمز، كل منها على مدخلات بيانات مختلفة. من الممكن اتباع نهج أكثر تقدمًا، ويمكنك الانتقال إلى وثائق JAX [documentation](https://jax.readthedocs.io/en/latest/index.html) لاستكشاف هذا الموضوع بمزيد من التفصيل إذا كنت مهتمًا!

`jax.pmap` يفعل شيئين:

1. يقوم بتجميع (أو "jit") الرمز الذي يشبه `jax.jit()`. لا يحدث هذا عند استدعاء `pmap`، ولكن فقط في المرة الأولى التي يتم فيها استدعاء الدالة `pmap`ped.
2. يضمن تشغيل الرمز المجمع بالتوازي على جميع الأجهزة المتاحة.

لإثبات ذلك، قم بالاتصال بـ `pmap` على طريقة `_generate` في خط الأنابيب (هذه طريقة خاصة تقوم بتوليد الصور وقد يتم إعادة تسميتها أو إزالتها في الإصدارات المستقبلية من 🤗 Diffusers):

```python
p_generate = pmap(pipeline._generate)
```

بعد استدعاء `pmap`، ستعمل الوظيفة المحضرة `p_generate` على:

1. قم بعمل نسخة من الوظيفة الأساسية، `pipeline._generate`، على كل جهاز.
2. إرسال كل جهاز جزء مختلف من حجج الإدخال (هذا هو السبب في أنه من الضروري استدعاء وظيفة *shard*). في هذه الحالة، يكون لـ `prompt_ids` الشكل `(8، 1، 77، 768)` لذا يتم تقسيم المصفوفة إلى 8 ويستقبل كل نسخة من `_generate` إدخالًا بشكل `(1، 77، 768)`.

أهم شيء يجب الانتباه إليه هنا هو حجم الدفعة (1 في هذا المثال) وأبعاد الإدخال التي لها معنى لرمزك. لا يلزمك تغيير أي شيء آخر لجعل الرمز يعمل بالتوازي.

تستغرق أول مكالمة لخط الأنابيب وقتًا أطول، ولكن المكالمات اللاحقة تكون أسرع بكثير. يتم استخدام وظيفة `block_until_ready` لقياس وقت الاستدلال بشكل صحيح لأن JAX يستخدم الإرسال غير المتزامن ويعيد التحكم إلى حلقة Python بمجرد أن يتمكن من ذلك. لا تحتاج إلى استخدام ذلك في رمزك؛ يحدث الحظر تلقائيًا عندما تريد استخدام نتيجة حساب لم يتم تحقيقه بعد.

```py
%%time
images = p_generate(prompt_ids, p_params, rng)
images = images.block_until_ready()

# CPU times: user 1min 15s, sys: 18.2 s, total: 1min 34s
# Wall time: 1min 15s
```

تحقق من أبعاد الصورة لمعرفة ما إذا كانت صحيحة:

```python
images.shape
# (8، 1، 512، 512، 3)
```

## الموارد

لمعرفة المزيد حول كيفية عمل JAX مع Stable Diffusion، قد تكون مهتمًا بقراءة:

* [تسريع استدلال Stable Diffusion XL باستخدام JAX على Cloud TPU v5e](https://hf.co/blog/sdxl_jax)