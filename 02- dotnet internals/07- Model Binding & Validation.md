لما تبعت Request من الـ Client الكلاينت نفسه بيعمل للـ Data حاجة اسمها **Encoding** يعني بيحولها لـ Bytes معينة من نوع `UTF-8` قبل ما تمشي في السلك

فلما يوصل للـ Kestrel بياخد الـ Bytes دي ويكريت اوبجيكت من نوع `HttpContext` وبيعمل عملية **Decoding** للـ **Headers** والـ **Path** بس عشان الـ Framework يعرف يستعمل الـ `UseRouting` والـ Middlewares وتعرف هيوديه فين وكدا

بس Kestrel مش بيعمل **Decode** للـ Body لـ String كبير وسيط في الميموري طب ليه؟

عشان الـ Body متعرفش حجمه وممكن يكون كبير جدا فده بيوفر في الأداء والميموري ويسيبه كـ Bytes Stream

طب أمال ايه بيحصل عشان يتحول لـ Objects والـ Model Binder يفهمه؟

هنا بقى بيجي دور الـ **Reflection** والـ **Startup Phase**

أول ما الابليكشن بيشتغل خالص الـ Reflection بيمسك الكود ويشوف الكنترولرز والاكشنز والـ Attributes والـ Parameters بالأنواع بتاعتهم زي `int` أو `DateTime` أو الـ DTOs ويجمع كل الـ **Metadata** دي في اوبجيكت واحد اسمه **Endpoint**

لما الـ Request يدخل والـ `UseRouting` يعمل Match بيمسك الـ Endpoint ده ويحطه جوه الـ `HttpContext`

أول ما نوصل للـ Endpoint بتاعتنا بيجي دور موضوعنا النهاردة **Model Binder**

بيقرا الـ Metadata المخزونة جوه الـ Endpoint في الـ `HttpContext` فيعرف كل Parameter نوعه ايه وجاي منين

لو جاي من Route أو Query بياخده يحوله بـ TypeConverters

ولو فيه داتا من نوع JSON وفي الـ Body بينادي الـ **Serializer** يقرا الـ Bytes Stream دي ويحولها لـ C# Object بشكل مباشر جوه الـ DTO بتاعك

ومن الرحلة دي كلها بقى تبدأ تخش الاكشن بتاعتك معاك Data جاهزة بـ Types اللي أنت محتاجها
****
## Model Binder
أنت عندك صور كتير من الـ Data اللي جاية من الـ Client سواء جاية في:
- **Route values** → `/products/10`
- **Query string** → `/products?id=10`
- **Request body** → JSON, XML, form data
- **Headers** → `Authorization`, custom headers
دول كلهم موجودين جوا الـ `HttpRequest` لو معندناش Model Binding كان زمانا عشان نستعملهم هنكتب كدا:
```cs
var id = httpContext.Request.RouteValues["id"];  
var name = httpContext.Request.Query["name"];  
var token = httpContext.Request.Headers["Authorization"];
```

ده شكل Manual وممل ومش عملي
فببساطة:
> **Model Binding = automatic data extraction**

فهي من الآخر بتقرأ الـ `HttpRequest` وتشوف فيه الـ **Sources** زي الـ body والـ route والـ headers وهكذا

وبعدها يعمل **matching** للـ **Values** ويعملها **Assign** للـ Action Method Parameters اللي عايزاها

فبدل ما تكتب كدا:
```cs
public IActionResult Get()  
{  
    var id = HttpContext.Request.RouteValues["id"];  
    ...  
}
```
هتكتب كدا:
```cs
public IActionResult Get(int id)  
{  
    // id is already filled  
}
```

### Default binding priority
لو مكتبتش الـ **Attribute** بإيدك زي `[FromRoute]` مثلا ففي ترتيب الـ Framework بيمشي بيه

- Simple Types (`int`,`string`,`bool`)
	- **Route values** (e.g., `/users/{id}`)
	- **Query string** (e.g., `?id=123`)
	- **Form data** (requires `[FromForm]`)
	- **Request body** (JSON, XML, etc. — requires `[FromBody]`)
	- **Headers** (requires `[FromHeader]`)
- Complex Types (Models/Classes)
	- **Request body** (JSON, XML, etc. — requires `[FromBody]`)
	- **Route values** (e.g., `/users/{id}`)
	- **Query string** (e.g., `?id=123`)
### How does ASP.NET know where to bind from?
في الطبيعي هو بيبحث في كل الـ Sources بس أنت ممكن تجبره بشكل مباشر زي أنك تعمل:
```cs
public IActionResult Get([FromRoute] int id)
```

وهكذا بقى عندك `[FromQuery]` و `[FromBody]` و `[FromHeader]` و `[FromForm]`

### Model Binding Validation
![](https://res.cloudinary.com/du7j80lol/image/upload/v1791738559/obsidian/oqqbzcayi47rd4gmbqlm.png)

ده بيبقى فاليديشن من الـ Model Binding قبل ما تروح للـ Validation اللي بعده هنا بيلقط الـ Type لو كاتب Type غلط بيرجعلك Bad Request دايركت قبل ما ينفذ أي فاليديشن بعده
### Model Validation using Data Annotations
دي الحاجة المعروفة اللي بتكتبها في الـ `DTO` وهكذا
```cs
[Required(ErrorMessage = "Person {0} should not be null")]
// OR
[Required]
// ...
```

فبكدا:
```text
Request arrives  
↓  
Model Binding creates Person object  
↓  
Model Validation runs (Data Annotations)  
↓  
Controller action executes
```
### **`ModelState`**
دي Property من `ControllerBase` بيبقى جواها الـ **Result** بتاعت الـ Model Validation

بتتملي **Automatic** بعد الـ Model Binding والـ Model Validation من نوع Data Annotations أو حتى **FluentValidation** بس دي مش اوتوماتيك

الحاجة المهمة فيها أنك ممكن تتـ Check الـ Validation تشوفه مثلا `ModelState.IsValid` أو تشوف الـ `ModelState.Values` أو `ModelState.ErrorCount` وهكذا

تقدر تضيف ليها Custom Errors مثلا تضيفها من الـ Fluentvalidation أو تكتبها بإيدك وهكذا بشكل كويس جدا يعني ممكن توحدها وتخليها المصدر اللي بيجي منه الـ Errors
### Fluentvalidation
دي مكتبة في .Net للفاليديشن زي Zod كدا وليها ميزات كتير:
- الـ **Separation of Concerns** لأنك بتفصل الفاليديشن عن اللوجيك
- الـ **Strongly Typed** عشان التايب سيفتي
- أنها **Reusable** تقدر تستعمل الـ Class أكتر من مرة
- أنها **Testable** تقدر تعملها Test بسهولة

أزاي نستعملها
- هتنزل المكتبة
```bash
dotnet add package FluentValidation 
dotnet add package FluentValidation.DependencyInjectionExtensions
```
- بعدها زي Zod بس هتعمل Class تفاليديت بيه الـ Class:
```cs
public class PersonValidator: AbstractValidator<Person>  
 {  
     public PersonValidator() {   
         RuleFor(person => person.Name).NotEmpty().WithMessage("Person name should not be null")  
             .MinimumLength(3).WithMessage("should be 3 chars long");  
         RuleFor(person => person.Email).NotEmpty().WithMessage("Email should not be null")  
                 .EmailAddress().WithMessage("Email is not in correct format");  
         RuleFor(person => person.Phone).NotEmpty().WithMessage("Phone should not be null")  
             .Matches(@"^\d{10}$").WithMessage("Phone should be 10 digits");  
         RuleFor(person => person.Password).NotEmpty().WithMessage("Password should not be null")  
             .MinimumLength(6).WithMessage("Password should be minimum 6 chars long");  
         RuleFor(person => person.ConfirmPassword).Equal(person => person.Password)  
             .WithMessage("Confirm password should match the password");  
           
     }  
 }
```
- تضيفه للـ `Program.cs`:
```cs
builder.Services.AddValidatorsFromAssemblyContaining<PersonValidator>();
```
- في الآخر تعمل Injection للـ Validatior:
```cs
[Route("person")]  
 public class PersonController : Controller  
 {  
     private readonly IValidator<Person> _validator;  
  
     public PersonController(IValidator<Person> validator)  
     {  
         _validator = validator;  
     }  
     public async Task<IActionResult> Index(Person person)  
     {  
         var result = await _validator.ValidateAsync(person);  
  
         if (!result.IsValid)  
         {  
             foreach (var error in result.Errors)  
             {  
                 ModelState.AddModelError(error.PropertyName, error.ErrorMessage);  
             }  
             return BadRequest(ModelState);  
         }  
        return Ok(person);  
     }  
 }
```