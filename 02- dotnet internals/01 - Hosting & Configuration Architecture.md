## WebApplication vs WebApplicationBuilder
في الـ `program.cs` بيبقى متقسم لجزئين
- جزء الـ Builder وهو الجزء اللي بتهندل فيه كل حاجة بيحتاجها السيرفر قل ما يقفل الباب عن التعديلات وهنا بنستعمل **DI** بشكل اساسي
- جزء الـ App وهو الجزء اللي بتبدأ تشوف فيه الـ **Middlewares** وتتعامل مع الـ `HttpContext` بشكل كامل
### Middleware Order
- الـ **`UseDeveloperExceptionPage`:** بيتضاف أول واحد لو البيئة شغال `Development`.
- الـ **`UseRouting`:** بيتضاف لو عندك Endpoints متعرفة زي `MapGet`.
- الـ **`UseAuthentication`:** بيتضاف مباشرة بعد `UseRouting` لو مسجل خدمة الـ Auth.
- الـ **`UseAuthorization`:** بيتضاف بعد الـ Authentication لو مسجل خدمة الـ Authorization.
- الـ **Custom Middlewares & Endpoints:** الكود بتاعك بيتنفذ في النقطة دي.
- الـ **`UseEndpoints`:** بيتضاف في النهاية لقفل المسار.

هنا في كام موضوع مهم
لما تعمل Builder هو بيحط UseRouting و UseEndpoint بشكل تلقائي من غير تدخل منك نهائي
وبيخلي أي Middleware أنت تضيفه بعد كدا يتضاف بينهم

حتى Middlewares زي UseAuthentication و UseAuthorization بتتضاف بردو بشكل تلقائي بس في أغلب الوقت أحنا بنكتبهم لأن في أغلب الوقت بنضيف Middlewares بين الـ Auth والـ Authz

هنا القاعدة بقى لو عايز تضيف Middleware قبل الـ Routing لازم تكتب UseRouting بإيدك لأن الترتيب مهم

نفس الكلام المفروض اخر حاجة بتتحط هي UseEndpoint لو عايز تحط حاجة بعدها لازم تكتبها وتحط بعدها الـ Middleware بتاعك

****
## Configuration in .NET
الـ Configuration دي زي ثوابت مثلا الـ`DB Connection String` وهكذا والفكرة هنا أن الكونيكشن استرينج بتاع الداتا بيز بيختلف بين الـ **Hosts** المختلفة سواء *Development* أو *Stagging* أو *Production* فلازم .NET يكون عارف هو في أني **Environments**

![](https://res.cloudinary.com/du7j80lol/image/upload/v1791130699/obsidian/nlx4avhnpv7zkutoinox.png)

وفي تعريف للـ Configuration
> **Configuration** is a collection of values that controls application behavior without changing the actual code
### Configuration providers
ببساطة خالص أنت عندك كذا نوع من الـ **Configuration** زي الـ *JSON File* أو *Environment Variables* أو *CLI Arguments*

فكل واحدة ليها شكل مختلف ليها سلوك مختلف، وعشان نوحدهم كلهم ونتعامل معاهم زي بعض
قسمنا الموضوع لـ 3 طبقات:
- Configuration Source
- Configuration Provider
- IConfiguration

ببساطة الـ IConfiguration تعريفه:
> represents the application’s combined configuration. It gives the application a single abstraction over all registered providers.

![](https://res.cloudinary.com/du7j80lol/image/upload/v1791133081/obsidian/ey6kqkadihtdmpoz4plg.png)

الـ Source هو نوع الـ Configuration على حسب كل نوع لي Provider خاص بيه مسئول أنه يحوله للصيغة القياسية اللي بنتعامل معاها في الكود بشكل موحد في **IConfiguration** ودي طريقة أحسن بكتير عشان نفصل المسئوليات لو مكنش كدا كان عملنا `IF Conditions` كتير جوا IConfiguration عشان تهندل أي نوع فايل كونفجريشن وده غلط

فبغض النظر عن المصدر سواء JSON زي كدا:
```json
{
  "Payments": {
    "Stripe": {
      "Timeout": 30,
      "RetryCount": 3
    }
  }
}
```

أو environment variable زي كدا:
```env
Payments__Stripe__Timeout=30
```

أو أيا يكن كلهم في الآخر بيبقى شكلهم كدا `Payments:Timeout = 30`
![](https://res.cloudinary.com/du7j80lol/image/upload/v1791133809/obsidian/ppr6qfgp8qiiioc52qg2.png)
### Configuration precedence
لو عندك كذا Configuration Provider عندهم نفس الـ Key ففي ترتيب للأولوية أو precedence وهو زي اللي في الصورة:
![](https://res.cloudinary.com/du7j80lol/image/upload/v1791133949/obsidian/puunmeomobzryitkiama.png)
### .NET default configuration
دي أنواع الـ configuration الإفتراضية أو الأساسية في .Net
#### appsettings.json
حاجة زي كدا:
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```
ده بيكون الـ Config العام المشترك بين كل الـ Hosts أو الـ Environments
#### appsettings.{Environment}.json
هنا أنت بتعمل **Override** للملف اللي فوق وتحط عليه قيمة حسب الـ Environment اللي أنت فيها دلوقتي زي كدا
```json
// appsettings.Production.json
{
  "Payments": {
    "Timeout": 60
  }
}
```

والأنفايرونمنت بتتقرأ من الملف ده
It reads the `ASPNETCORE_ENVIRONMENT` variable, set in `launchSettings.json`
#### Environment Variables
دي قيم بتيجي من الـ OS أو من الـ Docker Container أو من الـ CI/CD Pipline
![](https://res.cloudinary.com/du7j80lol/image/upload/v1791135411/obsidian/xq8qmbplynvndrwjvlnm.png)

زي ما قولنا مهما كان شكلها هتتحول للـ Standard بتاع الـ .Net
لو عندك Docker Compose نفس الكلام:
```yaml
services:
  api:
    environment:
      Payments__BaseUrl: https://payments.example.com
      Payments__Timeout: 60
```
#### Command Line
دي بتكتبها في الـ CLI وأنت بتفتح الأبلكيشن `dotnet MyApp.dll --Payments:Timeout=90`
### IConfiguration

هو ببساطة **read-only abstraction** معمول عشان الـ application يقرأ الـ **effective configuration** من مكان واحد، بغض النظر عن الـ providers اللي جابوا القيم.

يعني بدل ما الـ application يعرف القيمة جت من JSON ولا Environment Variables ولا CLI، هو بس بيقرأ:

```csharp
configuration["Payments:Timeout"];
```
أو:
```csharp
configuration.GetValue<int>("Payments:Timeout");
```

وبكده الـ application بيتعامل مع **configuration موحدة** بدل ما يتعامل مباشرة مع كل source.

****
## Options Pattern
The options pattern uses classes to provide strongly typed access to groups of related settings.
![](https://res.cloudinary.com/du7j80lol/image/upload/v1791198867/obsidian/mqv4u5mkvcqdipjkdvab.png)
### Binding
هنشوف مثال بسيط زي
```json
"App": {
"Name": "Google",
"Description": "Google It",
"PhotoUrl": "Google.com/logo.svg"
}
```

أول حاجة بنعملها أننا نعمل **Class** يمثل الـ **Options** دي:
```cs
public sealed class AppOptions
{
public const string SectionName = "App";
public string Name { get; set; } = string.Empty;
public string Description { get; set; } = string.Empty;
public string PhotoUrl { get; set; } = string.Empty;
}
```

ثم بنعرفه في الـ `program.cs` في كذا طريقة بس دي أنضفهم:
```cs
builder.Services.AddOptions<AppOptions>().BindConfiguration(AppOptions.SectionName);
```

تقدر خلاص مادام حقنتها في الـ DI تقدر تستعملها في أي Service أو Controller بالشكل ده:
```cs
public class AppService(IOptions<AppOptions> appOptions)
```

هتقولي ليه مش `AppOptions` بس؟ هقولك ﻷن العقد اللي أنت عملت فوق أنت قولتله أنه يتعامل مع الداتا بتاعتك ك IOptions
### Consuming options
في 3 أنواع من الـ options pattern:
#### `IOptions<T>`
دي أكتر حاجة بتستخدم بسيطة جدا معتمدة على الـ Singleton Pattern بتسجلها في الكونستراكتور عشان الـ DI
```cs
app.MapGet("/health/payments", (IOptions<PaymentOptions> options) =>
    Results.Ok(new { provider = options.Value.BaseUrl }));
```

الـ `IOptions<T>` بتديك **static value** لو الـ Configurations اتغيرت في الـ runtime لازم ترستر الـ App عشان تشوف المتغيرات الجديدة
#### `IOptionsMonitor<T>`
هنا بقى لما الـ Configurations تتغير الأنترفيس دي في ساعتها بتتحدث وهي بردو Singleton بس هي بتجيب الـ CurrentValue لأنك عندك ConfigurationsProvider بيبقى مسئول أنه يراقب أي تغيير في الـ Configurations فلما يلقى تغيير الـ Monitor بيعرف وبيغيرها من غير ما تعمل Build يعني هو مش تغيير كود
```cs
app.MapGet(
"/",
(IOptionsMonitor<AppOptions> appOptions) =>
{
return appOptions.CurrentValue;
}
);
```
#### `IOptionsSnapshot<T>`
هي متسجلة كـ Scoped فبتاخد Snapshot من الريكوست في لحظة البداية وبتفضل محافظة عليها لحد ما الريكوست يموت

هنا بنتعامل بمبدأ الـ Request تخيل ريكوست حصل وأنت في الـ Configurations عندك 3 حاجات فأنت خدت حاجتين وغيرت التالتة
هنا لو عامل Snapshot مش هتاخد القيمة التالتة الجديدة هتاخد القديمة عشان الريكوست ميكنش متلغبط بين قيم جديدة وقديمة

لو Monitor كانت لقت اي تغيير عملته وخلاص أنما هنا لا بتركز
لما تيجي بقى تعمل ريكوست تاني وغيرت هي هتتغير بس
### Configuration Validation
مش هنشرح فكرة الـ Validation أكيد عارفها بس يعني تخيل أنت عايز تضبط الـ Config مش عايزه يبقى عشوائي يعني لو حاطط رقم سالب في Timeout مثلا يضرب Validation Error وهكذا

عندك كذا طريقة أشهرها والمعروفة هي الـ
#### Data Annotations
حاجة زي كدا
```cs
[Required, Url]
public string PhotoUrl { get; set; } = string.Empty;
```
ولما بتيجي تحط الـ Option بتقوله خد بالك في Validation متنساش
زي كدا:
```cs
builder.Services.AddOptions<AppOptions>().BindConfiguration(AppOptions.SectionName).ValidateDataAnnotations();
```

وضيف السطر ده
```cs
.ValidateOnStart()
```
لو عايزه ميفتحش خالص لو لقى الـ Config Validation غلط
هنا ماشية بمبدأ Fail-Fast يعني بتشوف الغلط في الـ Config لو في غلط تقفل السيرفر كله متفتحش اصلا السيرفر غير لما الغلط يتصلح
****
## Service Registration And Finally Build
كل اللي كنا بنعمله في الـ Configurations وتضيفه والـ Services اللي كنا بنعملها كنا بنسجلها في الـ `builder`
```cs
var builder = WebApplication.CreateBuilder(args);
```
يعني بنحقن الـ DI عشان لما الـ Application يشتغل أو يحصله **Build** يكون شايف كل حاجة احنا حقناها في الـ **Runtime**

لما بتحط كل الـ DI والـ Services والـ Options وكل حاجة تخصك بعدها بتحط السطر ده
```cs
var app = builder.Build();
```
ومن هنا البرنامج بيختلف تماما
بتبدأ مرحلة الأكشن كل حاجة بتشتغل ومفيش تعديل خلاص كله شغال
الـ Middlewares هنا بتبدأ تشوف شغلها وبترتب الـ Middlewares وتحط ميدلويرز جديدة ويستقبل `HttpContext` مختلفة من كل الـ Clients وهكذا

ودي الرحلة كاملة من البداية للنهاية تقريبا بشكل مختصر