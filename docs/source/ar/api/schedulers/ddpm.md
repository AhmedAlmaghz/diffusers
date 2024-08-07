# DDPMScheduler

يقترح Ho و Jain و Abbeel في ورقتهم البحثية [Denoising Diffusion Probabilistic Models](https://huggingface.co/papers/2006.11239) (DDPM) نموذجًا قائمًا على الانتشار يحمل نفس الاسم. وفي سياق مكتبة 🤗 Diffusers، يشير DDPM إلى الجدولة التنظيف التلقائي المنفصلة من الورقة وكذلك خط الأنابيب.

مقدمة الورقة هي:

*نعرض نتائج تخليق صور عالية الجودة باستخدام نماذج احتمالية الانتشار، وهي فئة من نماذج المتغيرات الكامنة المستوحاة من اعتبارات الديناميكا الحرارية غير المتوازنة. نحصل على أفضل نتائجنا من خلال التدريب على حد متغير مرجح مصمم وفقًا لصلة جديدة بين النماذج الاحتمالية للانتشار ومطابقة درجات التنظيف مع ديناميكيات Langevin، وتقبل نماذجنا بشكل طبيعي مخطط ضغط فاقد التدريجي يمكن تفسيره على أنه تعميم لفك الترميز السياقي. بالنسبة لمجموعة بيانات CIFAR10 غير المشروطة، نحصل على درجة Inception تبلغ 9.46 ودرجة FID تبلغ 3.17، وهي الأفضل في المجال. وعلى 256x256 LSUN، نحصل على جودة عينة مماثلة لـ ProgressiveGAN. تنفيذنا متاح في [هذا عنوان URL HTTPS](https://github.com/hojonathanho/diffusion).*

## DDPMScheduler

[[autodoc]] DDPMScheduler

## DDPMSchedulerOutput

[[autodoc]] schedulers.scheduling_ddpm.DDPMSchedulerOutput