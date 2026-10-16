## 1. Controller Activation & DI
في ASP الـ Controllers تقدر ببساطة explicitly تحقن في الكونستراكتور الـ dependencies بتاعتك
### Controller Activation
لما بتكتب Controller بيكون مجرد Blueprint ملوش معنى بيبدأ يشتغل فعلا لما الريكوست يجي ويعدي على الـ UseRouting فيبدأ بعدها يتعمل للـ Controller الـ **Instantiate** دي اسمها بقى مرحلة الـ **Activation**
### Constructor injection
أحنا درسنا في دروس الـ [[03- Dependency injection In General]] أننا بنعمل Injection للـ dependencies في الكونستراكتور فده مثال بسيط عبارة عن Service صغيرة وزي ما مكتوب في المقال أن العرف أننا نعمل Interface دايما
```cs
public interface IDateTime
{
public DateTime Now { get; }
}
```

وده الـ Class المسئول
```cs
public class DateTimeService : IDateTime
{
public DateTime Now => DateTime.Now;
}
```

يعني أنت عملت انترفيس كعقد وعملت الـ Service عشان اللوجيك

بعدها هتعرف الـ Scope عشان الـ DI يلقطه زي ما شرحنا زمان
```cs
builder.Services.AddSingleton<IDateTime, DateTimeService>();
```

وبكدا خلاص استعمله في الـ Controller بتاعك:
```cs
public class ProductsController : ControllerBase
{
private readonly IDateTime _dateTime;

public ProductsController(IDateTime dateTime)
{
_dateTime = dateTime;
}
}
```

### Action injection with `FromServices`
هنا أنت تقدر تحقن dependencies بشكل مباشر في الأكشن
```cs
public IActionResult About([FromServices] IDateTime dateTime)
{
    return Content( $"Current server time: {dateTime.Now}");
}
```
****
## 2. Action Invocation & Parameters
المرحلة اللي فوق الـ `HttpContext` بيكون عرف الـ Controller ودخل عليه ونفذ الـ DI وكل ده

تاني مرحلة هو أنه يدخل الـ Action ذات نفسه، طب بيختاره أزاي؟

دي مرحلة أسمها **Routing Match** في الـ **Middleware** بيبدأ يشوف الـ **RequestURL** مع الـ **HTTP Verb** ويقارنها بالـ **Routing Table** فبكدا بيحدد الـ Action المناسب ويشوف بقى هينفذ ايه سواء الـ ModelState أو الـ Filters أيا يكن

بعد ما بيعرف الـ Controller والـ Action بيبدأ ينفذ الـ Action Parameters & Model Binding ودي شرحناها بالتفصيل في جزء الـ Model Binding والأولويات [[07- Model Binding & Validation]]

بعدها بيشوف الـ ModelState والـ Validation ولو لقى غلط وفي `[ApiController]` يبدأ يرد بـ 400 وهكذا
****
## 3. Results Execution & Content Negotiation

كلنا عارفين الـ `IActionResult` وبنستعملها في كل الـ Controllers تقريباً
بس الفكرة هي ليه اتعملت أصلا وايه اللي بيحصل لما بنعمل return؟

دي interface بتديك مرونة ترجع responses مختلفة على حسب اللوجيك بتاعك يعني لو الداتا تمام ترجع 200 ولو مش موجودة ترجع 404
### ايه اللي بيحصل Under The Hood؟
أول ما الـ Action بتخلص بترجع الـ Result والـ Framework بينادي ميثود اسمها `ExecuteResultAsync` :
1. الـ Output Formatter: المسؤول عن تحويل الـ Object هو كائن اسمه SystemTextJsonOutputFormatter
2. الـ Content Negotiation: بيروح يبص على الـ Accept Header اللي جاي في الـ Request زي application/json
3. لو الكلاينت طلب صيغة مش متسجلة زي XML من غير ما نضيف الـ XML Formatter في Program.cs بيحصل Fallback وبيرجع للـ Default وهو الـ JSON
4. الـ Formatter بياخد الـ Object ويكتبه علطول جوه الـ Response Body Stream

### الفرق بين `IActionResult` و `ActionResult<T>`
- الـ `IActionResult`: بترجع أي Response بس عيبها إن Swagger مابيبقاش عارف نوع الـ Model اللي راجع في الـ 200 OK
- الـ `ActionResult<T>`: بتحدد نوع الـ DTO صريحة فبتمكّنك ترجع الـ Object مباشر من غير Ok() وبيظهر صح في الـ Schemas بتاعة Swagger

```cs
[HttpGet("{id}")]
public ActionResult<Product> GetProduct(int id)
{
    var product = _service.GetById(id);
    if (product is null) return NotFound();
    
    return product;
}
```
