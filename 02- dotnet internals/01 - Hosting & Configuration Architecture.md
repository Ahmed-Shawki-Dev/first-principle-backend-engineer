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
