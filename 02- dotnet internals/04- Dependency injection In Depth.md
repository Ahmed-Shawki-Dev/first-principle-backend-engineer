الشرح اللي فات كان الفكرة العامة اللي الكل يعرفها بشكل بسيط، هنا أحنا حرفيا هنعمل الـ DI من الصفر خالص ونفهم الميكانيزم بتاعه بالكامل وأزاي مهندسين .Net عملوه
اتكلمنا عن:
1. أزاي الـ DI بيشتغل من تحت
2. الفرق بين التجميع والتنفيذ
3. أزاي السيرفس بيتخلقلها Scope حسب نوعا لما تتنفذ
4. أزاي الـ Scope بيتمسح لما يتنفذ
## `IServiceCollection`
دي بالمختصر هي Register Services
بإختصار دي الـ Interface اللي .Net في الـ WebApi معتمدة عليها عشان تعمل Register للـ Services اللي الأبلكيشن محتاجها عشان يكون في loose coupling
فببساطة الـ Key Features بتاعتها:
1. **Service Registration**: Register Services in DI Container With Different Lifetime Like:
	- `AddTransient`: New Instance Every Time It Requested
	- `AddScoped`: Single Instance Is Created Per Request
	- `AddSingleton`: Signle Instance Is Created And Shared Throughout The Entire Application Lifetime
2. **Configuration and Options**: تقدر تضيف الكونفجريشن وكل ده بردو بيها زي ما بيحصل في الـ Options Pattern
3. **Custom Services**: لو عايز تعمل سيرفس خاصة بيك وتخصصها هنا
```cs
services.AddSingleton<ICustomService>(sp => new CustomService(Configuration["ConnectionString"]));
```
### How `IServiceCollection` Works:
هي من الآخر تعتبر حاوية أو كوليكشن جواها كل الـ registered services اللي كل واحد فيهم عبارة عن `ServiceDescriptor` ودي هي أقل وحدة للـ DI يعتبر هو بيتسجل ك `ServiceDescriptor`

ببساطة لما أنت بتكتب `AddScoped` أو أيا كان اللي هتكتبه وتعرف سيرفس هو بياخد اللي أنت كاتبه يحوله لـ `ServiceDescriptor` اوبجيكت ويضيفها لنفسه اللي هو `IServiceCollection`
## `ServiceDescriptor`
أحنا عمالين نتكلم عنه فوق بس هنا هنشرحه بالتفصيل

هو ببساطة اللي بيوصف أزاي نـ Manage الـ Service في الـ DI زي مثلا:
1. **Service Type**
اللي هو الانترفيس
2. **Implementation Type**
يعني بياخد الـ Class الحقيقي فعلا
3. **Service Lifetime**
وبياخد اللايف تايم بتاع الـ Service زي ما حددنا فوق سينجلاتون وهكذا

طب هو أزاي بيشتغل؟
```cs
var descriptor = new ServiceDescriptor(  
    typeof(IMyService),   // Service Type  
    typeof(MyService),    // Implementation Type  
    ServiceLifetime.Scoped // Lifetime (Scoped, Singleton, or Transient)  
);
```
## Relationship Between ServiceDescriptor and IServiceCollection
ببساطة الكوليكشن هو عبارة عن كونتينر لكل الـ ServiceDescriptor أنت لما بتعمل أي حاجة زي AddScoped وهكذا هو بيروح يعمل ServiceDescriptor جديد

يعني من الآخر:
1.`IServiceCollection` is the **container** used to register all services in an application.
2.`ServiceDescriptor` acts as the **blueprint** that defines how each service should be created and managed, specifying the **type**, **implementation**, and **lifetime** (`Transient`, `Scoped`, or `Singleton`).
****
## What is `IServiceProvider`?
أحنا بعد ما عملنا الـ Collection خلاص محتاجين ننفذ بقى
محتاجين نوزع الـ Services على الـ Controllers أو الـ Services اللي عايزاها
فدي وظيفة الـ `IServiceProvider` الموضوع ده بيتم عن طريق الميثود اللي جوا الـ `IServiceCollection` اللي هي `BuildServiceProvider` اللي لما بتكتبها بتبدأ مرحلة الـ Resolve للسيرفسيس لما تحتاجها فعلا

فمن الآخر:
- `IServiceCollection` is used to **register services**.
- `IServiceProvider` is used to **resolve services**.
```cs
var serviceProvider = services.BuildServiceProvider();   
var myService = serviceProvider.GetService<IMyService>(); // Resolving the registered service
```
## Benefits of Using IServiceCollection and IServiceProvider

- **Loose Coupling:** Reduces dependency on implementations, enhancing flexibility.
- **Testability:** Simplifies mocking services for unit tests.
- **Maintainability:** Centralized registration allows for easy changes without widespread code updates.
- **Flexibility:** Supports managing service lifecycles based on requirements.
### Final Example
```cs
IServiceCollection services = new ServiceCollection();

ServiceDescriptor scissorsService = new ServiceDescriptor(
typeof(IPizzaCutter),
typeof(ScissorsPizzaCutter),
ServiceLifetime.Scoped
);

ServiceDescriptor pizzaCutterControllerService = new ServiceDescriptor(
typeof(PizzaCutterController),
typeof(PizzaCutterController),
ServiceLifetime.Transient
);

services.Add(scissorsService);

services.Add(pizzaCutterControllerService);

IServiceProvider serviceProvider = services.BuildServiceProvider();

var controller = serviceProvider.GetRequiredService<PizzaCutterController>();

controller.CutPizza();
```

والشكل المختصر بما أننا قولنا ان الـ Collection بيعمل Descriptor بشكل تلقائي هو أننا نكتب:
```cs
using Microsoft.Extensions.DependencyInjection;

IServiceCollection services = new ServiceCollection();

services.AddScoped<IPizzaCutterService, SuccorCutter>();
services.AddTransient<PizzaCutterController>();

IServiceProvider serviceProvider = services.BuildServiceProvider();

var pizzaCutterController = serviceProvider.GetRequiredService<PizzaCutterController>();

pizzaCutterController.CutPizza();
```
****
## Registration vs Resolution
دول المرحلتين اللي بيحصلوا جوا الـ DI أول مرحلة هي مرحلة الـ Registration دي بتحصل في **`IServiceCollection`**
أنت بتقول للفريمورك لما حد يطلب الـ Service اديله كذا وبـ Lifetime كذا

في المرحلة دي دي مجرد **Blueprint** مفيش أي **Object** بيتكون أنت هنا بس بتعمل تسجيل مش بتنفذ

عكس الـ Service Resolution هنا أنت بعد ما بتعمل `BuildServiceProvider()` البروفايدر بيبدأ يمسك ال **Descriptors** ويبدأ عملية البحث يشوف مين عايز ايه بيبدأ يفك الـ **Dependency Graph** يشوف مين معتمد على مين معتمد على مين وأزاي وأول ما يوصل لنهاية الشجرة يبدأ يعمل `new` وينفذ أوبجيكت أوبجيكت من الآخر للأول عشان يسلمك الكنترولر جاهز دايركت
## Dependency Graph
لما الأبلكيشن يكبر بتلاقي أن في حاجة معتمدة على حاجة معتمدة على حاجة وهكذا الموضوع مش بيكون بسيط
فالدوت نت بترسم Graph يوضح العلاقات ويجيب العلاقات بين الـ Services وبعضها
فلما تعوز مثلا تجيب `OrderController` هو بيعمل **Graph Traversal** ويبدأ ينزل للـ `leafs` وينفذ new لكله من الآخر للأول
والعملية أنه ينفذ **new** للأوبجيكتس اللي يشوفها دي اسمها `Object Construction`
## Root Service Provider vs Service Scopes
لما بتعمل
```cs
IServiceProvider serviceProvider = services.BuildServiceProvider();
```

بيتكون عندنا مستويين للـ Providers:
1. **Root Service Provider**
- بيعيش بطول عمر الأبلكيشن كامل
- مسؤول فقط عن الـ **Singletons**
- لو طلبت منه خدمة `Scoped` مباشرة، الـ Root هيمسك فيها ومش هيرميها للـ GC، فبالتالي هتعمل **Memory Leak** بشع ومشاكل Data Stale.
- بينادي `Dispose()` على الـ Singletons المسجلة عنده في حالة لما السيرفر كله ييجي يقفل

2. **Service Scope (`IServiceScope`)**
- دي الحل لمنع الـ Memory Leaks، وبتعيش فترة مؤقتة جدا مع الـ HTTP Request مثلا
- الـ Framework بينادي `rootProvider.CreateScope()` فيعمل **Child Service Provider** خاص بالـ Request ده بس.
- بيكريت جواه الـ **Scoped** والـ **Transient** Services الخاصة بالطلب ده.
3. **Scope Creation & Disposal**
- **الـ Creation:** عشان يفتح حاوية مؤقتة للـ Request عشان ينعزل عن بقية Requests التانية.
- **الـ Disposal:** أول ما الـ Request يخلص، الـ Framework بينادي `scope.Dispose()` فبيحصل الآتي
	- الـ Scope بيلف على كل الأوبجيكتس اللي اتكريتت جواه بتعمل `IDisposable`.
	- بينادي `Dispose()` بتاعتهم عشان يقفل الـ DB Connections والـ Unmanaged Resources.
	- بيمسح الـ Child Provider ويرمي كل اللي جواه للـ Garbage Collector لضمان نظافة الميموري.