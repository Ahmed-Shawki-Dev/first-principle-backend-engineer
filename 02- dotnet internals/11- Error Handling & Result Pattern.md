المقال ده في غاية الأهمية وهيتقسم لـ 4 مواضيع

كلنا بنبرمج عادي بس دايما في مشكلة أننا منقدرش نتخيل أو نمسك كل الـ **Errors** اللي بنواجهها سواء من الـ **Server** او الـ **Client**
أصلا أغلبنا مش بيستعمل Try-Catch لأن بتبقى تكرار غبي وصعب نلتزم بيها في كل Action

وبكل صراحة كـ Frontend قبل ما أكون Backend مشكلتي أن الـ Backend بيرجع ردود مش موحدة ودي فعلا مشكلة مش بسيطة لو معتمد عالـ Response بتاع الفريمورك، يعني أحنا عايزين نوحد الـ Response سواء Success أو Fail يكون موحد للفرونت اند ومش هنلتزم بالـ Default بتاع مايكروسفت

فهنا أحنا مش بس هنشرح أزاي تمسك الايرورز، أنت هتفهم أزاي تمسك الـ Error وكمان أزاي تبعته للـ Frontend أو ال Client بصورة موحدة فممكن تعتبرهم موضوعين وكل موضوع يندرج تحته موضوعين
****
## 1. Contract Layer
مع أن المفروض التوبيك عن الـ Errors بس هبدأ الأول نعمل Response عام للسيستم بتاعنا عشان نفهم الصورة من فوق
```cs
public record ApiResponse<T>(
    bool Success,
    string Message = "",
    T? Data = default,
    List<string>? Errors = null
);
```

ده أبسط شكل ومتنساش تضيف الـ Metadata بس مش عايز أعقد الدنيا في الشرح ده الشكل اللي الفرونت هيستعمله دايما

لو عايز تستعمله هتعمل كدا مثلا:
```cs
[ApiController]
public class StudentController:ControllerBase
{
    [HttpGet("/student")]
    public ActionResult<StudentDto> Get()
    {
        var student = new StudentDto("Ahmed", "Shawki");

        var studentResponse = new ApiResponse<StudentDto>(true,"Student Added Successfully",student,null);
        
        return Ok(studentResponse);
    }
}
```

موضوع مرهق جدا جدا أنك كل مرة تعمل كدا فهل في حل يسهل الدنيا شوية؟ بدل ما كل Action أفضل اعمل اوبجيكت `ApiResponse` الحل أننا نمسك الـ `ControllerBase` ونعمل class يعمله Wrap ونعمل Custom Action Results خاصة بينا كمان

بص كدا هنعمل Class بالشكل ده:
```cs
[ApiController]
public class BaseApiController:ControllerBase
{
    public ActionResult<T> Success<T>(T data,string message)
    {
        var apiResponse = new ApiResponse<T>(true, message, data, null);
        return Ok(apiResponse);
    }
    // ....
}
```

هتحط فيه كل الـ Methods اللي بتستعملها Bad Request بقى وهكذا وشوف شكل الكنترولر دلوقتي بقى عامل أزاي:
```cs
[ApiController]
public class StudentController:BaseApiController
{
    [HttpGet("/student")]
    public ActionResult<StudentDto> Get()
    {
        var student = new StudentDto("Ahmed", "Shawki");
        
        return Success(student,"Student Retrieved Successfully");
    }
}
```

الطريقة دي طريقة تواصل خارجية بينك وبين الـ Frontend وبتاع الفرونت هيدعيلك والله
****
## Result Pattern
الكلام اللي فات حلو أوي للتواصل مع الفرونت أو للـ Controllers بس أحنا مش بنشتغل كدا أحنا بنقسم الشغل مثلا بنعمل Services أو Repository والحاجات دي فأنت تخيل في Service بتعمل Check على حاجة مش موجودة وعايز ترجع Not Found يعني عايز تتواصل بشكل داخلي بين الكنترولر والـ Service هتعمل ايه

بص:
```cs
var account = await _accountRepository.GetAccountDetailAsync(accountId);  
    if (account == null)  
        return null;
```

هرجع `null`؟ لا مينفعش أنت كدا فرقت بين الـ NotFound ولا أي حاجة؟ لا دي طريقة جينيريك جدا ومش منظمة

خلاص هعمل Throwing Exceptions
```cs
throw new ArgumentException("Insufficient funds");
```

دي طريقة مش كويسة بردو عشان هتضطر تعمل Try-Catch وتعقد الدنيا ولو نسيت هتضطر تعمل

فالحل هنا للتنظيم الداخلي هو الـ **Result Pattern**

هنا ببساطة أنت بتخلي الموضوع أسهل بتخليه يرجع Success أو Fail ودي أبسط شكل في الشغل الحقيقي هتحدد كل Fail سواء NotFound أو BadRequst الخ بس أحنا هنشرح أبسط شكل:
```cs
public record Result<T>(bool IsSuccess, string Message, T? Data)
{
    public static Result<T> Success(T data, string message = "") 
        => new(true, message, data);

    public static Result<T> Failure(string message) 
        => new(false, message, default);
}
```

بص كدا بقى لو أنت في Service هيحصل ايه
```cs
public class StudentService
{
    public Result<StudentDto> GetStudent()
    {
        var student = new StudentDto("Ahmed", "Shawki");

        return Result<StudentDto>.Success(student, "Student Retrieved Successfully");
    }
}
```

هتيجي عالكنترولر بالشكل ده:
```cs
public class StudentController(StudentService studentService):BaseApiController
{
    [HttpGet("/student")]
    public ActionResult<StudentDto> Get()
    {
        var student = studentService.GetStudent();

        if (student.IsSuccess == false)
        {
        return BadRequest("Student Failure");
        }
        
        return Success(student.Data,student.Message);
    }
}
```

طبعا ده أبسط شكل للشرح بس أنت تعمق أكتر أهم حاجة تفهم الباترن بشكل بسيط

أحنا كدا خلصنا وهندلنا طرق التواصل أو الـ Contracts سواء خارجية أو داخلية هنبدأ بقى من دلوقتي الموضوع التاني وهو الـ Error Handling ذات نفسه
****
## Global Exception Handling
تخيل يا غالي حصلت مشكلة أو Server Error 500 والريكوست طالع، ممكن يطلع للمستخدم الإيرور على هيئة HTML أو بشكل مش كويس ومش واضح ودي مشكلة أحنا فوق عمالين نعمل نظام موحد ومش عارف ايه عشان ننضف الشغل

فهنا بقى .net اصلا في الطبيعي في آخر الـ Http Pipline بيكون في Middleware بيمسك أي ايرور مش متهندل ويرجع By Default حاجة أسمها `ProblemDetails` ده JSON مثالي للإيرورز ومعرفش ايه بس أحنا اتفقنا أننا عاملين Contract وعايزين نحافظ عليه عشان الفرونت ربنا يكرمك

فهنا أحنا هنعمل كلاس اسمه `GlobalExceptionHandler` هو اللي هيقف في الـ Pipline مكان بتاع مايكروسفت ويحط الـ Response بتاعنا الجميل

```cs
public class GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger) : IExceptionHandler
{
    public async ValueTask<bool> TryHandleAsync(HttpContext httpContext, Exception exception,
        CancellationToken cancellationToken)
    {
        logger.LogError(exception, "An unhandled exception occurred: {Message}", exception.Message);

        httpContext.Response.StatusCode = StatusCodes.Status500InternalServerError;
        httpContext.Response.ContentType = "application/json";

        var response = new ApiResponse<object>(false, "Something went wrong on our side. Please try again later.",
            null,
            [
                exception.Message
            ]);

        await httpContext.Response.WriteAsJsonAsync(response, cancellationToken);

        return true;
    }
}
```

هشرحلك متقلقش بص
أحنا عشان نقدر نعمل Class يهندل الإيرورز محتاجين نورث من `IExceptionHandler` وغالبا أنت متعود على كدا من مايكروسفت

فأنت أول ما هتورث هتخليه يكملك الميثود وأنت حط اللوجيك بقى

أحنا قولنا انه Middleware وواقف معنى كدا ايه؟ عسل أنه هيلعب في الـ `HttpContext` لو متعرفش ايه ده فروح ذاكره دلوقتي ده أهم حاجة في حياتك

هيمسك الريسبونس يبعت جواه `StatusCode` ويخليه `application/json` ويعمل `ApiResponse` الجميل بتاعنا في الآخر هيبعت الريسبونس ده للـ HttpContext ويرجع `true` ودي زي رسالة ضمنية بتقولها للفريمورك اللي هو أنا هندلت الشيك خلاص أنا ضبطت الدنيا

عايز بقى تشغل الكلام ده هتروح للـ `Program.cs`
تحط دول:
```cs
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();
```

وعند الـ app تحت حط دي:
```cs
app.UseExceptionHandler();
```

ومبروك عليك ده بص كدا شكل الريسبونس الموحد الجميل بتاعنا:
```json
{
"success":false,
"message":"Something went wrong on our side. Please try again later.",
"data":null,
"errors":[
"Unable to resolve service for type 'ResultPattern.Services.StudentService' while attempting to activate 'ResultPattern.Controllers.StudentController'."
	]
}
```

طبعا الريسبونس ده كارثة ومينفعش في البرودكشن ترجع Errors زي دي أبدا بس المهم بس تكون فاهم الفكرة العامة

لقيت دول ممكن يفيدوا
### Interview-Ready Summary
- `UseExceptionHandler` = catch-all middleware, compatible with all ASP.NET Core versions
- `IExceptionHandler` (.NET 8+) = typed, chainable handlers preferred for new projects
- Exception filters = MVC-specific, controller-level
- `ProblemDetails` = RFC 7807 standard for structured error responses
- Define a domain exception hierarchy to map business errors to HTTP status codes
- Log every exception with request context and a correlation/trace ID
- Never expose raw stack traces to clients

A strong interview answer:
> _**"In ASP.NET Core, global exception handling is best done with UseExceptionHandler middleware or the IExceptionHandler interface introduced in .NET 8. Both catch all unhandled exceptions in one place, log them, and return a structured ProblemDetails response. IExceptionHandler allows typed, chainable handlers where each handles a specific exception type. The key is to keep controllers free of try-catch blocks and let the global handler decide the HTTP status code."**_

****
## Custom Bad Request Response
تخيل أنت دلوقتي عامل فليديشن مثلا أو الـ Client عامل حاجة غلط مثلا:
```cs
public record AddStudentDto(
    [MinLength(2, ErrorMessage = "First Name At Least 2 Chars")]
    string FirstName,
    [MinLength(2, ErrorMessage = "Last Name At Least 2 Chars")]
    string LastName);
```

هنا أنت حاطط شرط وهو كتب:
```json{
  "firstName":"a",
  "lastName":"a",
}
```

هنا بيتدخل حاجة أسمها `ModelState` بس أحنا مش زي زمان بنهندله دايركت في كل Controller مايكروسفت ضافت زي Global Validation Response اسمه `ValidationProblemDetails` ده شكله:
```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "LastName": [
      "Last Name At Least 2 Chars"
    ],
    "FirstName": [
      "First Name At Least 2 Chars"
    ]
  },
  "traceId": "00-92612aed4b9deb86056599184d777930-ef2d21e400000a72-00"
}
```

وزي ما أنت عارف هدفنا أننا نخليه يبقى شكله زي `ApiResponse` يعني في السكشن ده هنتكلم أزاي نوحد الـ Client Bad Request عشان بردو الفرونت اند التوبيك اللي فات كان للسيرفر ايرورز

اللي بيطلع الإيرورز ويضبط كل ده لما تحط أتربيوت `[ApiController]` فأحنا هنلعب فيه بقى عشان نخليه بدل ما يرجع الريسبونس اللي فوق يرجع الريسبونس بتاعنا

وهنا لازم تكون فاهم الـ `ModelState` شغال أزاي ممكن تذاكره بس كملخص اعتبره ديكشنري شايلة أيرورز

المهم دلوقتي أحنا هنعدل زي ما قولنا في `[ApiController]` ودي حاجة تبع الفريمورك فبص بقى بما أن دي **Configuration** فالفريمورك بيخليها `IConfigureOptions` عشان تتحكم فيها براحتك فهنعمل Class يورث منها ونعدل في السلوك عشان يرجع `ApiResponse`


بالمناسبة تاني بردو الـ Class المسئول عنه `[ApiController]` هو `ApiBehaviorOptions`

بالشكل ده:
```cs
public class ConfigureApiBehaviorOptions:IConfigureOptions<ApiBehaviorOptions>
{
    public void Configure(ApiBehaviorOptions options)
    {
        throw new NotImplementedException();
    }
}
```

هنكتب بقى اللوجيك أنه هيمسك الموديل ستيت كل الـ Invalid اللي اتفقنا أنهم ديكشنري ويعرضهم في الأوبجيكت بتاعنا مع ايرور ماسج عامة

بالشكل ده:
```cs
public class ConfigureApiBehaviorOptions : IConfigureOptions<ApiBehaviorOptions>
{
    public void Configure(ApiBehaviorOptions options)
    {
        options.InvalidModelStateResponseFactory = context =>
        {
            var errors = context.ModelState
                .Where(ms => ms.Value?.Errors.Count > 0)
                .SelectMany(kvp => kvp.Value!.Errors.Select(e =>
                    $"{kvp.Key}: {(string.IsNullOrEmpty(e.ErrorMessage) ? e.Exception?.Message ?? "بيانات غير صالحة" : e.ErrorMessage)}"
                ))
                .ToList();

            var response = new ApiResponse<object>(false, "حدث خطأ ما", default, errors);

            return new BadRequestObjectResult(response);
        };
    }
}
```

وتعرف التعديل للـ DI:
```cs
builder.Services.ConfigureOptions<ConfigureApiBehaviorOptions>();
```

وكدا بقى شكل الإيرور مبروك عليك:
```json
{
  "success": false,
  "message": "حدث خطأ ما",
  "data": null,
  "errors": [
    "LastName: Last Name At Least 2 Chars",
    "FirstName: First Name At Least 2 Chars"
  ]
}
```

****
أتمنى يكون الشرح مفيد وشكرا