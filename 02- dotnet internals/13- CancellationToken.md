هنتكلم عن موضوع مهم جدا وبيبان كتير قدامنا في الكود وهو `CancellationToken` دايما بنشوفه بيتضاف في الميثودز وساعات بنحطه ونمرره وخلاص من غير ما نفهم هو بيعمل ايه Under The Hood

الفكرة كلها ان السيرفر لما يجيله ريكوست وياخد وقت في التنفيذ زي انه يروح للداتابيز أو يكلم API خارجي واليوزر يقرر يقفل الصفحة أو النت يقطع عنده من غير ما العمليات دي تدعم الـ Cancellation السيرفر ممكن يفضل شغال وبيستهلك Resources وفي الآخر محدش مستني النتيجة أصلا

فـ `CancellationToken` بيجي عشان يحل المشكلة دي ويكون زي إشارة بتوصل للكود بتاعنا إن العملية المفروض تتوقف لو مبقاش فيه داعي إنها تكمل

هنفهم بقى إزاي بيشتغل كـ **Cooperative Cancellation** وإزاي بنمرره بين الـ Layers وإزاي المكتبات اللي بنستخدمها بتتعامل معاه

---
## Understanding CancellationToken

`CancellationToken` is a .NET type that allows code to observe a cancellation request.

يعني هو Struct مش Service ولا حاجة وبيمثل إشارة الإلغاء اللي الكود يقدر يراقبها

بس هنا عندنا حاجتين مهمين لازم نفرق بينهم

الـ `CancellationTokenSource` هو اللي بيقدر يطلب الإلغاء عن طريق `Cancel()` أما الـ `CancellationToken` فهو اللي بنمرره للميثودز عشان تقدر تعرف إن فيه إلغاء حصل

يعني عندنا حاجة بتطلب الإلغاء وحاجة تانية بتوصل الإشارة للكود اللي محتاج يعرف عنها

```cs
using var cancellationTokenSource =
    new CancellationTokenSource();

CancellationToken token =
    cancellationTokenSource.Token;

cancellationTokenSource.Cancel();

Console.WriteLine(token.IsCancellationRequested);
```

الكود هيوضح الفكرة جدا لما بنعمل `Cancel()` الـ Token بيبقى عنده علم إن فيه Cancellation Request حصلت وده اللي بتوضحه `IsCancellationRequested`

بس خد بالك إن ده مش معناه إن أي عملية بتستخدم الـ Token هتقف بشكل إجباري

إحنا هنا بنتكلم عن **Cooperative Cancellation** يعني الكود أو المكتبة اللي بتنفذ العملية لازم تكون بتدعم الإلغاء وتستجيب للإشارة دي

مثلا لو عندك Loop طويلة تقدر بنفسك تراجع الـ Token وتوقف التنفيذ ولو بتستخدم ميثود من Library بتدعم `CancellationToken` تقدر تبعتهولها وهي تتعامل مع الإلغاء بالطريقة المناسبة ليها

---
## How ASP.NET Core Detects Request Cancellation

طيب إحنا عرفنا إن فيه Token وإن فيه حاجة بتطلب الإلغاء، بس في تطبيق ASP.NET Core مين اللي بيطلب الإلغاء لما الريكوست يتوقف؟

في Property جوا الـ `HttpContext` اسمها `RequestAborted` ودي بتوفر الـ Cancellation Token المرتبط بالريكوست

وفي الـ Controllers مش محتاج تجيب الـ `HttpContext` بنفسك عشان تستخدمه، تقدر تستقبل `CancellationToken` كـ Parameter والـ ASP.NET Core هيوفرلك التوكين المرتبط بالريكوست

```cs
[HttpGet("{id:int}")]
public async Task<IActionResult> GetOrder(
    int id,
    CancellationToken cancellationToken)
{
    var order = await _dbContext.Orders
        .FirstOrDefaultAsync(
            order => order.Id == id,
            cancellationToken);

    return Ok(order);
}
```

لما الريكوست يتلغي التوكين بيجيله إشارة إن حصل Cancellation وعلى أساس الإشارة دي مثلا الـ DB Provider يقدر يحاول يلغي الكويري لو بيدعم ده

بس خلي بالك إن مجرد إنك بعت التوكين مش معناه إن العملية هتتقفل أو إن كل حاجة حصلت بالفعل هتتراجع

---
## Passing CancellationToken Through Every Layer

في هنا نقطة مهمة زي ما أنت شايف فوق إحنا ضيفنا الكانسليشن للـ Controller بس تخيل إنك استدعيت Service من الـ Controller والـ Service دي بتكلم Repository والـ Repository بتكلم الداتابيز

لو التوكين وقف عند الـ Controller ومبعتوش للـ Service أو الـ Repository فالطبقات دي مش هتقدر تستخدم نفس الإشارة في العمليات اللي بتنفذها

ودي اسمها **Propagation** إنك تمرر الـ Token من طبقة للتانية لحد ما يوصل للعمليات اللي محتاجة تستجيب للإلغاء

يعني مش كفاية إن الميثود تستقبل `CancellationToken` أهم حاجة إنك تمرره للعمليات اللي بتنفذ شغل فعلي

---
## CancellationToken with Entity Framework Core

في EF Core عندنا ميثودز كتير بتدعم استقبال `CancellationToken` زي `FirstOrDefaultAsync` و `ToListAsync` و `SaveChangesAsync`

فإحنا لما بنبعت التوكين للميثود دي بنسمح للـ EF Core والـ Database Provider إنهم يستجيبوا لطلب الإلغاء أثناء تنفيذ العملية

بس في نقطة مهمة جدا هنا

لو أنت بعت حاجة للداتابيز وخلاص اتنفذت وفي نفس اللحظة الريكوست اتلغى ووصلتلك `OperationCanceledException` هتفتكر إن خلاص البروسيس اتلغت مع إن في الحقيقة العملية ممكن تكون اتنفذت بالفعل في الداتابيز

وده مهم جدا في العمليات اللي بتعمل تعديل على الداتا لأن إلغاء الريكوست مش معناه بالضرورة إن التغيير اللي حصل في الداتابيز اتراجع

---
## CancellationToken with HttpClient

هنا هنتعامل مع الـ `HttpClient` في الـ Request والـ Read عشان لو الـ Client قفل منفضلش مستنيين رد من API خارجي محدش مستني نتيجته

```cs
public class InventoryService
{
    private readonly HttpClient _httpClient;

    public InventoryService(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    public async Task<string> GetInventoryAsync(
        int productId,
        CancellationToken cancellationToken)
    {
        using var response = await _httpClient.GetAsync(
            $"inventory/{productId}",
            cancellationToken);

        response.EnsureSuccessStatusCode();

        return await response.Content.ReadAsStringAsync(
            cancellationToken);
    }
}
```

هنا إحنا بنمرر نفس التوكين للـ HTTP Request ولعملية قراءة الـ Response Content

فلما يحصل Cancellation والعمليات دي تستجيب ليه نقدر نوقف انتظار الرد أو قراءة المحتوى بدل ما نكمل شغل مش محتاجينه

بس ده مش معناه إن السيرفر الخارجي نفسه وقف التنفيذ لأن ممكن يكون استقبل الريكوست وبدأ ينفذ العملية بالفعل

---
## CancellationToken in CPU-Bound Work

الحوارات اللي فاتت دي كلها كانت ليها علاقة بالـ DB والـ HTTP والاعتماد على Libraries بتدعم الـ Cancellation

بس تخيل أنت دلوقتي بتعمل عملية CPU-Bound تقيلة جدا وبتلف في Loop والريكوست حصله Abort في لحظة ما هتوقف الدنيا إزاي عشان متكملش عالفاضي؟

هنا بقى إحنا اللي محتاجين نراعي الـ Cancellation أثناء التنفيذ

```cs
public Result CalculateReport(
    IEnumerable<Order> orders,
    CancellationToken cancellationToken)
{
    decimal total = 0;

    foreach (var order in orders)
    {
        cancellationToken.ThrowIfCancellationRequested();

        total += order.Total;
    }

    return new Result(total);
}

public record Result(decimal Total);
```

هنا بنستخدم `ThrowIfCancellationRequested()` عشان لو فيه Cancellation Request حصلت يرمي `OperationCanceledException` ويوقف تنفيذ الميثود بدل ما يكمل اللوب

ودي حاجة إحنا بنكتبها مانيول لأن اللوب دي مش بتستخدم عملية Async بتدعم التوكين تلقائيا

---
## Sources

- [How to Use CancellationToken in ASP.NET Core Without Leaving Work Running](https://www.c-sharpcorner.com/article/how-to-use-cancellationtoken-in-asp-net-core-without-leaving-work-running)