لو عندنا E-Commerce وفي عميل بيعمل Transaction لفلوس
وفي نفس اللحظة دي السيرفر وقف لأي سبب سواء تحديث أو أي حاجة، فكدا الفلوس هتروح عالراجل ودي كارثة.

هي اه ليها حلول أننا لما نوقف السيرفر بنهيأ سيرفر جديد تاني كتعويض وبتفتحه بعدها تقفل القديم بس لسه المشكلة موجودة لو التحويل حصل في نفس المللي ثانية اللي السيرفر القديم فيها اتقفل

دي مشكلة موجودة من زمان جدا مع أول لحظة من ظهور الباك اند والسيرفرات

الحل ببساطة كان في
## Graceful Shutdown
ببساطة أحنا بنعلم السيرفر الـ Good Manners أو الأخلاقيات لما يجي يقف ميبقاش عشوائي ويقف وخلاص لا يعمل الحاجات اللي أحنا عايزنها بعدها يقفل
### Process Lifecycle
أي Application بيفتح في الـ OS بيشتغل كـ Process وزي أي حاجة حية فليها Lifecycle:
- How it Starts
- When It Ends
- How It Ends
لما الـ Process تيجي تقفل مش بتقفل مرة واحدة هي ليها **Established Protocol** خطوات بتحصل بعدها الـ Process تقفل
#### Signals
لما تيجي تقفل الـ Application بتاعك بيكلم الـ OS قبل ما يقفل وبيتكلموا مع بعض عن طريق Signals دي في أي بيئة Unix بتستعمل حاجة اسمها **IPC** أو **Inter Process Communication** دي الطريقة اللي ذكرناها قبل كدا في أزاي اتنين Processes يتواصلوا مع بعض
أنت لما بتفتح الـ App والـ OS يفتحله Process بيتحط معاه Handlers من الـ OS عشان يراقبوا الـ Process بشكل دائم
حاجات زي `SIGTERM` و `SIGKILL` و `SIGINT`
##### `SIGTERM`
دي طريقة قفل بس محترمة مش قاسية تخيل حد جه جنبك وقالك أقفل بطريقة لطيفة فالـ Backend بتاعك بيكون عنده فرصه أنه يخلص اللي بيعمله هو مش مطلوب منه يقفل في لاحظتها هو بيخلص اللي بيعمله عشان الدنيا متخربش بعدها يقفل

**Q) What Teminate Do?**
1. Finish Existing Requests
2. Clean Up Resources
3. Exit
##### `SIGINT`
دي معنى لـ Signal Intruption يعني بنعمل عملية مقاطعة بنقفل الأبلكيشن بس أحنا اللي بنقفله بإيدينا كمبرمجين مثلا ندوس `Ctrl+C` وبتتم في الغالب في مرحلة الـ Development

الفرق بينها وبين الـ `SIGTERM` أن اليوزر هو اللي بينفذ `SIGINT` إنما اللي بينفذ الـ `SIGTERM` الـ OS نفسه أو الـ Process Manager

وخد بالك هي تعتبر Graceful Shutdown لأنها مش بتقفل بأمر حاسم هي بتعمل نفس الـ Terminate بتخلص الحاجة اللي في إيدها وتخرج
##### `SIGKILL`
لما تتعمل الأبلكيشن بيقفل فورا بيموت حرفيا في لحظتها وبيسيب كل اللي في ايده بشكل لحظي
****
### Connection Draining
أول ما بيحصل `SIGTERM` بتحصل حاجة اسمها تفريغ للكونيكشنز
لما تقرر تقفل الريكوستات اللي جايالك غالبا أصلا بيكون عندك Requests جوا عمال تهندلها

تخيل أنت رايح مطعم والمطعم خلاص هيقفل أكيد صحاب المطعم مش عشان المطعم هيقفل هيقفلوا الأنوار ويطردوا الزباين اللي جوا هما بيعاملوهم أحسن معاملة لحد ما يخلصوا كلهم بس بيقولولهم مثلا في 20 دقيقة وهنقفل
وفي نفس الوقت مش بيقبلوا أي زباين جديدة من برا عشان خلاص هيقفلوا

فلما يحصل `SIGTERM` بيحصل Stop Accepting New Connections
وبيشتغل في الـ Connections اللي معاه

طب تخيل الحاجات اللي جوا تقيلة ومعقدة جدا وهتاخد وقت كبير جدا جدا مش منطقي تفضل شغالة ومعطلة الأبلكيشن أنه يقفل
#### Timeout
هنا بنـ Setting Timeout بنبدأ نحط وقت للريكوستات اللي جوا مثلا 30 ثانية لو خلصت ياعم في أول ثانية الأبلكيشن هيقفل
وهكذا ولو عدت الـ 30 ثانية هيقفل بردو حتى لو هما لسه شغالين عشان الـ Server ميفضلش مفتوح ومش عارفين نقفله عشان الريكوستات معلقة

وأختيار الـ Timeout في أحتمالات كتير وهو على حسب الـ Application بتاعك تحدد يخلص بسرعة ولا لا
#### Resource Cleanup
قبل ما تقفل لازم تعمل عملية **Cleanup** حاجة زي الـ Network Connections أنت أي ريكوست بيجي للسيرفر بتاعك اللي بيستلمه هو الـ OS وبيفتح Network Connection فلو جيت تقفل البرنامج ونسيت ممكن تحصل مشاكل Out Of Memory أو حاجة

نفس الكلام DB Connections لو كنت شغال على حاجة وقفلت من غير ما تعمل Cleanup ممكن يحصل **Deadlocks** أو مشاكل أو الداتا انتجرتي تضيع

لما بنعمل Graceful Shutdown بنعمل Cleanup بعكس ترتيبهم اللي اتفتحوا بيهم عشان منمسحش اوبريشن معتمدة على اوبريشن تانية وتسبب مشاكل سواء في الـ Mapping او Mismatch أو اي حاجة زي كدا
****
## `IHostApplicationLifetime`
الأنترفيس دي هي اللي بتعرف الأبلكيشن الـ Signals اللي جاية من الـ OS وبتوفر 3 تريجرز رئيسين:
1. **`ApplicationStarted`**
دي هي اللي بتشتغل أول ما الـ Server يفتح ويدخل في حالة الـ Running
2. **`ApplicationStopping`**
دي بتشتغل لما يحصل `SIGTERM` أو `SIGINT` وبيحصل **Connection Draining** ويعمل **Timeout** زي ما شرحنا فوق
3. **`ApplicationStopped`**
بتشتغل بعد ما كل حاجة تتنضف والـ Active Requests تخلص والسيرفر هيموت خلاص

وده مثال في الـ WebApi يوضح الفكرة أكتر:
```cs
var app = builder.Build();

var lifetime = app.Services.GetRequiredService<IHostApplicationLifetime>();

// 1. When App Start
lifetime.ApplicationStarted.Register(() =>
Console.WriteLine("🟢 App Started: Kestrel is listening for requests.")
);

// 2. When Shutting Down
lifetime.ApplicationStopping.Register(() =>
Console.WriteLine(
"🟡 SIGTERM/SIGINT Received: Connection Draining Started. Stop accepting new requests!"
)
);

// 3. When App Finally Die
lifetime.ApplicationStopped.Register(() =>
Console.WriteLine("🔴 App Stopped: All resources cleaned up. Exit.")
);
```
