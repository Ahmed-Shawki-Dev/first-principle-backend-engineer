أول ما الـ **Application** يفتح بيحصل كام حاجة كدا ليها علاقة بالـ **Reflection**
أول حاجة بتحصل أن الفريمورك بيمسك الكود ويشوف الـ Classes والـ Methods واللي موجودة ويشوف مين الـ Classes اللي فيها `ControllerBase` بعدها يدخل يشوف الـ **Attributes** اللي موجودة فوقيها زي مثلا `[ApiController]`, `[HttpGet]`, `[Authorize]` وهكذا

المهم طب أزاي الحاجات دي بتتحول لـ **Runtime Metadata**؟
الـ **Attributes** بتكون مجرد **Tag** أو علامة مميزة بس أول لما الـ **Reflection** يشوفها بيعمل منها **Object** في الـ **Memory**

المهم الـ Endpoint بالـ Attributes اللي معاها دي بتتجمع في اوبجيكت واحد اسمه `Endpoint` جواه الـ **`RequestDelegate`** والـ **`Metadata`**

فمن الآخر كدا أنت بيتجمعلك كل الـ Endpoints في حاجة زي جدول أول ما الأبلكيشن يشتغل عشان في السكشن الجاي لما يحصل راوتنج كل حاجة تخلص بسرعة ومش مع كل ريكوست يقعد يعمل الحوار ده من الصفر
## Modern Endpoint Model
معمارية الـ **Routing** في .net الموضوع ببساطة ماشي أزاي؟
1. أول مرحلة هي الـ `UseRouting` بنعمل Matching للـ **URL** ولما يلاقيه ياخد الـ **Endpoint** يحطها في الـ `HttpContext` اللي معاه وفي ملحوظة مهمة جدا أن الـ **Route** بيتحدد من حاجتين هما الـ **URL** والـ **Method** عشان ممكن 4 Endpoints يكون ليهم نفس الـ URL عادي مثلا فاللي بيفرق بينهم الـ **Method**
2. بعد ما بيمسك الـ Endpoint مش بينفذه بشكل مباشر أكيد، بيبدأ ينفذ الـ **Middlewares** فهي بتفتح الـ `HttpContext` اللي اتخزن وتشوف ايه الميدلويرز اللي تنفذها وهكذا
3. آخر مرحلة يبدأ ينفذ الـ Endpoint واللوجيك اللي جواها ويرجع الـ Response
## Endpoints
> a specific **URL** combined with an **HTTP method** where a client application (like a mobile app or a browser) can access a particular resource or functionality provided by the server

فهو حاجة ممكن تكون:
- Selected, by matching the URL and HTTP method.
- Executable: Has a RequestDelegate.
- Extensible: Has a Metadata collection.
- Selectable: Optionally, has routing information.
- Enumerable: The collection of endpoints can be listed by retrieving the EndpointDataSource from DI.
### Endpoint metadata
أنت عندك Endpoints كتير، طب لو واحد منهم محتاج شرط عشان يشتغل أو حاجة، مثلا **Authentication** أو أيا يكن فدي أسمها **Metadata** يعني بمجرد ما يكون عندك **routing-aware** تقدر تضيف أي Metadata لأي Endpoint أو Controller أنت عايزه
****
