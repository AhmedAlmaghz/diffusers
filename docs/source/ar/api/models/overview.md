# النماذج

يوفر 🤗 Diffusers نماذج مُدربة مسبقًا لخوارزميات ومكونات شائعة لإنشاء أنظمة انتشار مخصصة. الوظيفة الأساسية للنماذج هي إزالة الضوضاء من عينة الإدخال كما يحددها التوزيع \\(p_{\theta}(x_{t-1}|x_{t})\\).

جميع النماذج مبنية على الفئة الأساسية [`ModelMixin`]، والتي هي [`torch.nn.Module`](https://pytorch.org/docs/stable/generated/torch.nn.Module.html) توفر وظائف أساسية لحفظ وتحميل النماذج، محليًا ومن Hugging Face Hub.

## ModelMixin

[[autodoc]] ModelMixin

## FlaxModelMixin

[[autodoc]] FlaxModelMixin

## PushToHubMixin

[[autodoc]] utils.PushToHubMixin