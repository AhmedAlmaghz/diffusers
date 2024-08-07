# ControlNet with Stable Diffusion XL

تم تقديم ControlNet في [Adding Conditional Control to Text-to-Image Diffusion Models](https://huggingface.co/papers/2302.05543) بواسطة ليفمين جانج، وأني راو، ومانيش أغروالا.

باستخدام نموذج ControlNet، يمكنك توفير صورة تحكم إضافية لتوجيه عملية التوليد والتحكم بها في Stable Diffusion. على سبيل المثال، إذا قدمت خريطة عمق، فإن نموذج ControlNet يقوم بتوليد صورة تحافظ على المعلومات المكانية من خريطة العمق. إنها طريقة أكثر مرونة ودقة للتحكم في عملية توليد الصور.

ملخص الورقة هو:

*نحن نقدم ControlNet، وهو تصميم شبكة عصبية لإضافة عناصر تحكم مكانية إلى النماذج الضخمة المُدربة مسبقًا للانتشار النصي-الصور. تقوم ControlNet بتأمين نماذج الانتشار الضخمة الجاهزة للإنتاج، وتعيد استخدام طبقات الترميز المتعمقة والمتينة المُدربة مسبقًا مع مليارات الصور كعمود فقري قوي لتعلم مجموعة متنوعة من عناصر التحكم الشرطية. يتم توصيل البنية العصبية بـ "التقنيات الصفرية" (طبقات التقنية الصفرية) التي تنمو تدريجياً المعلمات من الصفر وتضمن عدم وجود ضوضاء ضارة يمكن أن تؤثر على الضبط الدقيق. نختبر عناصر تحكم شرطية مختلفة، مثل الحواف والعمق والتجزئة ووضع الإنسان، وما إلى ذلك، مع Stable Diffusion، باستخدام شرط واحد أو أكثر، مع أو بدون موجهات. نُظهر أن تدريب ControlNets قوي مع مجموعات بيانات صغيرة (<50 ألف) وكبيرة (> 1 مليون). تُظهر النتائج المستفيضة أن ControlNet قد تسهل تطبيقات أوسع للتحكم في نماذج انتشار الصور.*

يمكنك العثور على نقاط تفتيش إضافية أصغر لـ Stable Diffusion XL (SDXL) ControlNet من منظمة 🤗 [Diffusers](https://huggingface.co/diffusers) Hub، واطلع على نقاط التفتيش [المُدربة من قبل المجتمع](https://huggingface.co/models?other=stable-diffusion-xl&other=controlnet) على Hub.

<Tip warning={true}>
🧪 العديد من نقاط تفتيش SDXL ControlNet تجريبية، وهناك مجال كبير للتحسين. لا تتردد في فتح [قضية](https://github.com/huggingface/diffusers/issues/new/choose) وترك تعليق لنا حول كيفية تحسينها!
</Tip>

إذا لم تشاهد نقطة تفتيش تهمك، فيمكنك تدريب SDXL ControlNet الخاصة بك باستخدام [سكريبت التدريب](../../../../../examples/controlnet/README_sdxl).

<Tip>
تأكد من الاطلاع على دليل Schedulers [guide](../../using-diffusers/schedulers) لمعرفة كيفية استكشاف المقايضة بين سرعة وجودة الجدولة، وانظر قسم [إعادة استخدام المكونات عبر الأنابيب](../../using-diffusers/loading#reuse-components-across-pipelines) لمعرفة كيفية تحميل المكونات نفسها بكفاءة في أنابيب متعددة.
</Tip>

## StableDiffusionXLControlNetPipeline

[[autodoc]] StableDiffusionXLControlNetPipeline
- all
- __call__

## StableDiffusionXLControlNetImg2ImgPipeline

[[autodoc]] StableDiffusionXLControlNetImg2ImgPipeline
- all
- __call__

## StableDiffusionXLControlNetInpaintPipeline

[[autodoc]] StableDiffusionXLControlNetInpaintPipeline
- all
- __call__

## StableDiffusionPipelineOutput

[[autodoc]] pipelines.stable_diffusion.StableDiffusionPipelineOutput