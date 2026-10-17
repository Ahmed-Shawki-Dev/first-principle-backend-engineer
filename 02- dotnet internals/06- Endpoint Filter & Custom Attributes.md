## Endpoint Filters
دي ببساطة Filters بتتحط على أي Endpoint الميزة فيها أنها سهلة ومناسبة للـ Modern Endpoints ببساطة تقدر بيها تحط زي فلتر أو شروط سواء Validation مثلا أو قاعدة معينة تتطبق على الاندبوينت دي
الميزة فيها أنها بتكون قادرة تشوف الـ Endpoint وتعدل فيه بالكامل بشكل مخصوص عكس الـ Middlewares اللي بس بتشتغل على الـ Streams الرايحة للريسبونس وهي راجعة للـ User لأنه بيجيلها JSON وعشان تتحكم فيها محتاج تفكه وحوارات

الـ **Endpoint Filters** شغالة **FILO** يعني زي الـ Pipline بالضب المهم ممكن نستعملها بكذا شكل
أول شكل البسيط خالص الدايركت
```cs
app.MapGet("/colors/{color}", (string color) => $"Color Is: {color}")
.AddEndpointFilter(
async (context, next) =>
{
string color = context.GetArgument<string>(0);

if (color == "Red")
{
return Results.Problem("Red Not Allowed");
}
return await next(context);
}
);
```

هنا أنت بتعمل فاليديشن دايركت الكونتيكست الموجود ده من نوع `EndpointFilterInvocationContext` والـ next من نوع `EndpointFilterDelegate` زي ما أنت شايف هي بردو **Chain** ماشية بترتيب **FILO**
وممكن بردو تعمل كلاس منفصل تحط في اللوجيك وتربطه بالـ Endpoint
```cs
namespace LifetimeLab;

public class FirstFilter : IEndpointFilter
{
    private readonly ILogger _logger;

    public FirstFilter(ILoggerFactory loggerFactory)
    {
        _logger = loggerFactory.CreateLogger<FirstFilter>();
    }

    public async ValueTask<object?> InvokeAsync(
        EndpointFilterInvocationContext context,
        EndpointFilterDelegate next
    )
    {
        var filter = context.GetArgument<string>(0);
        _logger.LogInformation($"The Endpoint Is Work On {filter}");
        return next(context);
    }
}
```

وبعدها تربطه كدا:
```cs
app.MapGet(
        "/{name}",
        (string name) =>
        {
            app.Logger.LogInformation("Endpoint");
            return Results.Ok(name);
        }
    )
    .AddEndpointFilter<FirstFilter>();
```

وتقدر تضيفها للكنترولر عن طريق
```cs
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.MapControllers()
    .AddEndpointFilter(async (efiContext, next) =>
    {
        efiContext.HttpContext.Items["endpointFilterCalled"] = true;
        var result = await next(efiContext);
        return result;
    });

app.Run();
```

هتضاف لكل الـ Controllers والـ Endpoints اللي عندك!
****
## Custom Attributes
