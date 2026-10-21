هنتكلم النهارده عن كونسبت مهم جدا من كونسبتس الـ **HTTP** وهي الـ **Idempotency** وليه هو مصطلح مهم جدا
## What Is Idempotency?
Performing The Same Action Multiple Times **Has The Same Effect** As Performing It Once
يعني مثلا لما تعمل GET Request مرة زي ما هتعملها ألف مرة
### GET, PUT, DELETE
يعتبروا Idempotency لأن لو عملت Update أو Get أو Delete مرة زي ما هتعمل ألف مرة
### POST
تعتبر **Non-Idempotency** لأن كل POST Request هتبعته للسيرفر هيعتبر عملية جديدة تماما
****
## The Problem
المشكلة حاليا بقى في الـ **POST** تخيل أنت بتحول فلوس لحد وراح السيرفر وقبل ما تستلم الـ Response قفلت النت أو النت هنج، فلما النت اشتغل هتلاقي أن الريكوست بيتبعت تاني هو هو عشان هو أول مرة مستلمش الـ Response
فأصلا الطلب أول مرة كان راح السيرفر فهيروح مرة كمان وكدا هيحصل التحويل مرتين ودي كـ Data Integrity كارثة
****
## Solution
تفكيرنا في الحل هيمشي على خطوات منطقية هشرحها بشكل نظري بعدها نقسم الكود

هو عموما حاجة زي كدا تتعمل مخصوص للـ Endpoints الحساسة أو المهمة اللي مينفعش تكرار فيها وهنعمل التالي:

1. في الـ Frontend هتخلي لما الـ API ده يتبعت هتحط معاه Header اسمه `X-Idempotency-Key` أو لو على Postman وهنا هتخلي الفرونت هو اللي يـ Generate Random Key للعملية أو الـ Fetch
2. هنتخيل بقى السيناريو أنه لو الـ Browser بعت وراح للسيرفر بس الـ Client مستلمش الـ Response عشان النت، هنا البراوزر والفرونت اند بيعملوا تكنيك أنه هيبعتوا الريكوست تاني بشكل تلقائي عشان مستلموش Response فهنا بقى المشكلة أن اصلا الريكوست الأول اتنفذ بس موصلش فهيجي ريكوست تاني ويتنفذ تاني ده لو مش مهندلين الدنيا
3. عشان نحل المشكلة هنمسك الـ **Header** جوا الفلتر ونتشيك عليه لو لقيناه يكمل لو ملقاهوش يرجع Bad Request
4. لو اتبعت ولقى أنه مفيش ريكوست زيه قبل كدا هيكمل عادي ويعمل Cache للـ Result عشان يستعملها بعد كدا
5. لو لقى أن الـ Key متكرر هيقوم مرجع الـ Response بس من غير ما يدخل على الـ Action وينفذ اللوجيك لا هيبعت ريسبونس من برا برا اللي هو بس يريح الـ Browser وخلاص

دي كل الفكرة عشان متتعقدش من الكود وطبعا شرح الـ Action Filters ده موجود بشكل منفصل هنا [[09- MVC Filters]]
****
## Code
```cs
public class IdempotencyKeyActionFilter(IDistributedCache cache) : IAsyncActionFilter
{
    public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
    {
        var idempotencyKeyHeader = context.HttpContext.Request.Headers["X-Idempotency-Key"];

        if (StringValues.IsNullOrEmpty(idempotencyKeyHeader))
        {
            context.Result = new BadRequestObjectResult("Idempotency Key Not Found Please Try Again");
            return;
        }

        var key = idempotencyKeyHeader.ToString();
        var fullKey = $"Idempotency:{key}";

        var cachedResponse = await cache.GetStringAsync(fullKey);

	if (cachedResponse != null)
        {
            context.Result = new ContentResult
            {
                Content = cachedResponse,
                ContentType = "application/json",
                StatusCode = StatusCodes.Status200OK
            };
            return;
        }
        
        var executedContext = await next();

        if (executedContext.Result is ObjectResult objectResult && objectResult.Value != null)
        {
            var responseJson = JsonSerializer.Serialize(objectResult.Value);

            var options = new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(24)
            };

            await cache.SetStringAsync(fullKey, responseJson, options);
        }
    }
}
```

وده هيبقى شكل الـ Redis:
![](https://res.cloudinary.com/du7j80lol/image/upload/v1792349104/obsidian/sdphrpaozzryi4erwoj6.png)

وواخد صلاحية يوم كامل، طبعا هنا بردو لازم تكون مذاكر Redis بس لو عايز المختصر المفيد أعتبره ديكشنري بيشيل Key و Value في الرام عشان يوفر وقت زي Cache الـ Processor كدا والكمبيوتر أركتكشر وكل ده