لمزيد من المعلومات، راجع [رخصة Apache، الإصدار 2.0](http://www.apache.org/licenses/LICENSE-2.0)

# ControlNet-XS with Stable Diffusion XL

تم تقديم ControlNet-XS في [ControlNet-XS](https://vislearn.github.io/ControlNet-XS/) بواسطة Denis Zavadski وCarsten Rother. ويستند إلى الملاحظة التي مفادها أن نموذج التحكم في [ControlNet](https://huggingface.co/papers/2302.05543) الأصلي يمكن أن يكون أصغر بكثير ولا يزال ينتج نتائج جيدة.

وعلى غرار نموذج ControlNet الأصلي، يمكنك توفير صورة تحكم إضافية لتكييف عملية إنشاء Stable Diffusion والتحكم فيها. على سبيل المثال، إذا قدمت خريطة عمق، فإن نموذج ControlNet سينشئ صورة تحافظ على المعلومات المكانية من خريطة العمق. إنها طريقة أكثر مرونة ودقة للتحكم في عملية إنشاء الصور.

ينشئ ControlNet-XS صورا بجودة مماثلة لتلك التي ينشئها ControlNet العادي، ولكنه أسرع بنسبة 20-25% (انظر [معيار القياس](https://github.com/UmerHA/controlnet-xs-benchmark/blob/main/Speed%20Benchmark.ipynb)) ويستخدم ذاكرة أقل بنسبة 45%.

فيما يلي نظرة عامة من [صفحة المشروع](https://vislearn.github.io/ControlNet-XS/):

> *مع زيادة قدرات الحوسبة، تبدو بنيات النماذج الحالية وكأنها تتبع الاتجاه المتمثل في توسيع جميع المكونات دون التحقق من ضرورة القيام بذلك. في هذا المشروع، نبحث في حجم وتصميم بنية ControlNet [Zhang et al.، 2023] للتحكم في عملية إنشاء الصور باستخدام النماذج المستندة إلى Stable Diffusion. ونظهر أن بنية جديدة تحتوي على عدد قليل من المعلمات مثل 1% من معلمات النموذج الأساسي تحقق نتائج متقدمة للغاية، أفضل بكثير من ControlNet من حيث درجة FID. ولهذا نطلق عليه اسم ControlNet-XS. ونقدم الكود للتحكم في StableDiffusion-XL [Podell et al.، 2023] (النموذج B، 48 مليون معلمة) وStableDiffusion 2.1 [Rombach et al. 2022] (النموذج B، 14 مليون معلمة)، وكلها بموجب ترخيص openrail.*

تمت المساهمة بهذا النموذج من قبل [UmerHA](https://twitter.com/UmerHAdil). ❤️

<Tip warning={true}>
🧪 العديد من نقاط تفتيش SDXL ControlNet تجريبية، وهناك مجال كبير للتحسين. لا تتردد في فتح [قضية](https://github.com/huggingface/diffusers/issues/new/choose) وترك تعليق لنا حول كيفية تحسينها!
</Tip>

<Tip>
تأكد من مراجعة الدليل الخاص بـ [Schedulers](../../using-diffusers/schedulers) لمعرفة كيفية استكشاف المقايضة بين سرعة وجودة الجدول، وقسم [إعادة استخدام المكونات عبر الأنابيب](../../using-diffusers/loading#reuse-components-across-pipelines) لمعرفة كيفية تحميل المكونات نفسها بكفاءة في أنابيب متعددة.
</Tip>

## StableDiffusionXLControlNetXSPipeline

[[autodoc]] StableDiffusionXLControlNetXSPipeline

- all

- __call__

## StableDiffusionPipelineOutput

[[autodoc]] pipelines.stable_diffusion.StableDiffusionPipelineOutput