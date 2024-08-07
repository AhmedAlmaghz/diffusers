# AutoencoderKL

تم تقديم نموذج فك تشفير التباين التلقائي (VAE) مع فقدان KL في [Auto-Encoding Variational Bayes](https://arxiv.org/abs/1312.6114v11) بواسطة Diederik P. Kingma و Max Welling. ويستخدم النموذج في 🤗 Diffusers لتشفير الصور إلى تمثيلات كامنة وفك تشفير التمثيلات الكامنة إلى صور.

الملخص من الورقة هو:

*كيف يمكننا إجراء الاستدلال والتعلم بكفاءة في النماذج الاحتمالية الموجهة، في وجود متغيرات كامنة مستمرة ذات توزيعات لاحقة يصعب التعامل معها، ومجموعات بيانات كبيرة؟ نقدم خوارزمية استدلال وتعلم احتمالي متغير تناسب مجموعات البيانات الكبيرة، وتحت بعض شروط قابلية التمييز البسيطة، تعمل حتى في الحالة التي يصعب التعامل معها. وتتمثل مساهماتنا في شقين. أولاً، نُظهر أن إعادة معلمية الحد الأدنى التفاوتي تؤدي إلى مُقدِّر حد أدنى يمكن تحسينه ببساطة باستخدام طرق التدرج العشوائي القياسية. ثانيًا، نُظهر أنه بالنسبة لمجموعات البيانات المحددة الهوية مع متغيرات كامنة مستمرة لكل نقطة بيانات، يمكن جعل الاستدلال اللاحق فعالًا بشكل خاص عن طريق ملاءمة نموذج استدلال تقريبي (يُطلق عليه أيضًا نموذج التعرف) إلى اللاحق باستخدام مُقدِّر الحد الأدنى المقترح. تنعكس المزايا النظرية في النتائج التجريبية.*

## التحميل من التنسيق الأصلي

افتراضيًا، يجب تحميل [`AutoencoderKL`] باستخدام [`~ModelMixin.from_pretrained`]، ولكن يمكن أيضًا تحميله
من التنسيق الأصلي باستخدام [`FromOriginalVAEMixin.from_single_file`] كما يلي:

```py
from diffusers import AutoencoderKL

url = "https://huggingface.co/stabilityai/sd-vae-ft-mse-original/blob/main/vae-ft-mse-840000-ema-pruned.safetensors"  # يمكن أن يكون أيضًا ملفًا محليًا
model = AutoencoderKL.from_single_file(url)
```

## AutoencoderKL

[[autodoc]] AutoencoderKL

- Decoder
- Encoder
- all

## AutoencoderKLOutput

[[autodoc]] models.autoencoders.autoencoder_kl.AutoencoderKLOutput

## DecoderOutput

[[autodoc]] models.autoencoders.vae.DecoderOutput

## FlaxAutoencoderKL

[[autodoc]] FlaxAutoencoderKL

## FlaxAutoencoderKLOutput

[[autodoc]] models.vae_flax.FlaxAutoencoderKLOutput

## FlaxDecoderOutput

[[autodoc]] models.vae_flax.FlaxDecoderOutput