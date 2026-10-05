## What Happen After Kestrel?
ببساطة Kestrel بياخد الـ Bytes اللي جاية يحولها لـ Text وبعدها يحولها لاوبجيكت من نوع `HttpContext` الاوبجيكت ده بيكون شكله مثلا
```cs
HttpContext
{
  "Request": {
    "Method": "GET",
    "Path": "/api/users",
    "Headers": { "Authorization": "Bearer eyJhbGci..." },
    "Body": "Stream"
  },
  "Response": {
    "StatusCode": 200,
    "Body": "Stream"
  },
  "User": null,
  "Items": {}
}
```

بيقوم الـ Object ده مسلمه للـ **Request Pipeline**
****
## Request Pipeline

![](https://res.cloudinary.com/du7j80lol/image/upload/v1790525304/obsidian/eztfupzwfazbrchqes0n.png)

دي عبارة عن زي خط إنتاج بيكون جواه كذا عامل ودول هما الـ **Middlewares**

الخط ده شغال رايح جاي، طب أزاي؟
```cs
// 1. Inbound Request رايح
Console.WriteLine("قبل ما يعدي");

await next(context); // Go Next Middleware

// 2. Outbound Response جاي
Console.WriteLine("بعد ما رجع");
```

لو في Middleware نسيت تستدعي `next()` أو حصل شرط ميكملش الـ Request بيقف ويرجع ميدخلش الـ **Endpoint**

وفي قاعدة أن أي حاجة قبل `next` بتتنفذ في الريكوست
وأي حاجة بعد ال `next` بتتنفذ بعد ما الـ **Response** يجي أو يعني بعد ما يوصل للـ **Endpoint** ويرجع تاني بالعكس
****
## Middleware
الـ Middleware زي ما قولنا أنه عامل بيقف جوا الـ **Pipeline** كل وظيفته أنه يعدل في الـ `HttpContext` سواء يزود فيه أو يحط فيه يتشيك حاجة فيه أيا كان هو بيتمحور حوالينه

أول ما بيخلص اللي عمله الطبيعي انه يعمل `next()` عشان يخليه يروح للـ Middleware اللي بعده

بس لو قرر ميعملش `next()` فدي بتبقى اسمها **Short Circuiting** أنه بيوقف الكونتيكست قبل ما يوصل للـ **Endpoint** أو الخطوة اللي بعدها وده طبعا بيرجعلك الـ Response يأما بأيرور ماسج أو مشكلة حقيقية
### Middlewares (Order of Execution)
وفي هنا قاعدة مهمة أن الترتيب في الـ Middlewares أجباري ومفهوش أي هزار
يعني لازم تحدد مين هيدخل قبل مين ومين بعد مين

يعني مثلا الميدلوير بتاع الـ **Authorization** لازم يجي بعد الـ Routing والـ Authentication، عشان يتأكد هل اليوزر ده مسموح له يدخل الـ Route ده ولا لأ

### Routing Middleware
هنا بقى بعد غالبا أغلب الـ Middlewares بيبدأ يروح للـ Routing Middleware عشان يشوف هو هيروح لأني Endpoint بالضبط وهنا الـ Endpoint لي **Unique Composit Key** بيتكون من الميثود بتاعته مع الـ **Path** فالميدلوير بيجيبه من هنا وبعدها يدخل بقى على الـ Endpoint Middleware
### Endpoint Middleware
هنا خلاص بقى بتدخل على الـ Controller بتاعك وتنفذ سواء البيزنس لوجيك أو الداتا بيز اوبريشنز اللي هنتكلم عنها قدام
****
بعدها الريسبونس بيرجع تاني بالعكس لحد ما يوصل لـ **Kestrel** ويروح للكلاينت بتاعنا في سلام

