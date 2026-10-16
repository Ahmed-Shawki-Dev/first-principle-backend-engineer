دلوقتي لو عندك فرونت اند بـ **JS** والباك بتاعك بـ **dotnet** مثلا
فالـ **JS** ليها اوبجيكت خاص بيها و **dotnet** انت بتشتغل بـ **Classes** وهكذا فهما وسطين مختلفين تماما وكل واحد لي شكل للداتا محدد

طب لما أنت تبعت داتا من الفرونت للباك هل من المنطقي تبعت JS Object؟
مش هينفع عشان dotnet مش هتفهمها هتقولي يعم نخليها تفهمها طب تخيل موبايل أبلكيشن مش عارف تكنولوجي جديدة وهكذا فمينفعش دي كدا Tide Coupling طب ايه الحل؟

الحل في الـ **JSON** أو **Javascript object notation** هي طريقة Standard أننا نخلي أي وسطين يتواصلوا مع بعض
****
## JSON
أكيد مش هقعد اتكلم عن الـ JSON بس ده شكله
عبارة عن Key و Value وأكيد في كلام كتير بس الـ JSON معروف

```json
{
"success":true,
"message":"تم تسجيل الدخول بنجاح.",
"data":{
"userId":"01a071e1-5479-72f1-ae14-743061c1d828",
"role":"Teacher",
"tenantSlug":"eldaftar",
"tenantId":"dd43af95-7280-4acb-8503-a2dab5136292",
"teacherId":"f3c179f9-3348-4b38-9918-00d8083c43fc"
},
"pagination":null,
"errors":null
}
```

****
## Serialization and Deserialization
ده المصطلح المسئول عن العملية دي كلها أنه يحول من C# Objects للـ JSON اللي هو Text-based هما المسئولين عن انهم يحولوا الـ JSON لأوبجيكت والأوبجيكت لـ JSON

زي ما قولنا ده مصطلح نظري، تنفيذه بشكل كامل في .Net بيتم بـ `System.Text.Json` ودي Native من اللغة نفسها

![](https://res.cloudinary.com/du7j80lol/image/upload/v1791792885/obsidian/bwmdrkrg38skrp4oqqjd.png)
### Serialization
هنا أحنا بنحول الـ Object بتاع C# لـ JSON ببساطة
### Deserialization
إنما هنا أحنا بنعمل reverse process يعني بنحول الـ JSON اللي جاي من الـ Request لـ C# Object عشان اللغة تفهمه وتعرف تتعامل معاه
****
## Request/Response Execution Pipeline
الكلام اللي فات كان أغلبه تعريف عام للعملية نفسها طب الموضوع بيتم فين في ASP.NET؟

**هنا بتكلم كـ Controllers!**

1. الـ Request يدخل بيكون جواه Request Body
2. الـ Pipline بتاخده وتديه لـ `JsonInputFormatter`
3. الفورماتر بينادي `JsonSerializer.DeserializeAsync()` عشان يحول الـ Text لـ C# DTO ويحطه في الـ Action Parameter
4. أنت بترجع الـ Response خلاص كـ C# Object من الـ Endpoint
5. الـ Pipline بتاخدها وتديها لـ `JsonSerializer.SerializeAsync()` عشان يحول الـ Object لـ Stream من JSON Text ويرجعه في الـ Response Body
****
## Reflection VS Source Generation
دول طريقتين عامين جدا بس أحنا هنتكلم في سياق الـ Serialization
بالمناسبة By Default الـ `System.Text.Json` بتستخدم الـ Reflection
### Reflection
أحنا هنتكلم عن الـ Reflection وده الطبيعي اللي بنستخدمه دايما، الكونسبت ببساطة مبني أننا بنكتشف ونتعامل مع الكود في الـ Runtime مثلا لو عندك JSON جاي من ريكوست كدا:
```json
{
  "name": "Ahmed",
  "grade": "One",
  "age": 30
}
```
رايح للـ Endpoint دي:
```cs
app.MapPost("/", (StudentDto student) => student);
```

مكتبة `System.Text.Json` محتاجة تعرف شوية حاجات:
- ايه الـ Properites اللي موجودة جوا StudentDto
- ونوع كل Property
- أزاي تحول الـ JSON Number لـ int وهكذا
هنا بنستعمل الـ Reflection عشان تساعدنا في عملية التحويل دي هي بتتشك الكود نفسه بتاعنا في الـ Runtime

لما المكتبة تحتاج تتعامل مع نوع لأول مرة، بتجمع الـ metadata وتخزنها مؤقتا لإعادة استخدامها. مش بتعيد اكتشاف خصائص `StudentDto` من الصفر مع كل request.
### Source Generation
دي حاجة حديثة في كومبايلر .Net ذات نفسه، هي ليها شرح وتفاصيل وليلة طويلة بس هتبقى حاجة صعبة جدا تتشرح بشكل أبسط، فهنعرف بس أنها موجودة مش أكتر

هي بكل بساطة بتجهز الـ metadata أو كود محسن وقت الـ build، وده ممكن يقلل تكلفة اكتشاف الأنواع وقت التشغيل ويحسن الأداء حسب الاستخدام. مش ضمان إنه أسرع في كل حالة.

ممكن تقرأ المقال ده لو حابب تعرف أزاي تتعمل
https://okyrylchuk.dev/blog/intro-to-serialization-with-source-generation-in-system-text-json/