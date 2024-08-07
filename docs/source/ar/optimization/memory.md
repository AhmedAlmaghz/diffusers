# تقليل استخدام الذاكرة 

تشكل الكمية الكبيرة من الذاكرة المطلوبة عائقًا أمام استخدام نماذج الانتشار. وللتغلب على هذا التحدي، هناك العديد من التقنيات لتقليل الذاكرة التي يمكنك استخدامها لتشغيل حتى أكبر النماذج على وحدات معالجة الرسوميات (GPU) من الفئة المجانية أو المخصصة للمستهلكين. ويمكن حتى الجمع بين بعض هذه التقنيات لزيادة تقليل استخدام الذاكرة. 

في كثير من الحالات، يؤدي التحسين من أجل الذاكرة أو السرعة إلى تحسين الأداء في الجانب الآخر، لذلك يجب عليك محاولة التحسين لكليهما كلما استطعت. ويركز هذا الدليل على تقليل استخدام الذاكرة إلى الحد الأدنى، ولكن يمكنك أيضًا معرفة المزيد حول كيفية تسريع الاستنتاج. 

تم الحصول على النتائج أدناه من خلال إنشاء صورة واحدة بحجم 512x512 من موجه "صورة لرائد فضاء يركب حصانًا على المريخ" مع 50 خطوة من DDIM على Nvidia Titan RTX، مما يوضح التسريع الذي يمكن توقعه نتيجة لانخفاض استهلاك الذاكرة. 

|                  | الكمون | التسريع |
| ---------------- | ------- | ------- |
| الأصلي         | 9.50 ثانية   | 1x      |
| FP16             | 3.61 ثانية   | 2.63x   |
| القنوات الأخيرة    | 3.30 ثانية   | 2.88x   |
| تتبع UNet      | 3.21 ثانية   | 2.96x   |
| انتباه فعال الذاكرة | 2.63 ثانية  | 3.61x   |

## VAE المقطوع

تمكّن VAE المقطوعة من فك تشفير دفعات كبيرة من الصور ذات ذاكرة VRAM محدودة أو دفعات تحتوي على 32 صورة أو أكثر عن طريق فك تشفير دفعات المخفونات صورة واحدة في كل مرة. من المحتمل أن ترغب في اقتران هذا مع [`~ModelMixin.enable_xformers_memory_efficient_attention`] لتقليل استخدام الذاكرة بشكل أكبر إذا كان لديك xFormers مثبتًا. 

لاستخدام VAE المقطوعة، اتصل بـ [`~StableDiffusionPipeline.enable_vae_slicing`] على خط الأنابيب الخاص بك قبل الاستدلال: 

```python
import torch
from diffusers import StableDiffusionPipeline

pipe = StableDiffusionPipeline.from_pretrained(
"runwayml/stable-diffusion-v1-5",
torch_dtype=torch.float16,
use_safetensors=True,
)
pipe = pipe.to("cuda")

prompt = "a photo of an astronaut riding a horse on mars"
pipe.enable_vae_slicing()
#pipe.enable_xformers_memory_efficient_attention()
images = pipe([prompt] * 32).images
``` 

قد تشاهد زيادة طفيفة في الأداء في فك تشفير VAE على دفعات متعددة الصور، ولا ينبغي أن يكون هناك أي تأثير على الأداء في دفعات الصور الفردية. 

## معالجة VAE المبلطة

تمكّن معالجة VAE المبلطة أيضًا من العمل مع صور كبيرة ذات ذاكرة VRAM محدودة (على سبيل المثال، إنشاء صور 4K على 8 جيجابايت من ذاكرة VRAM) عن طريق تقسيم الصورة إلى بلاطات متداخلة، وفك تشفير البلاطات، ثم مزج المخرجات معًا لتكوين الصورة النهائية. يجب أيضًا استخدام VAE المبلطة مع [`~ModelMixin.enable_xformers_memory_efficient_attention`] لتقليل استخدام الذاكرة بشكل أكبر إذا كان لديك xFormers مثبتًا. 

لاستخدام معالجة VAE المبلطة، اتصل بـ [`~StableDiffusionPipeline.enable_vae_tiling`] على خط الأنابيب الخاص بك قبل الاستدلال: 

```python
import torch
from diffusers import StableDiffusionPipeline, UniPCMultistepScheduler

pipe = StableDiffusionPipeline.from_pretrained(
"runwayml/stable-diffusion-v1-5",
torch_dtype=torch.float16,
use_safetensors=True,
)
pipe.scheduler = UniPCMultistepScheduler.from_config(pipe.scheduler.config)
pipe = pipe.to("cuda")
prompt = "a beautiful landscape photograph"
pipe.enable_vae_tiling()
#pipe.enable_xformers_memory_efficient_attention()

image = pipe([prompt], width=3840, height=2224, num_inference_steps=20).images[0]
``` 

توجد بعض الاختلافات في التدرج بين البلاط والبلاط في الصورة الناتجة لأن البلاطات يتم فك تشفيرها بشكل منفصل، ولكن لا يجب أن ترى أي درزات حادة وواضحة بين البلاطات. يتم إيقاف التبليط للصور التي يبلغ حجمها 512x512 أو أصغر. 

## تفريغ الذاكرة إلى وحدة المعالجة المركزية

يمكن أيضًا توفير الذاكرة عن طريق تفريغ الأوزان إلى وحدة المعالجة المركزية (CPU) وتحميلها فقط على وحدة معالجة الرسومات (GPU) عند إجراء تمرير للأمام. وغالبًا ما يمكن أن تقلل هذه التقنية من استهلاك الذاكرة إلى أقل من 3 جيجابايت. 

لأداء التفريغ إلى وحدة المعالجة المركزية، اتصل بـ [`~StableDiffusionPipeline.enable_sequential_cpu_offload`]: 

```Python
import torch
from diffusers import StableDiffusionPipeline

pipe = StableDiffusionPipeline.from_pretrained(
"runwayml/stable-diffusion-v1-5",
torch_dtype=torch.float16,
use_safetensors=True,
)

prompt = "a photo of an astronaut riding a horse on mars"
pipe.enable_sequential_cpu_offload()
image = pipe(prompt).images[0]
``` 

يعمل تفريغ الذاكرة إلى وحدة المعالجة المركزية على الوحدات الفرعية بدلاً من النماذج الكاملة. هذه هي الطريقة المثلى لتقليل استهلاك الذاكرة، ولكن الاستدلال أبطأ بكثير بسبب الطبيعة المتكررة لعملية الانتشار. تقوم مكونات UNet في خط الأنابيب بتشغيل عدة مرات (مثل عدد `num_inference_steps`)؛ في كل مرة، يتم تحميل الوحدات الفرعية لـ UNet وتحميلها بشكل متسلسل حسب الحاجة، مما يؤدي إلى عدد كبير من عمليات نقل الذاكرة. 

ضع في اعتبارك استخدام [تفريغ الذاكرة إلى النموذج](#model-offloading) إذا كنت تريد التحسين من أجل السرعة لأنه أسرع بكثير. المقايضة هي أن وفورات الذاكرة الخاصة بك لن تكون كبيرة. 

<Tip> 

عند استخدام [`~StableDiffusionPipeline.enable_sequential_cpu_offload`]، لا تنقل خط الأنابيب إلى CUDA مسبقًا وإلا ستكون الزيادة في استهلاك الذاكرة طفيفة فقط (راجع هذا [القضية](https://github.com/huggingface/diffusers/issues/1934) لمزيد من المعلومات). 

[`~StableDiffusionPipeline.enable_sequential_cpu_offload`] هي عملية ذات حالة تقوم بتثبيت الخطافات على النماذج. 

</Tip> 

## تفريغ الذاكرة إلى النموذج

<Tip> 

يتطلب تفريغ الذاكرة إلى النموذج 🤗 Accelerate الإصدار 0.17.0 أو أعلى. 

</Tip> 

يحافظ [تفريغ الذاكرة إلى وحدة المعالجة المركزية بشكل متسلسل](#cpu-offloading) على الكثير من الذاكرة ولكنه يجعل الاستدلال أبطأ لأن الوحدات الفرعية يتم نقلها إلى وحدة معالجة الرسومات (GPU) حسب الحاجة، ويتم إرجاعها على الفور إلى وحدة المعالجة المركزية (CPU) عندما يتم تشغيل وحدة جديدة. 

والتفريغ إلى النموذج بالكامل هو بديل ينقل النماذج الكاملة إلى وحدة معالجة الرسومات (GPU)، بدلاً من التعامل مع الوحدات الفرعية لكل نموذج. هناك تأثير ضئيل على وقت الاستدلال (مقارنة بنقل خط الأنابيب إلى `cuda`)، ولا يزال يوفر بعض وفورات الذاكرة. 

أثناء تفريغ الذاكرة إلى النموذج، يتم وضع أحد المكونات الرئيسية لخط الأنابيب فقط (عادةً ما يكون مشفر النص، و UNet و VAE) على وحدة معالجة الرسومات (GPU) بينما تنتظر المكونات الأخرى في وحدة المعالجة المركزية (CPU). تظل المكونات مثل UNet التي تعمل لعدة تكرارات على وحدة معالجة الرسومات (GPU) حتى تصبح غير مطلوبة. 

قم بتمكين تفريغ الذاكرة إلى النموذج عن طريق الاتصال بـ [`~StableDiffusionPipeline.enable_model_cpu_offload`] على خط الأنابيب: 

```Python
import torch
from diffusers import StableDiffusionPipeline

pipe = StableDiffusionPipeline.from_pretrained(
"runwayml/stable-diffusion-v1-5",
torch_dtype=torch.float16,
use_safetensors=True,
)

prompt = "a photo of an astronaut riding a horse on mars"
pipe.enable_model_cpu_offload()
image = pipe(prompt).images[0]
``` 

<Tip> 

لإلغاء تحميل النماذج بشكل صحيح بعد استدعائها، من الضروري تشغيل خط الأنابيب بالكامل ويجب استدعاء النماذج في الترتيب المتوقع لخط الأنابيب. توخ الحذر إذا تمت إعادة استخدام النماذج خارج سياق خط الأنابيب بعد تثبيت الخطافات. راجع [إزالة الخطافات](https://huggingface.co/docs/accelerate/en/package_reference/big_modeling#accelerate.hooks.remove_hook_from_module) لمزيد من المعلومات. 

[`~StableDiffusionPipeline.enable_model_cpu_offload`] هي عملية ذات حالة تقوم بتثبيت الخطافات على النماذج والحالة على خط الأنابيب. 

</Tip> 

## تنسيق الذاكرة للقنوات الأخيرة

تنسيق الذاكرة للقنوات الأخيرة هو طريقة بديلة لترتيب نسيج NCHW في الذاكرة للحفاظ على ترتيب الأبعاد. يتم ترتيب نسيج القنوات الأخيرة بطريقة تجعل القنوات هي البعد الأكثر كثافة (تخزين الصور بكسل لكل بكسل). نظرًا لأن المشغلين لا يدعمون جميع تنسيق القنوات الأخير حاليًا، فقد يؤدي ذلك إلى أداء أسوأ، ولكن يجب عليك مع ذلك المحاولة ومعرفة ما إذا كان يعمل لنموذجك. 

على سبيل المثال، لتعيين UNet في خط الأنابيب لاستخدام تنسيق القنوات الأخير: 

```python
print(pipe.unet.conv_out.state_dict()["weight"].stride())  # (2880, 9, 3, 1)
pipe.unet.to(memory_format=torch.channels_last)  # عملية في المكان
print(
pipe.unet.conv_out.state_dict()["weight"].stride()
)  # (2880, 1, 960، 320) وجود خطوة 1 للبعد 2 يثبت أنه يعمل
```

## التتبع 

يقوم التتبع بتشغيل نسيج إدخال مثال عبر النموذج ويحتفظ بالعمليات التي يتم إجراؤها عليه أثناء انتقال الإدخال عبر طبقات النموذج. يتم تحسين الدالة القابلة للتنفيذ أو `ScriptFunction` التي يتم إرجاعها باستخدام التجميع المسبق في الوقت المناسب. 

لتتبع UNet: 

```python
import time
import torch
from diffusers import StableDiffusionPipeline
import functools

# إيقاف تشغيل تدرج الشعلة
torch.set_grad_enabled(False)

# تعيين المتغيرات
n_experiments = 2
unet_runs_per_experiment = 50


# تحميل المدخلات
def generate_inputs():
sample = torch.randn((2, 4, 64, 64), device="cuda"، dtype=torch.float16)
timestep = torch.rand(1, device="cuda"، dtype=torch.float16) * 999
encoder_hidden_states = torch.randn((2, 77, 768), device="cuda"، dtype=torch.float16)
return sample, timestep, encoder_hidden_states


pipe = StableDiffusionPipeline.from_pretrained(
"runwayml/stable-diffusion-v1-5"،
torch_dtype=torch.float16,
use_safetensors=True,
).to("cuda")
unet = pipe.unet
unet.eval()
unet.to(memory_format=torch.channels_last)  # استخدام تنسيق الذاكرة للقنوات الأخيرة
unet.forward = functools.partial(unet.forward, return_dict=False)  # تعيين return_dict=False كافتراضي

# الإحماء
for _ in range(3):
with torch.inference_mode():
inputs = generate_inputs()
orig_output = unet(*inputs)

# التتبع
print("التتبع..")
unet_traced = torch.jit.trace(unet, inputs)
unet_traced.eval()
print("انتهى التتبع")


# الإحماء وتحسين الرسم البياني
for _ in range(5):
with torch.inference_mode():
inputs = generate_inputs()
orig_output = unet_traced(*inputs)


# المعايرة
with torch.inference_mode():
for _ in range(n_experiments):
torch.cuda.synchronize()
start_time = time.time()
for _ in range(unet_runs_per_experiment):
orig_output = unet_traced(*inputs)
torch.cuda.synchronize()
print(f"استغرق الاستدلال المقطوع لـ UNet {time.time() - start_time:.2f} ثانية")
for _ in range(n_experiments):
torch.cuda.synchronize()
start_time = time.time()
for _ in range(unet_runs_per_experiment):
orig_output = unet(*inputs)
torch.cuda.synchronize()
print(f"استغرق الاستدلال لـ UNet {time.time() - start_time:.2f} ثانية")

# حفظ النموذج
unet_traced.save("unet_traced.pt")
``` 

استبدل سمة `unet` لخط الأنابيب بالنموذج المقطوع: 

```python
from diffusers import StableDiffusionPipeline
import torch
from dataclasses import dataclass


@dataclass
class UNet2DConditionOutput:
sample: torch.Tensor


pipe = StableDiffusionPipeline.from_pretrained(
"runwayml/stable-diffusion-v1-5"،
torch_dtype=torch.float16,
use_safetensors=True,
).to("cuda")

# استخدام UNet المقطوع
unet_traced = torch.jit.load("unet_traced.pt")


# حذف pipe.unet
class TracedUNet(torch.nn.Module):
def __init__(self):
super().__init__()
self.in_channels = pipe.unet.config.in_channels
self.device = pipe.unet.device

def forward(self، latent_model_input، t، encoder_hidden_states):
sample = unet_traced(latent_model_input، t، encoder_hidden_states)[0]
return UNet2DConditionOutput(sample=sample)


pipe.unet = TracedUNet()

with torch.inference_mode():
image = pipe([prompt] * 1, num_inference_steps=50).images[0]
```

## انتباه فعال الذاكرة 

أدى العمل الأخير حول تحسين عرض النطاق الترددي في كتلة الاهتمام إلى تسريع كبير وتخفيضات في استخدام ذاكرة وحدة معالجة الرسومات (GPU). وآخر نوع من انتباه فعال الذاكرة هو [Flash Attention](https://arxiv.org/abs/2205.14135) (يمكنك الاطلاع على الكود الأصلي في [HazyResearch/flash-attention](https://github.com/HazyResearch/flash-attention)). 

<Tip> 

إذا كان لديك PyTorch >= 2.0 مثبتًا، فلا ينبغي أن تتوقع تسريع الاستدلال عند تمكين `xformers`. 

</Tip> 

لاستخدام Flash Attention، قم بتثبيت ما يلي: 

- Python > 1.12
- CUDA متاح
- [xFormers](xformers) 

ثم اتصل بـ [`~ModelMixin.enable_xformers_memory_efficient_attention`] على خط الأنابيب: