الـ MVC Filter هو Middleware مخصص لطبقة الـ MVC أو الـ Controller بس

بنستعمله عشان مبدأ الـ Sepatation Of Concerns وعشان منكررش الكود وعشان منخلطش البيزنس لوجيك مع الفلاتر اللي مسئولة عن حاجات معينة زي الكاشنج أو الأوثنتكيشن وهكذا

ده ترتيب الـ Filters والصورة الكبيرة قبل ما نبدأ:
```
Request Arrives
   │
   ▼
[Authorization Filters]  ──► (أول حارس: هل مسموح لك تدخل اصلا؟)
   │
   ▼
[Resource Filters]       ──► (تاني حارس: بيحوط العملية كلها - مفيد للـ Caching)
   │
   ▼
 (Model Binding & Validation)
   │
   ▼
[Action Filters]         ──► (حارس الميثود: قبل وبعد تنفيذ الـ Business Logic)
   │
   ▼
 (Action Execution)      ──► (الكود بتاعك جوه الـ Controller هينفذ هنا)
   │
   ▼
[Result Filters]         ──► (حارس الناتج: قبل وبعد تشغيل الـ ActionResult)
   │
   ▼
[Exception Filters]      ──► (شبكة الأمان: بتلقط أي Exception حصل في أي مرحلة فوق)
```

أحنا بنستعمل الـ Filters تخيلها كبواب الشقة مش العمارة، بواب العمارة كلها هو الـ Middleware أو الـ Request Pipline أنما الفلترز معمولة عشان تضبط Controllers أو أكشنز معينة وبيكون ليها صلاحيات أكبر في بعض الأنواع ووصول للـ Arguments وحاجات أكتر بكتير من الميدلوير
****
## Action Filter
بيتنفذ قبل وبعد الـ Action أو الـ API ولي Access كامل على كل الـ Arguments اللي موجودة في الـ **Action** بنستعمله في سيناريوهات كتير من أكترها شهرة أننا نمنع التكرار، لو في كونديشن بتكرره كل مرة ممكن تعمل فلتر هو ينفذ كل ده من غير ما تكرره كل مرة

وده كود عملي ببساطة أحنا هنورث من `IAsyncActionFilter` هو جواها بتنفذ زي الـ Middleware قبل وبعد الـ next() فاهم الفكرة؟

وبالمناسبة في حاجة اسمها `context.HttpContext.Items["UserContext"]`
أنت هنا فتحت زي حقيبة أو مكان يتخزن في داتا بدل ما في الكنترولر تنادي الـ UserContext تاني انت خزنته في الخزنة وبتنادي على الـ HttpContext جوا الـ Controller وبارك الله فيما رزق

```cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Filters;
namespace api.Filters;

public class CheckTeacherTenantFilter : IAsyncActionFilter
{
public async Task OnActionExecutionAsync(
ActionExecutingContext context,
ActionExecutionDelegate next
)
{
// 1. Validate Route Parameter
if (
!context.ActionArguments.TryGetValue("slug", out var slugObj)
|| slugObj is not string tenantSlugRoute
)
{
context.Result = new ObjectResult(ApiResponse<object>.Fail("مسار غير صالح."))
{
StatusCode = StatusCodes.Status400BadRequest,
};
return;
}

// 2. Get User Context
var userContext = context.HttpContext.User.GetUserContext();
var tenantSlugContext = userContext?.TenantSlug;

// 3. Ownership & Tenant Check
if (userContext == null || tenantSlugContext != tenantSlugRoute)
{
context.Result = new ObjectResult(
ApiResponse<object>.Fail("غير مصرح لك بالوصول لبيانات هذا السنتر/المدرس.")
)
{
StatusCode = StatusCodes.Status403Forbidden,
};
return;
}

// 4. Pass Data & Continue
context.HttpContext.Items["UserContext"] = userContext;
await next();
}
}
```

هنا بدل ما كل مرة بنكرر اللوجيك في الدفتر أننا نقارن بين الـ Slug Argument والـ Slug Claim كل مرة في الكنترولر خلينا فلتر موجود هو يتشك وبكدا منعنا التكرار وفصلنا المسئوليات

عشان تشغل بقى الكود ده في كذا طريقة أنا استعملت الـ Scope عادي وضيفته لكل الكنترولرز اللي عندي طريقة مرهقة ومتكررة بس سهلة ممكن تشوف طرق تانية قدام
بتحط الأتربيوت ده
```cs
[ServiceFilter(typeof(CheckTeacherTenantFilter))]
```
بس