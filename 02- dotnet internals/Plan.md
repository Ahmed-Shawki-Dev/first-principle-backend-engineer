## Phase 02 — ASP.NET Core Internals & Deep API Architecture
**Duration:** 3 Weeks
**Dates:** October 5 – October 25, 2026
****
## Week 1 — Hosting, DI & Application Composition
### Saturday — Hosting & Configuration 1
* [x] Study `WebApplication.CreateBuilder(args)`.
* [x] Understand `WebApplicationBuilder` and the Generic Host.
* [x] Understand Host configuration vs Application configuration.
* [x] Study configuration providers and configuration precedence.
* [x] Understand environment-specific configuration.
* [x] Study `IConfiguration`.
**Practical:** جرّب تشوف الـ configuration فعليًا في مشروع ASP.NET Core صغير.
---
### Sunday — Hosting & Configuration 2
* [x] Study `IOptions<T>`.
* [x] Study `IOptionsSnapshot<T>`.
* [x] Study `IOptionsMonitor<T>`.
* [x] Study Options validation.
* [x] Understand service registration during application startup.
* [x] Understand what `Build()` creates.
**Practical:** اعمل Options class واربطها بالـ configuration وجرب الـ 3 options patterns.
---
### Monday — Application Lifetime
* [x] Understand application lifetime and graceful shutdown.
* [x] Study `IHostApplicationLifetime`.
* [ ] Understand graceful shutdown and cancellation on Linux.
**Practical:** شغّل التطبيق، اقفله، وراقب lifecycle/cancellation behavior.
---
### Tuesday — DI Foundations
* [x] Study `IServiceCollection`.
* [x] Study `ServiceDescriptor`.
* [x] Study `IServiceProvider`.
* [x] Understand service registration vs service resolution.
* [x] Understand dependency graphs and object construction.
* [x] Study Singleton / Scoped / Transient lifetimes.
* [x] Understand Root Service Provider vs Service Scopes.
* [ ] Understand scope creation and disposal.
**Practical:**
* [x] Inspect registered `ServiceDescriptor`s.
* [x] Reproduce Singleton / Scoped / Transient behavior.
---
### Wednesday — DI Internals
* [ ] Study `IDisposable` / `IAsyncDisposable` disposal through DI.
* [ ] Understand captive dependencies.
* [ ] Study `ValidateScopes` and `ValidateOnBuild`.
* [ ] Study factory registrations.
* [ ] Study open generic registrations.
* [ ] Study keyed services in modern .NET.
**Practical:**
* [ ] Reproduce a captive dependency.
* [ ] Observe scope disposal with `IDisposable`.
* [ ] Experiment with open generics.
* [ ] Experiment with keyed services.
---
### Thursday — DI Construction & Final Experiment
* [ ] Study `ActivatorUtilities`.
* [ ] Understand Dependency Inversion vs Service Locator.
* [ ] Understand where factories and other design patterns appear inside the DI system.
**Practical:**
* [ ] Use `ActivatorUtilities` to construct objects.
* [ ] Trace a dependency from registration → resolution → construction → disposal.
---
## Week 2 — ASP.NET Core API Execution & Extensibility

### Day 1 — Endpoint & Metadata Model
- [x] Understand the modern Endpoint model.
- [x] Study `Endpoint`.
- [x] Study `EndpointMetadata`.
- [x] Understand how attributes become runtime metadata.
- [x] Study `[ApiController]`, `[Route]`, `[HttpGet]`, `[HttpPost]`.
- [x] Study `ControllerActionDescriptor`.
- [x] Understand how controllers and actions are represented by the framework.
- [x] Understand how framework components consume endpoint metadata.
- [x] Study Endpoint Filters.
- [x] Understand when Endpoint Filters are useful.
#### Practical
- [ ] Inspect endpoint metadata at runtime.
- [ ] Create custom attributes and inspect them through reflection.

### Day 2 — Model Binding & Validation
- [x] Study Model Binding internals.
- [x] Understand Route, Query, Headers, Form, and Body binding sources.
- [x] Understand how request data becomes .NET objects.
- [x] Study `ModelState`.
- [x] Study validation metadata.
- [x] Understand automatic API validation.

#### Practical
- [x] Build an endpoint with custom model binding or validation behavior.
- [x] Test valid requests, invalid values, missing data, and binding failures.

### Day 3 — Serialization & JSON
- [x] Study `System.Text.Json` in the API pipeline.
- [x] Understand serialization vs deserialization.
- [x] Understand source-generated JSON serialization.
- [ ] Compare reflection-based serialization with source generation at a practical level.
- [x] Understand how serialization fits into request and response execution.

#### Practical
- [x] Test JSON request deserialization and response serialization.
- [x] Compare reflection-based and source-generated serialization behavior.

### Day 4 — MVC Filter Pipeline
- [ ] Study Authorization Filters.
- [ ] Study Resource Filters.
- [x] Study Action Filters.
- [ ] Study Exception Filters.
- [ ] Study Result Filters.
- [x] Understand filter execution order.
- [ ] Understand filter scopes.
- [ ] Understand dependency injection inside filters.
- [ ] Compare Middleware, Endpoint Filters, and MVC Filters.
- [ ] Identify the correct extension point for common API concerns.

#### Practical
- [x] Build a custom Action Filter.
- [ ] Build a custom Exception Filter.
- [ ] Observe filter execution order with logging or breakpoints.

### Day 5 — Controller, Action & Result Execution
- [x] Understand controller activation.
- [x] Understand constructor dependency injection into controllers.
- [x] Study action invocation.
- [x] Understand action parameters.
- [x] Study Action Results.
- [x] Understand result execution.
- [x] Study formatters and content negotiation at a practical level.
- [x] Understand object → JSON → HTTP response.

#### Practical
- [x] Trace controller activation and action execution with breakpoints.
- [x] Observe how an action result becomes an HTTP response.

### Day 6 — Integration & Framework Tracing
- [ ] Build a custom Endpoint Filter.
- [ ] Trace a real endpoint with breakpoints.
- [ ] Inspect relevant ASP.NET Core framework source when behavior is unclear.
- [ ] Trace endpoint selection, metadata consumption, binding, validation, filter execution, action invocation, and response generation.
- [ ] Revisit unclear concepts from Days 1–5.

#### Practical completion checklist
- [ ] Inspect endpoint metadata at runtime.
- [ ] Create custom attributes and inspect them through reflection.
- [ ] Build a custom Endpoint Filter.
- [ ] Build a custom Action Filter.
- [ ] Build an endpoint with custom model binding/validation behavior.
- [ ] Trace a real endpoint with breakpoints and framework source.

---
## Week 3 — Error Architecture, Outbound HTTP & Defensive Backend Design
### Goal
تفهم إزاي الـ Backend بيتعامل مع الأخطاء في الأنظمة الحقيقية، وإزاي يحمي العمليات من التكرار، ويتواصل مع الخدمات الخارجية، ويدير الاتصالات والإلغاء بشكل صحيح.

الهدف مش إنك تحفظ أسماء الـ APIs، لكن إنك تفهم سلوكها، وليه بنستخدمها، وإزاي بتتفاعل مع بعضها، وتقدر تطبّق المفاهيم دي على API موجودة بالفعل بدل ما تبني مشاريع تجريبية ملهاش علاقة بشغلك.
****
### 1. Error Handling & Result Pattern
الموضوع ده عن إزاي الـ API تتعامل مع الفشل بشكل موحّد، وإزاي تفرّق بين خطأ متوقع من الـ Business Logic ومشكلة غير متوقعة في التطبيق.

هتبدأ بفهم Centralized Exception Handling في ASP.NET Core، ودور `IExceptionHandler` في التعامل مع الـ Exceptions على مستوى التطبيق. هتفهم كمان دور `ProblemDetails` و`IProblemDetailsService` في تكوين Error Response موحّد، وعلاقتهم بمعيار RFC 7807، وإزاي تصمم API Error Contract واضح للـ Frontend.

هتتعرف على الحقول الأساسية في `ProblemDetails` زي `type` و`title` و`status` و`detail`، وإمكانية إضافة معلومات خاصة بالتطبيق من خلال `extensions`. وهتفهم إزاي تختار HTTP Status Code مناسب، وإزاي تتعامل مع الأخطاء في Production من غير ما تسرّب Stack Traces أو تفاصيل داخلية، مع تسجيل المعلومات اللازمة للـ Debugging من خلال Logging.

بعدها هتنتقل للـ Result Pattern. الفكرة هنا إن مش كل عملية فشلت لازم ترمي Exception. مثلًا، لو الطالب مش موجود أو المجموعة مكتملة، ده ممكن يكون فشل متوقع في الـ Business Logic، مش بالضرورة مشكلة استثنائية.

هتفهم شكل Result Type بسيط، وإزاي يعبّر عن Success أو Failure، وإزاي يحمل البيانات الناتجة أو معلومات الخطأ. كمان هتفهم إزاي الـ Controller يحوّل النتيجة إلى HTTP Response مناسب، وإمتى يكون إرجاع Result أفضل من رمي Exception، وليه الـ Result Pattern مش بديل عن Centralized Exception Handling.

في التطبيق، هتستخدم `IExceptionHandler` مع `ProblemDetails` في API موجودة، وبعدها تطبّق Result Pattern على عملية حقيقية، وتشوف الفرق بين التعامل مع Business Failure والتعامل مع Unexpected Exception.

**نطاق الموضوع:** Centralized Exception Handling، `IExceptionHandler`، `ProblemDetails`، `IProblemDetailsService`، RFC 7807، API Error Contract، Expected vs. Exceptional Failures، Result Pattern، Result Type، HTTP Status Codes، Logging، وProduction Error Responses.
****
### 2. Idempotency & Defensive API Design
الموضوع ده عن حماية العمليات من التكرار، خصوصًا لما يحصل Network Failure أو العميل يضطر يعمل Retry لنفس الـ Request.

تخيّل إن العميل بعت Request لإنشاء عملية دفع، والسيرفر نفّذ العملية فعلًا، لكن الـ Response ضاع قبل ما يوصل للعميل. العميل مش عارف العملية نجحت ولا لأ، فبعت نفس الطلب تاني. هنا لازم تفهم إزاي تمنع إنشاء عمليتين بدل واحدة.

هتدرس معنى Idempotency في HTTP، والفرق بين إن تكرار الطلب ينتج نفس الحالة النهائية وبين إن التنفيذ نفسه يتكرر. هتفهم كمان ليه `PUT` و`DELETE` ليهم Idempotent Semantics، لكن ده مش معناه إن أي Implementation ليهم مضمون إنه يمنع كل أنواع التكرار.

بعدها هتدرس Idempotency Keys، وإزاي العميل يبعت مفتاح يميّز العملية، وإزاي السيرفر يحتفظ بالمفتاح والنتيجة المرتبطة بيه عشان يتعامل مع الطلبات المتكررة. هتفهم كمان إزاي تتعامل مع نفس المفتاح لو اتبعت مع بيانات مختلفة، وإيه اللي يحصل لو طلبين بنفس المفتاح وصلوا في نفس الوقت.

هتربط ده بمفاهيم Database Constraints وTransactions، لأن مجرد إنك تعمل Check وتشوف المفتاح موجود ولا لأ قبل الـ Insert مش كفاية لو فيه Requests متزامنة. وهتفهم كمان أهمية Key Expiration، وإزاي تختار مدة الاحتفاظ بالمفاتيح حسب طبيعة العملية.

بعدها هتدرس Fail-Fast Validation عند حدود الـ API. هتفرّق بين التحقق من صحة المدخلات وبين Business Validation اللي بيتأكد إن العملية مسموح بيها وفقًا لحالة النظام.

وهتدرس Cancellation Propagation من خلال `CancellationToken`، وإزاي الإلغاء ممكن ينتقل من الـ HTTP Request للـ Services وعمليات EF Core والـ Outbound HTTP Requests. هتفهم إن طلب الإلغاء مش معناه إن كل عملية اتلغت فعلًا، خصوصًا لو بدأت تنفّذ Side Effects أو تغيّر بيانات.

في التطبيق، هتصمم عملية حقيقية في API موجودة، زي إنشاء Order أو تسجيل Payment Record، بحيث تتعامل مع تكرار الطلبات والطلبات المتزامنة بطريقة صحيحة. وهتراجع كمان أماكن الـ Validation وإزاي الـ Cancellation بيتنقل بين الطبقات.

**نطاق الموضوع:** HTTP Idempotency، Idempotent Operations، Duplicate Requests، Retry Scenarios، Idempotency Keys، Key Persistence، Request Fingerprints، Concurrent Requests، Database Constraints، Transactions، Key Expiration، Fail-Fast Validation، API Boundary Validation، Business Validation، وCancellation Propagation.

---

### 3. HttpClient & Outbound HTTP Fundamentals

لحد هنا أغلب تركيزك كان على الـ Requests اللي بتدخل الـ API. الموضوع ده بينقلك للاتجاه العكسي: إزاي الـ Backend نفسه بيبعت HTTP Requests لخدمات خارجية.

هتبدأ بفهم `HttpClient`، وإزاي بيبعت Requests ويستقبل Responses، وإيه العلاقة بينه وبين `HttpMessageHandler` و`SocketsHttpHandler`. هتفهم مسؤولية كل جزء، وإزاي الـ Client والـ Handler بيشاركوا في عملية الاتصال.

بعدها هتفهم ليه إنشاء `HttpClient` والتخلص منه مع كل Request ممكن يسبب مشاكل في إدارة الاتصالات. هتدرس Connection Pooling، وإزاي إعادة استخدام الاتصالات بتقلل الحاجة لإنشاء TCP Connections جديدة باستمرار، وعلاقة ده بمشكلة Socket Exhaustion.

هتدرس كمان DNS Resolution وConnection Lifetime. مثلًا، لو عنوان خدمة خارجية اتغيّر في DNS، ممكن اتصال موجود بالفعل يفضل شغال على العنوان القديم. هنا هتفهم العلاقة بين عمر الاتصال وإعدادات زي `PooledConnectionLifetime` داخل `SocketsHttpHandler`.

المهم إنك تفرّق بين المفاهيم دي: Connection Pooling معناها إعادة استخدام الاتصالات، وDNS Resolution معناها تحويل اسم الخدمة لعنوان IP، وConnection Lifetime بتحدد المدة اللي الاتصال يقدر يفضل فيها قابلًا لإعادة الاستخدام. أما Socket Exhaustion فهي مشكلة ممكن تحصل لما التطبيق يستهلك عددًا كبيرًا من الـ Sockets المتاحة.

في التطبيق، هتستخدم `HttpClient` للتواصل مع External Service من API موجودة، وتلاحظ سلوك الاتصالات وإعادة استخدامها. وعلى Linux، تقدر تستخدم أدوات زي `ss` لمراقبة الاتصالات والـ Sockets، وتربط اللي بتشوفه بطريقة إدارة الاتصالات. هتفهم كمان إن عدد الاتصالات وحده مش كفاية عشان تحكم إن عندك مشكلة.

**نطاق الموضوع:** Modern `HttpClient`، `HttpMessageHandler`، `SocketsHttpHandler`، Connection Pooling، TCP Connections، DNS Resolution، Connection Lifetime، `PooledConnectionLifetime`، Socket Exhaustion، Client Lifetime، Handler Lifetime، وLinux Socket Inspection.

---

### 4. IHttpClientFactory, Named Clients & Typed Clients

بعد ما تفهم مشاكل إدارة الاتصالات، هتدرس الطريقة اللي ASP.NET Core بتوفرها لإدارة الـ HTTP Clients من خلال Dependency Injection.

هتفهم دور `IHttpClientFactory` في إنشاء `HttpClient` Instances وإدارة الـ Handlers وإعادة استخدامها، وإزاي ده بيساعد في تجنب مشاكل الاتصال اللي ممكن تحصل مع إنشاء Clients والتخلص منها بطريقة غير مناسبة.

هتدرس العلاقة بين عمر الـ `HttpClient` وعمر الـ Handler والـ Connection Pool، وهتفهم إن الـ Factory مش مجرد طريقة مختصرة لإنشاء Client، لكنه كمان بيدير دورة حياة الـ Handlers بطريقة مناسبة.

بعدها هتدرس Named Clients، ودي بتسمحلك تجهّز إعدادات مختلفة لكل External Service. مثلًا، خدمة الدفع ممكن يكون ليها Base Address وTimeout معينين، وخدمة تانية ليها إعدادات مختلفة.

ثم هتدرس Typed Clients، وإزاي تعمل Class متخصصة للتعامل مع خدمة خارجية محددة بدل ما توزّع تفاصيل الـ HTTP Requests في أكتر من Application Service.

هتفهم إزاي الـ Typed Client بيتسجل في DI، وإزاي بياخد `HttpClient` من خلال الـ Factory، وإزاي الـ Application Service بيتعامل معاه من غير ما يحتاج يعرف تفاصيل تكوين الـ Request والاتصال بالخدمة الخارجية.

هتقارن بين استخدام `HttpClient` مباشرة، وNamed Clients، وTyped Clients، وتفهم إمتى كل اختيار يكون مناسب، وتأثيره على فصل المسؤوليات والاختبار وسهولة تغيير إعدادات الخدمات الخارجية.

في التطبيق، هتجهّز Typed Client لخدمة خارجية موجودة أو مناسبة لمشروعك، وتسجله باستخدام `IHttpClientFactory`، وتضبط إعداداته، وتستخدمه من خلال Application Service.

**نطاق الموضوع:** `IHttpClientFactory`، Client Lifetime، Handler Lifetime، Named Clients، Typed Clients، Base Address، Default Headers، Timeout Configuration، DI Registration، External Service Encapsulation، والفرق بين Direct وNamed وTyped Client Usage.

---

### 5. DelegatingHandler & Outbound HTTP Pipelines

الموضوع ده عن إزاي تضيف سلوك مشترك للـ HTTP Requests الخارجة من التطبيق، من غير ما تكرر نفس المنطق في كل Service بتتعامل مع External Service.

هتبدأ بفهم `HttpMessageHandler` ودوره في تنفيذ الـ Requests، وبعدها هتدرس `DelegatingHandler`، وهو Handler بيقدر ينفّذ منطق قبل ما يمرر الـ Request للـ Handler اللي بعده، وبعد رجوع الـ Response يقدر ينفّذ منطق إضافي.

هتربط الفكرة بالـ Decorator Pattern: بدل ما تعدّل كل Service بتبعت Requests، بتضيف طبقة بتغلّف عملية الاتصال وتضيف مسؤولية جديدة.

هتفهم إزاي أكتر من Handler ممكن يتجمعوا في Outbound HTTP Pipeline، وإزاي ترتيبهم بيأثر على التنفيذ. وهتدرس أمثلة زي Outbound Logging، وإضافة Authentication Headers، وقياس زمن الاستجابة، وإضافة Correlation Information.

هتفرّق بين مسؤولية إضافة Access Token موجود بالفعل وبين مسؤولية الحصول على Token وتجديده وإدارة انتهاء صلاحيته. كمان هتفهم أهمية حماية البيانات الحساسة، وإن الـ Logging مش المفروض يكشف Tokens أو Credentials أو بيانات خاصة موجودة في الـ Requests والـ Responses.

هتدرس إزاي تسجل `DelegatingHandler` من خلال `IHttpClientFactory` وتربطه بـ Named Client أو Typed Client. وهتفهم تأثير Handler Lifetime على الـ Dependencies الموجودة جواه، وليه مش صح إن Handler طويل العمر يحتفظ بحالة مرتبطة بـ Request معين أو يعتمد بشكل مباشر على بيانات Scoped خاصة بطلب واحد.

في التطبيق، هتضيف `DelegatingHandler` لعميل خارجي موجود، وتراقب سلوكه قبل إرسال الـ Request وبعد استقبال الـ Response، وتفهم تأثير ترتيب الـ Handlers وطريقة تسجيلها في DI.

**نطاق الموضوع:** `HttpMessageHandler`، `DelegatingHandler`، Decorator Pattern، Handler Chains، Outbound HTTP Pipelines، Handler Ordering، Logging، Authentication Headers، Correlation، Request Timing، DI Registration، Handler Lifetimes، وSensitive Data Protection.

---

### 6. Outbound Timeouts, Cancellation & External Service Failures

وجود `HttpClient` مضبوط مش معناه إن الخدمة الخارجية هترد دائمًا أو إن الاتصال هيكتمل بنجاح. الموضوع ده عن سلوك التطبيق لما الخدمة الخارجية تتأخر أو تفشل.

هتدرس Timeout Behavior، والفرق بين انتهاء المهلة المحددة للاتصال وبين Cancellation ناتج عن إلغاء الـ Request الأصلي. وهتفهم إن الإلغاء مش ضمان إن الخدمة الخارجية نفسها وقفت تنفيذ العملية، خصوصًا لو كانت بدأت تنفّذ عملية لها Side Effects.

هتفرّق بين HTTP Error Response وبين Transport Failure. مثلًا، لو الخدمة رجعت `500 Internal Server Error`، ده معناه إنك استلمت HTTP Response بكود يدل على الفشل. لكن لو الاتصال نفسه فشل قبل ما تستقبل Response، فدي حالة مختلفة.

هتفهم إن `HttpClient` مش بيحوّل كل HTTP Status Code من نوع `4xx` أو `5xx` تلقائيًا إلى Exception في كل استخدام. وهتدرس دور `EnsureSuccessStatusCode` في التعامل مع الـ Responses غير الناجحة، وإزاي الكود اللي بيستخدم الـ Client بيحدد طريقة معالجة النتيجة.

بعدها هتربط الموضوع بالـ Error Handling والـ Result Pattern اللي درستهم في بداية الأسبوع. هتفهم إزاي الـ Application Service ممكن يحوّل فشل External Dependency إلى نتيجة مفهومة أو Exception مناسب، وإزاي الـ API ترجع Response واضح للعميل من غير ما تكشف تفاصيل الاتصال الداخلية.

هتدرس كمان Retry Safety، وإمتى إعادة المحاولة تكون مناسبة وإمتى ممكن تسبب تكرار عملية لها Side Effects. هتتعرف على Resilience Strategies زي Retries وCircuit Breakers باعتبارها مفاهيم مرتبطة بالموضوع، من غير ما تتوسع في تفاصيل تنفيذ كل Policy في الأسبوع ده.

في التطبيق، هتحاكي External Service بطيئة أو غير متاحة، وتراقب الـ Response أو الـ Exception والـ Timeout والـ Cancellation والـ Logs. وبعدها تفهم إزاي التطبيق بيتصرف عند فشل الخدمة، وهل العملية ينفع تتعاد بأمان، وإيه السلوك المناسب للـ API.

**نطاق الموضوع:** Outbound Timeouts، `CancellationToken`، Timeout vs. Cancellation، HTTP Error Responses، Transport Failures، `EnsureSuccessStatusCode`، External Dependency Failures، Error Translation، Logging، Retry Safety، Resilience Concepts، وFailure Simulation.

---

### Practical Integration

التطبيق العملي للأسبوع كله يكون على API موجودة بالفعل، مش مشروع تجريبي منفصل.

هتربط Centralized Exception Handling و`ProblemDetails` بتصميم Error Contract موحّد. وتستخدم Result Pattern في عملية Business حقيقية. وبعدها تصمم Idempotent Operation تمنع التكرار وتتعامل مع الطلبات المتزامنة.

في الجزء الخاص بالـ Outbound HTTP، هتجهّز Typed `HttpClient` لخدمة خارجية، وتضيف `DelegatingHandler` للـ Logging، وتلاحظ Connection Reuse والـ Open Sockets باستخدام أدوات Linux. وفي النهاية، هتحاكي فشل الخدمة الخارجية وتراقب تأثيره على التطبيق.

التجارب دي مترابطة: هتشوف إزاي الأخطاء بتتحرك خلال التطبيق، وإزاي العمليات بتحافظ على سلوكها الصحيح، وإزاي الخدمات الخارجية بتأثر على استقرار الـ API.

مش مطلوب تعمل نظام دفع كامل أو تبني Infrastructure ضخمة. المطلوب إن كل تجربة تكشفلك سلوكًا محددًا وتساعدك تفهم السبب وراء التصميم المستخدم.

---

### Phase Completion — ASP.NET Core Internals

القسم ده بيوضح الصورة العامة للمرحلة كلها، بما فيها المفاهيم اللي درستها قبل Week 3. مش مطلوب تعيد دراسة كل المواضيع دي خلال الأسبوع الحالي.

لازم تكون فاهم إزاي Dependency Injection والـ Service Lifetimes بيشتغلوا في ASP.NET Core، وإزاي Configuration وOptions بيتجمعوا داخل التطبيق.

كمان لازم تفهم إزاي Attributes بتتحول إلى Runtime Metadata، وإيه دور Endpoint Metadata وFilters، وإزاي Model Binding وValidation وSerialization بيشاركوا في معالجة الـ Request.

لازم تقدر تشرح الفرق بين Middleware وEndpoint Filters وMVC Filters، وتطبّق Centralized `ProblemDetails` Error Handling، وتصمم Idempotent API Operations، وتشرح `HttpClient` والـ Handlers والـ Connection Pooling ودور `IHttpClientFactory`.

ولما تلاقي سلوك في الـ Framework مش واضح، تكون قادر ترجع للـ Documentation أو الـ ASP.NET Core Source Code المرتبط بالمشكلة بدل ما تعتمد على التخمين فقط.

في النهاية، الهدف إنك تطبّق المفاهيم على API حقيقية وتقدر تشرح أسباب اختياراتك والـ Trade-offs المرتبطة بيها، مش إنك تحفظ مصطلحات أو تنجز عددًا معينًا من التجارب.