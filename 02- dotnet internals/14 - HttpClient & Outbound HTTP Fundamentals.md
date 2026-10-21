طول عمرنا متعودين أن الـ Server هو اللي بيستقبل هو يعتبر الـ Inbound بس مع الوقت هتلاقي نفسك محتاج حاجات السيرفر هو اللي يكون Client فيها ويكلم سيرفر تاني وهنا هو بيعمل **Outbound Http Request**

لو اشتغلت JS فأنت متعود أنك تستعمل fetch بس عشان الفيتش بتشتغل على براوزر كل Client بشكل مستقل فمفيهاش تعقيد كبير إنما أنت كسيرفر لو في API بيخليك تروح تضرب ريكوست لسيرفر تاني فأنت ممكن تتخيلها أنك تفتح fetch لأكتر من 1000 كلاينت في نفس اللحظة مثلا فهنا التحدي ومشاكل كتير

وهنا لازم نفهم أن أي ريكوست طالع بيحتاج يفتح TCP Connection والأول بيحصل TCP Handshake ووراها TLS Handshake لو شغال HTTPS ودي شرحناها هنا [[01- Request Lifecycle Mental Model]] والتكلفة دي بتاخد وقت وموارد

وعشان تفتح الـ Connection ده الـ OS بيخصص بورت محلي **Ephemeral Port** من البورتات المتاحة عنده وده ينقلنا لمشكلة ليمت الـ Sockets والـ Ports لأن الـ OS عنده حدود للموارد المتاحة وعدد البورتات المؤقتة المستخدمة في الاتصالات اللي جاية

فهنا لما تعمل `HttpClient` السيرفر بيعتبر كـ Client فهيكون معرض لنفس المشاكل ودي مشكلة لأن لو فتحت TCP Connections جديدة مع كل ريكوست وقفلتها بسرعة الاتصالات دي ممكن تفضل في حالة `TIME_WAIT` داخل الـ OS وده مع الضغط العالي ممكن يسبب**Port Exhaustion**

وكمان لو عملت العكس واحتفظت بالـ Connections لفترة طويلة من غير إدارة مناسبة ممكن تلبس في مشكلة **Stale DNS** لأن الـ IP بتاع السيرفر الخارجي ممكن يتغير والاتصالات الموجودة مش هتكتشف التغيير ده لوحدها

وبالمناسبة ممكن تعتبر البديل الأساسي للـ fetch في C# هو HttpClient وده الموضوع بتاعنا النهارده

هناخد توبيكس زي
- How to use HttpClient correctly
- Why creating HttpClient repeatedly is a problem
- How to simplify configuration with IHttpClientFactory
- How to configure Named Clients
- How to configure Typed Clients
- How SocketsHttpHandler manages connections
- Why you should avoid Typed Clients in singleton services
- When to use which option
- DelegatingHandler => Like axios interceptor
- TokenCancellations

## What Is HttpClient in .NET
من الآخر الـ `HttpClient` هي الطريقة اللي بتبعت بيها HTTP Requests لسيرفر تاني وليها كل خصائص الـ `HTTP` اللي كلنا عارفينها نفس الـ Methods ونفس الـ Headers ونفس الـ Status Codes وكل حاجة تقريبا وليها دعم للـ JSON وهكذا

هي سهلة جدا في الاستعمال بس سهل جدا تستعملها غلط لأنك ممكن تكتب كود شغال في التست البسيط لكن مع الضغط في الـ Production تظهر مشاكل ملهاش علاقة بشكل الريكوست نفسه لكن بطريقة إدارة الاتصالات

فبما أننا فهمنا إيه هي الـ `HttpClient` أزاي بقى نستعملها صح
****
## The Naive Way
أبسط وأول حاجة بتيجي في دماغ أي حد يكتب C# أنه يدخل جوه الـ Service ويعمل
`using var client = new HttpClient()`
يضبط الـ `Headers` والـ `BaseAddress` ويبعت الريكوست وبس

الكود بيبان شغال وزي الفل في التست البسيط بس في الـ **Production** ممكن يعمل مشاكل كبيرة

ليه بقى لأن الـ `HttpClient` مش المفروض يتعمله Create ويتقفل مع كل ريكوست بالشكل ده لأن إنشاء الـ Client جديد كل مرة ممكن يؤدي لإنشاء **Connection Pools** جديدة وفتح اتصالات كتير بشكل متكرر

وده مثال يوضح الفكرة

```cs
public class GitHubService
{
    private readonly GitHubSettings _settings;

    public GitHubService(IOptions<GitHubSettings> settings)
    {
        _settings = settings.Value;
    }

    public async Task<GitHubUser?> GetUserAsync(string username)
    {
        using var client = new HttpClient();

        client.DefaultRequestHeaders.Add("Authorization", _settings.GitHubToken);
        client.DefaultRequestHeaders.Add("User-Agent", _settings.UserAgent);
        client.BaseAddress = new Uri("https://api.github.com");

        GitHubUser? user = await client
            .GetFromJsonAsync<GitHubUser>($"users/{username}");

        return user;
    }
}
```

المشكلة هنا مش في إنك بتستخدم using لكن في إنك بتنشئ `HttpClient` جديد لكل عملية وبتتخلص منه بعدها

لما يحصل ضغط كبير والـ Application يفضل يفتح اتصالات جديدة ويقفلها بسرعة ممكن تلاقي عدد كبير من الاتصالات في حالة `TIME_WAIT` وده يساهم في استهلاك الـ **Ephemeral Ports** لحد ما يحصل **Port Exhaustion** وساعتها الريكوستات الجديدة ممكن تفشل

طب ما نعمل العكس ونخلي الـ `HttpClient` واحد ثابت ونستخدمه طول عمر الابلكيشن

الحل ده أحسن من إنك تعمل Client جديد مع كل ريكوست لكنه برضه محتاج تفهم مشكلة تانية وهي الـ **Stale DNS**

لأن الـ `HttpClient` ممكن يحتفظ بالـ Connections الموجودة في الـ Pool لفترات طويلة والاتصالات دي مش بتعمل **DNS Resolution** من جديد مع كل ريكوست فلو الـ IP بتاع السيرفر الخارجي اتغير ممكن تفضل تستخدم الاتصالات القديمة بدل ما تكتشف التغيير

فإحنا محتاجين حل يخلينا نعيد استخدام الاتصالات عشان نتجنب استنزاف البورتات وفي نفس الوقت نقدر نجدد الاتصالات مع الوقت عشان نتعامل مع تغييرات الـ DNS

وهنا بقى ييجي دور `IHttpClientFactory`
****
## The Smart Way Using IHttpClientFactory
بدل ما ندير الـ Lifetime بتاع الـ `HttpClient` والـ Connections بنفسنا بنستخدم `IHttpClientFactory`

هنا بقى فيه تفصيلة لازم تفهمها الـ HttpClient كأوبجيكت هو خفيف نسبيا ومش هو الجزء المكلف فعليا الجزء المهم هو الـ `HttpMessageHandler` وده اللي بيدير الاتصالات والـ **Connection Pool**

فاللي بيعمله الـ Factory إنه بيقدر ينشئ HttpClient جديد وخفيف كل ما تطلبه لكنه بيعيد استخدام الـ `HttpMessageHandler` بين الـ Clients المختلفة طول ما الـ Handler لسه صالح للاستخدام

وبكده مش محتاج تحتفظ بنفس الـ `HttpClient` طول عمر الابلكيشن عشان تستفيد من إعادة استخدام الاتصالات

وكمان الـ Factory بيدير عمر الـ **Handlers** وبيجددها بشكل دوري عشان الاتصالات القديمة متفضلش مستخدمة للأبد وده بيساعد على التعامل مع تغييرات الـ DNS

وده مثال بسيط يوضح الفكرة

```cs
app.MapGet("/todo/{id}", async (IHttpClientFactory factory, int id) =>
{
    var client = factory.CreateClient();

    client.BaseAddress = new Uri("https://dummyjson.com/");

    var jsonString = await client.GetStringAsync($"todo/{id}");

    return Results.Content(jsonString, contentType: "application/json");
});
```

بس لازم تفعل الـ Service في الـ DI Container

```cs
builder.Services.AddHttpClient();
```

كده تقدر تطلب `HttpClient` من الـ Factory من غير ما تضطر تدير الـ **Handler** والـ **Connection Pool** بنفسك

بس لسه عندنا مشكلة تانية

تخيل إن عندك أكتر من API بتتعامل معاه وكل واحد ليه BaseAddress مختلفة و Headers مختلفة و Authentication مختلفة هتلاقي نفسك بتكرر نفس الإعدادات في أماكن كتير وده مش أحسن تنظيم للكود

وده ينقلنا للخطوة اللي بعدها

## Reducing Code Duplication With Named Clients

الـ `IHttpClientFactory` حل مشكلة إدارة الـ Connections لكن لسه محتاجين طريقة ننظم بيها إعدادات كل API

زي في الفرونت مثلا لما تعمل **Axios Instance** مخصوصة لكل API وتحط فيها الـ BaseURL والـ Headers والإعدادات المشتركة بدل ما تكررها في كل ريكوست

نفس الفكرة هنا باستخدام **Named Clients**

الفكرة إنك بتسجل Client باسم معين في الـ `Program.cs` وتحط فيه الإعدادات المشتركة بتاعته ولما تحتاجه بتطلبه من الـ Factory بالاسم ده

يعني مثلا لما تكتب

```cs
factory.CreateClient("github")
```

الـ Factory هيرجعلك Client بالإعدادات اللي أنت سجلتها تحت اسم github

وده مثال على تسجيل الـ Client

```cs
builder.Services.AddHttpClient("github", (serviceProvider, client) =>
{
    var settings = serviceProvider
        .GetRequiredService<IOptions<GitHubSettings>>().Value;

    client.BaseAddress = new Uri("https://api.github.com");
    client.DefaultRequestHeaders.Add("Authorization", settings.GitHubToken);
    client.DefaultRequestHeaders.Add("User-Agent", settings.UserAgent);
});
```

وبعد كده تقدر تستخدمه جوه الـ Service بالشكل ده

```cs
public class GitHubService
{
    private readonly IHttpClientFactory _factory;

    public GitHubService(IHttpClientFactory factory)
    {
        _factory = factory;
    }

    public async Task<GitHubUser?> GetUserAsync(string username)
    {
        using var client = _factory.CreateClient("github");

        return await client.GetFromJsonAsync<GitHubUser>($"users/{username}");
    }
}
```

كده فصلنا الإعدادات عن الكود اللي بيبعت الريكوست وبقينا نقدر نغير إعدادات الـ Client من مكان واحد

بس لسه فيه حاجة ممكن تتحسن

أنت لسه محتاج تعمل Inject للـ `IHttpClientFactory` وتطلب الـ Client بالاسم بنفسك وكل مرة تكتب اسم زي `"github"` وده اسمه **Magic String** لأن الكود بيعتمد على نص عادي بدل ما يعتمد على Dependency واضحة في الـ Type System

طب ما نربط الـ `HttpClient` بالخدمة نفسها بدل ما نفضل نطلبه بالاسم

وده اللي بيعمله الـ Typed Clients
****
## Replacing Named Clients With Typed Clients
الـ Typed Clients بتخليك تربط `HttpClient` بخدمة معينة زي `GitHubService` بحيث الإعدادات بتاعته تتسجل في الـ `Program.cs` والخدمة تستقبل الـ `HttpClient` جاهز من خلال الـ Constructor

وبكده بدل ما تعمل Inject للـ Factory وتكتب اسم الـ Client بنفسك بتطلب الـ `HttpClient` مباشرة من الـ DI Container عن طريق الـ Service اللي أنت محتاجها

وده مثال على تسجيل الـ Typed Client

```cs
public class DummyJsonService(System.Net.Http.HttpClient client)
{
    public async Task<string> GetTodoByIdAsync(int id)
    {
        return await client.GetStringAsync($"todos/{id}");
    }
}
```

وفي البروجرام

```cs
builder.Services.AddHttpClient<DummyJsonService>(client =>
{
    client.BaseAddress = new Uri("https://dummyjson.com/");
    client.Timeout = TimeSpan.FromSeconds(10);
});
```

وهنيجي نستخدمها كده

```cs
app.MapGet("/todos", async (DummyJsonService dummyService) =>
{
    var tasks = Enumerable.Range(1, 200).Select(id => dummyService.GetTodoByIdAsync(id));

    var results = await Task.WhenAll(tasks);

    return Results.Ok(results.Length);
});
```

لاحظ إننا مش محتاجين نسجل `DummyJsonService` بشكل منفصل لأن `<AddHttpClient<DummyJsonService` بتسجلها كـ **Transient** وبتربطها بالـ `HttpClient` اللي اتظبط في التسجيل

وبكده بقى عندنا Service واضحة بتعبر عن التعامل مع **DummyJson** وإعدادات الـ HTTP بتاعتها متجمعة في مكان واحد

بس قبل ما نكمل لازم نفهم حاجة مهمة

الـ Named Clients والـ Typed Clients مش طريقتين مختلفتين لإدارة الـ Connections لأن الاتنين بيعتمدوا على `IHttpClientFactory` في إدارة الـ Handlers والـ Connection Pools الفرق الأساسي بينهم هو طريقة تنظيم الكود والوصول للـ Client

الـ Named Client مناسب لما تحتاج تختار Client بالاسم وقت التشغيل أو تتعامل مع مجموعة Clients بشكل مرن أما الـ Typed Client فمناسب لما يكون عندك Service مسؤولة عن التعامل مع API معينة وعايز تعتمد على Constructor Injection بدل الـ Magic Strings

طب إيه اللي بيحصل تحت الغطاء عشان كل الكلام ده يشتغل
****
## Understanding HttpMessageHandler
لحد دلوقتي إحنا عرفنا إن الـ Factory بيدير الـ Handlers وإن الـ Handlers هي اللي بتدير الاتصالات بس لسه محتاجين نفهم إيه هي الـ Handlers دي أصلا

الـ `HttpClient` مش هو اللي بينفذ كل تفاصيل الاتصال بالشبكة بنفسه لكنه بيعتمد على `HttpMessageHandler` عشان يعالج الريكوستات ويبعتها ويرجع الاستجابات

وفيه أكتر من **Handler** حسب الوظيفة المطلوبة لكن لما نتكلم عن الـ Handler الأساسي المسؤول عن تنفيذ HTTP Requests في .NET فهنا عندنا `HttpClientHandler` و `SocketsHttpHandler`

الـ `HttpClientHandler` هو الـ Handler التقليدي اللي بتتعامل معاه في حالات كتير لو محتاج تضبط إعدادات زي الـ Cookies والـ Proxy والشهادات

أما `SocketsHttpHandler` فهو التطبيق الحديث اللي بيعتمد عليه .NET في إدارة اتصالات HTTP وبيوفر تحكم أكبر في تفاصيل زي الـ Connection Pooling والـ DNS والاتصالات نفسها

وفي إصدارات .NET الحديثة الـ `HttpClientHandler` بيستخدم `SocketsHttpHandler` داخليا في التنفيذ المعتاد لكن ده مش معناه إن الاتنين نفس الـ Type أو إنك لازم تستخدم الاتنين بنفسك

طيب إحنا عرفنا إن `SocketsHttpHandler` بيدير الـ **Connection Pool** إزاي بقى نتحكم في عمر الاتصالات دي
****
## SocketsHttpHandler and DNS Refresh
إحنا قلنا إن إعادة استخدام الاتصالات بتحسن الأداء وبتقلل تكلفة إنشاء Connections جديدة لكن لو الاتصال فضل موجود لفترة طويلة ممكن يفضل مربوط بالـ IP القديم حتى لو الـ DNS رجع IP جديد للسيرفر الخارجي

عشان كده عندنا خاصية اسمها `PooledConnectionLifetime` بتحدد أقصى عمر للاتصال داخل الـ Pool قبل ما يبقى مؤهل للاستبدال باتصال جديد

وده مثال على ضبطها

```cs
builder.Services
    .AddHttpClient<MyService>()
    .ConfigurePrimaryHttpMessageHandler(() => new SocketsHttpHandler
    {
        PooledConnectionLifetime = TimeSpan.FromMinutes(2)
    });
```

كده أنت بتحدد عمر الاتصالات الموجودة في الـ Pool بدقيقتين وبعد ما الاتصال يوصل للعمر ده مش هيتم استخدامه لاتصالات جديدة وبدل ما يفضل موجود للأبد هيتم إنشاء اتصال جديد عند الحاجة

ولما الاتصال الجديد يتعمل بيقدر يعمل **DNS Resolution** من جديد وياخد الـ IP الحالي للسيرفر

بس خد بالك إن ده مش معناه إن كل دقيقتين كل الاتصالات بتتقفل وتتفتح في نفس اللحظة لأن الخاصية بتحدد عمر الاتصال نفسه مش عمر الـ Handler كله

وبرضه ده مش معناه إن كل DNS Change هيتم اكتشافه لحظيا لأن الاتصال الجديد مش بيتعمل إلا عند الحاجة وليه هو كمان وقت التنفيذ الخاص بيه

ولو أنت بتستخدم `IHttpClientFactory` بالفعل فهو بيدير عمر الـ Handlers بشكل دوري وده بيساعد على تجديد الاتصالات والتعامل مع تغييرات الـ DNS أما لو عندك سبب يخليك تتحكم في عمر الاتصالات بشكل مباشر فتقدر تستخدم `SocketsHttpHandler` وتضبط `PooledConnectionLifetime`

وهنا لازم نفرق بين حاجتين `PooledConnectionLifetime` بتتحكم في عمر الاتصالات أما `HandlerLifetime` اللي بتتضبط من خلال الـ Factory فبتتحكم في المدة اللي الـ Factory بيحتفظ فيها بالـ Handler قبل ما يستبدله

الاتنين بيساعدوا في التعامل مع الاتصالات طويلة العمر لكن كل واحد بيشتغل على مستوى مختلف
****
## Why You Should Avoid Typed Clients in Singleton Services
بعد ما فهمنا الـ Typed Clients والـ DI Lifetimes لازم ناخد بالنا من نقطة مهمة جدا

إحنا قلنا إن الـ Typed Client بيتسجل تلقائيا كـ Transient يعني كل مرة الـ DI Container يحتاج ينشئه ممكن يطلع Instance جديدة لكن ده مش معناه إن الـ `HttpMessageHandler` نفسه بيتعمل من جديد كل مرة لأن الـ Factory هو اللي بيدير الـ Handlers

المشكلة بتظهر لما تحقن Typed Client جوه Singleton Service

لأن الـ Singleton بيتعمل مرة واحدة وبيفضل موجود طول عمر الـ Application وبالتالي هيحتفظ بنفس الـ Typed Client اللي اتحقن جواه حتى لو المفروض الـ Typed Client يكون Transient

وده ممكن يخلي الـ `HttpClient` والـ Handler المرتبط بيه يفضلوا مستخدمين لفترة أطول من اللي أنت متوقعها وده يأثر على الاستفادة من تجديد الـ Handlers اللي الـ Factory بيديرها عشان يتعامل مع تغييرات الـ DNS

عشان كده لو عندك Singleton محتاج يبعت HTTP Requests فممكن تستخدم `IHttpClientFactory` مباشرة وتطلب منه Client وقت الحاجة بدل ما تحتفظ بواحد من الـ Typed Clients طول عمر الـ Singleton

لكن ده مش معناه إن أي Typed Client جوه Singleton هيعمل مشكلة في كل الحالات لأن الموضوع بيعتمد على طريقة استخدامه وإدارة الاتصالات عندك لكنه اختيار محتاج انتباه لو بتعتمد على تجديد الـ Handler بشكل دوري

## When to Use Which Option

بعد ما فهمنا الطرق المختلفة لإدارة الـ HttpClient نقدر نحدد إمتى نستخدم كل واحدة

لو عندك API بسيطة ومحتاج تبعت منها Requests تقدر تستخدم IHttpClientFactory مباشرة وتعمل CreateClient وتضبط الإعدادات المطلوبة

لو عندك APIs متعددة وكل واحدة ليها إعدادات مختلفة زي الـ BaseAddress والـ Authentication والـ Headers فـ Named Clients بتخليك تجمع إعدادات كل API تحت اسم واضح وتطلبها وقت الحاجة

أما لو عندك Service مسؤولة عن التعامل مع API معينة وعايز تخلي الـ Dependencies واضحة وتبعد عن الـ Magic Strings فـ Typed Clients غالبا اختيار مناسب لأنك بتربط الـ HttpClient بالخدمة نفسها وبتخلي الإعدادات في مكان واحد

ولو عندك احتياج للتحكم في تفاصيل الاتصالات نفسها زي عمر الاتصالات داخل الـ Connection Pool فتقدر تضبط SocketsHttpHandler وتستخدم PooledConnectionLifetime حسب احتياجك

وفي النهاية أهم حاجة تفهمها إن المشكلة مش مجرد إنك تعمل HttpClient جديد أو تحتفظ بواحد ثابت لأن كل اختيار ليه طريقة صحيحة حسب السياق

الـ `IHttpClientFactory` بيساعدك تدير الـ Handlers والـ Connection Pools والـ Named Clients بتنظم إعدادات الـ HTTP والـ Typed Clients بتنظم طريقة اعتماد الخدمات على الـ HTTP Client أما `SocketsHttpHandler` بيديك تحكم أكبر في تفاصيل الاتصالات
****
## DelegatingHandler
الـ `DelegatingHandler` هو ببساطة عبارة عن Middleware مصغر خاص بالـ Outbound HTTP Requests يعني كود بيقف في السكة بين الـ `HttpClient` والنت وظيفته يفتش الريكوست وهو طالع ويعدل عليه أو يفتش الريسبونس وهو راجع وياخد منه داتا قبل ما يرجع للـ Controller بتاعك

السر كله إن أي `DelegatingHandler` بيكتور جوه ميثود واحدة اسمها `SendAsync` والميثود دي بتتنفذ على خطوتين خطوة قبل ما الريكوست يخرج وخطوة بعد ما الرد يرجع من النت

وده مثال يوضح كلاس الـ Handler نفسه

```cs
public class LoggingDelegateHandler : DelegatingHandler
{
    protected override async Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request, 
        CancellationToken cancellationToken)
    {
        // 1. كود بيتنفذ قبل ما الريكوست يخرج للنت (Request Interceptor)
        // تقدر تفتح أوبجيكت الـ request وتحط فيه أي Headers أو Tokens أنت عايزها
        request.Headers.Add("X-App-Name", "MyDotNetApp");
        Console.WriteLine($"Sending request to: {request.RequestUri}");

        // 2. السطر ده هو اللي بيودي الريكوست للنت وبيستنى الرد يرجع
        var response = await base.SendAsync(request, cancellationToken);

        // 3. كود بيتنفذ أول ما الريسبونس يرجع من النت (Response Interceptor)
        // هنا معاك أوبجيكت الـ response تقدر تقرأ الـ Status Code أو تقرأ الـ JSON نفسه
        Console.WriteLine($"Received response with status: {response.StatusCode}");

        // 4. بنسلم الرد النهائيات للـ App بتاعك
        return response;
    }
}
```

وعشان دوت نت تفهم إن الـ Handler ده موجد في السلسلة لازم تسجله الأول في الـ DI Container كـ Transient وبعدين تربطه بالـ Client بتاعك بالشكل ده

```cs
// 1. بنسجل الـ Handler في الـ DI
builder.Services.AddTransient<LoggingDelegateHandler>();

// 2. بنربطه بالـ HttpClient
builder.Services.AddHttpClient("dummy", client =>
{
    client.BaseAddress = new Uri("https://dummyjson.com/");
})
.AddHttpMessageHandler<LoggingDelegateHandler>();
```

كده أي ريكوست يخرج من الـ Client اللي اسمه dummy هيعدي أوتوماتيك على `LoggingDelegateHandler` يحط الـ Header ويطبع الـ Log وهو راجع من غير ما تضطر تكتب الكود ده جوه كل Service
****
## Timeouts and Cancellation Tokens
لما بتبعت Outbound Request لسيرفر خارجي أنت مش مضمون السيرفر ده هيرد في قد إيه وممكن يفضل معلق السيرفر بتاعك ومستنزف الـ Sockets والموارد

عشان كده بنحتاج نتحكم في زمن الريكوست أو نلغيه خالص وده بيحصل بأسلوبين رئيسيين

الأسلوب الأول هو الـ Timeout المباشر وده بتتظبط على مستوى الـ Client نفسه زي ما بنحدد أقصى وقت للريكوست وبعدها ترمي Exception وتقفل الاتصال

```cs
builder.Services.AddHttpClient("dummy", client =>
{
    client.BaseAddress = new Uri("https://dummyjson.com/");
    client.Timeout = TimeSpan.FromSeconds(5);
});
```

الأسلوب الثاني وهو الأهم والأكتر احترافية استخدام `CancellationToken` وده المفهوم المقابل للـ `AbortController` والـ `AbortSignal` في JavaScript

الفكرة إن الـ Client اللي باعت ريكوست للـ API بتاعتك لو قفل الصفحة أو عمل Cancel للريكوست فالـ Framework بتاع ASP.NET Core بيبعتلك `CancellationToken` بيعبر عن إن الكلاينت مشي

بدل ما السيرفر بتاعك يفضل مكمل في بعت الـ Outbound HTTP Requests على الفاضي أنت بتاخد التوكين ده وتباصيه للـ `HttpClient` بالشكل ده

```cs
app.MapGet("/todos/{id}", async (IHttpClientFactory factory, int id, CancellationToken cancellationToken) =>
{
    var client = factory.CreateClient("dummy");

    // بنمرر الـ cancellationToken للـ GetAsync
    var response = await client.GetAsync($"todos/{id}", cancellationToken);

    var jsonString = await response.Content.ReadAsStringAsync(cancellationToken);

    return Results.Content(jsonString, contentType: "application/json");
});
```

كده لو اليوزر كنسل الريكوست من ناحيته الـ .NET بتعمل Abort فوراً للـ Outbound Request وتقفل الـ Socket عشان متستهلكش موارد عالخاضي ولا تستنى سيرفر خارجي مش محتاجين الرد بتاعه خلاصCaching
