أي حاجة بتعتمد عليها اسمها **Dependency** فبالتالي أي Class أو Interface بتعتمد على Class أو Interface تانية فدي بالنسبالها Dependency

الـ **DI** بيكون جواه **Container** ده اللي بتسجل فيه كل الـ **Dependencies** اللي عندك ويعمل منها **Resolve** 

### AddScoped
أنت كدا قولتله أن في Dependency معينة لو حد محتاجها جهزهاله وابعتهاله
عملية الـ Dependency Registration بيكون ليها Lifetime أو Scope اللي هو لو عملت دلوقتي الـ Dependency دي هتفضل موجودة لحد امتى

عندك 3 أنواع من الـ Scopes
1. AddTransient
في الريكوست الواحد لو في كذا مكان طالب Dependency فكل مكان اعمله Instance لوحده
2. AddScoped
اعمل Instance واحدة لكل ريكوست ودي بنستعملها في أكتر من 80% من الحالات
3. AddSingleton
اعمل Instance واحد للأبلكيشن كله حرفيا

وهنا بالمناسبة لو سجلت السيرفس أكتر من مرة ورا بعض هي بتعمل Override بتاخد آخر حاجة عملتها

جرت العادة أننا نعمل Interface لأي Scope ومش بنعتمد على الـ Class بس
يعني دايما اعمل Interface حتى لو هيبقى معتمد عليه Class واحد عشان لو خصوصا هتعمل أكتر من Class لنفس الـ Interface مثلا واحد للـ Development وواحد للـ Testing وهكذا