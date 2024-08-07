# unCLIP

[Hierarchical Text-Conditional Image Generation with CLIP Latents](https://huggingface.co/papers/2204.06125) هي ورقة بحثية من تأليف أديتيا راملش، وبرافولا دهاريوال، وأليكس نيكول، وكيسي تشو، ومارك تشين. ونموذج unCLIP في 🤗 Diffusers مأخوذ من [karlo](https://github.com/kakaobrain/karlo) الخاص بـ kakaobrain.

ملخص الورقة البحثية هو كما يلي:

*أظهرت النماذج التمييزية مثل CLIP قدرتها على تعلم تمثيلات قوية للصور التي تلتقط كلًا من الدلالات والأسلوب. وللاستفادة من هذه التمثيلات في توليد الصور، نقترح نموذجًا مكونًا من مرحلتين: نموذج سابق يولد تضمين صورة CLIP معطى تعليق نصي، وفك تشفير يقوم بتوليد صورة مشروطة بتضمين الصورة. ونظهر أن توليد تمثيلات الصور بشكل صريح يحسن تنوع الصور مع فقدان طفيف في الواقعية التشابه في التعليق. ويمكن أيضًا لفك التشفير الخاص بنا المشروط بتمثيلات الصور أن ينتج متغيرات لصورة ما مع الحفاظ على دلالاتها وأسلوبها، مع تغيير التفاصيل غير الأساسية الغائبة عن تمثيل الصورة. علاوة على ذلك، يمكّن مساحة التضمين المشتركة لـ CLIP من إجراء تعديلات على الصور الموجهة باللغة بطريقة Zero-shot. ونستخدم نماذج الانتشار للنموذج فك التشفير، ونجري تجارب مع كل من النماذج التلقائية والنماذج الانتشارية للنموذج السابق، ونجد أن الأخير أكثر كفاءة من الناحية الحسابية وينتج عينات ذات جودة أعلى.*

يمكنك العثور على إعادة إنشاء DALL-E 2 من lucidrains على [lucidrains/DALLE2-pytorch](https://github.com/lucidrains/DALLE2-pytorch).

<Tip>
تأكد من الاطلاع على دليل Schedulers [guide](../../using-diffusers/schedulers) لمعرفة كيفية استكشاف المقايضة بين سرعة المجدول والجودة، وقسم [إعادة استخدام المكونات عبر الأنابيب](../../using-diffusers/loading#reuse-components-across-pipelines) لمعرفة كيفية تحميل المكونات نفسها بكفاءة في أنابيب متعددة.
</Tip>

## UnCLIPPipeline

[[autodoc]] UnCLIPPipeline

- all

- __call__

## UnCLIPImageVariationPipeline

[[autodoc]] UnCLIPImageVariationPipeline

- all

- __call__

## ImagePipelineOutput

[[autodoc]] pipelines.ImagePipelineOutput