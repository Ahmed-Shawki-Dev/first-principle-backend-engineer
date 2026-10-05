## Latency
Time In Between Server Receives The Request and when it sends the response

بتتقاس بالـ ms والـ s يعني من الآخر بتقيس الزمن اللي خدته حاجة معينة عشان تتنفذ ليوزر واحد
****
## Throughput
rate at which server handles requests
it's measure of how much work it's able to get through at a unit time
يعني السيستم يقدر يشيل كام ريكوست في زمن معين
مثلا السيستم بتاعنا فرضا يقدر يشيل 100 ريكوست في الثانية وهكذا

بنستخدمها عشان نحلل قدرة السيستم
وبتخدمنا جدا لو هنعمل Batching Processing
### Requests Per Second (RPS)
هي وحدة قياس الـ **Throughput** الأشهر في الـ Web APIs بتعبر عن عدد الـ HTTP Requests اللي السيرفر يقدر يعالجها ويبعت الـ Response في ثانية واحدة
### Latency vs Throughput
ده Trade-off غالبا لو اخترت اعلى Throughput الـ Latency هتزيد وهكذا
****
## Bottleneck
هي تعريف لأبطأ وأضعف مورد في السيستم اللي بيحدد الحد الأقصى للأداء كله
زي مثلا لو معاك أقوى واحسن كارت شبكة و CPI بس الـ DB Connection Pool اخره 10 اتصالات بس السيستم مش هيقدر يعدي سرعة ال 10 دول والباقي هيفضل مستني
****
## p95 Latency
مينفعش تحسب الأداء والسرعة بتاعت سيرفرك بأنك تجيب الـ Average عشان تخيل عندك 100 يوزر 99 منهم خلصوا في ثانية واخر واحد خد 10 ثواني الـ Average بتاعهم هيطلع رقم منطقي

الـ **p95** بقى معناها أن 95% من المستخدمين الريكويست بتاعهم خلص في الوقت المطلوب إنما اـ 5% المتبقين واجهوا مشاكل وخدوا وقت أطول وده اللي بيعكس التجربة الحقيقية للي حصل 