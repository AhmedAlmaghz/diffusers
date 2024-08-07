# DDPM

يقترح Jonathan Ho و Ajay Jain و Pieter Abbeel في ورقتهم البحثية [Denoising Diffusion Probabilistic Models](https://huggingface.co/papers/2006.11239) (DDPM) نموذجًا قائمًا على الانتشار يحمل نفس الاسم. في مكتبة 🤗 Diffusers، يشير DDPM إلى *مخطط إزالة التشويش المتقطع* من الورقة وكذلك خط الأنابيب.

مقدمة الورقة هي:

*نحن نقدم نتائج تخليق صور عالية الجودة باستخدام نماذج الانتشار الاحتمالية، وهي فئة من نماذج المتغيرات الكامنة المستوحاة من اعتبارات الديناميكا الحرارية اللادولابية. نحصل على أفضل نتائجنا من خلال التدريب على حد متغير مرجح مصمم وفقًا لصلة جديدة بين نماذج الانتشار الاحتمالية ومطابقة درجات إزالة التشويش مع ديناميكيات Langevin، وتوفر نماذجنا بشكل طبيعي مخططًا ضياعًا تدريجيًا يمكن تفسيره على أنه تعميم لفك الترميز السياقي. بالنسبة لمجموعة بيانات CIFAR10 غير المشروطة، نحصل على درجة Inception تبلغ 9.46 ودرجة FID رائدة في المجال تبلغ 3.17. على 256x256 LSUN، نحصل على جودة عينة مماثلة لـ ProgressiveGAN.*

يمكن العثور على الكود الأصلي في [hohonathanho/diffusion](https://github.com/hojonathanho/diffusion).

<Tip>
تأكد من الاطلاع على دليل Schedulers [guide](../../using-diffusers/schedulers) لمعرفة كيفية استكشاف المقايضة بين سرعة المخطط وجودته، وراجع قسم [إعادة استخدام المكونات عبر الأنابيب](../../using-diffusers/loading#reuse-components-across-pipelines) لمعرفة كيفية تحميل المكونات نفسها بكفاءة في عدة أنابيب.
</Tip>

# DDPMPipeline

[[autodoc]] DDPMPipeline
- all
- __call__

## ImagePipelineOutput

[[autodoc]] pipelines.ImagePipelineOutput