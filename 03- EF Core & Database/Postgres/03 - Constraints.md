## Constraints
الـ **Constraints** من أهم الحاجات اللي بتساعدنا نحافظ على جودة وصحة الداتا اللي بتتخزن في الـ Database

احنا ممكن نحدد الـ Data Type بتاع كل Column بس ساعات ده مش بيكون كفاية

يعني مثلا لو عندنا Product وعايزين نحطله Price مينفعش يكون بالسالب أو بصفر عايزين نضمن أن كل Product ليه رقم مميز ميتكررش

هنا بييجي دور الـ **Constraints**

ببساطة دي Rules بنحطها على الـ Columns أو الـ Tables عشان نتحكم في الداتا اللي بتدخل الـ Database ولو حد حاول يدخل Data بتخالف القواعد دي بيحصل Error والـ Database بترفض العملية حتى لو القيمة جاية من Default Value

---
## Check Constraint

ده من أكتر أنواع الـ Constraints مرونة لأنه بيسمحلك تحط شرط معين لازم القيمة تحققه

يعني مثلا لو عندك Product وعايز تضمن أن الـ Price دايما أكبر من صفر

```sql
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric CHECK (price > 0)
);
```

هنا أي Price بالسالب أو بصفر مش هيتقبل

وممكن كمان تسمي الـ Constraint باسم معين عشان يبقى واضح في الـ **Error Messages** وتقدر تتعامل معاه بعد كده

```sql
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric CONSTRAINT positive_price CHECK (price > 0)
);
```

ومش شرط الـ Check يكون متعلق بـ Column واحدة بس ممكن تقارن بين أكتر من Column

يعني مثلا لو عندك `Price` و `DiscountedPrice` وعايز تضمن أن السعر بعد الخصم أقل من السعر الأصلي

```sql
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric CHECK (price > 0),
    discounted_price numeric CHECK (discounted_price > 0),
    CHECK (price > discounted_price)
);
```

هنا آخر Check مش متعلق بـ Column معينة لكنه بيقارن بين الاتنين وده اسمه **Table Constraint** بعكس الـ Column Constraint اللي بيكون مكتوب جنب تعريف الـ Column نفسها

وممكن تكتب الـ Check كـ Table Constraint حتى لو بتتحقق من Column واحدة عادي

بس خد بالك من حاجة مهمة جدا

الـ CHECK لو الـ Expression بتاعه رجع TRUE أو NULL الاتنين بيتعتبروا Valid

يعني لو الـ Price بـ NULL فالـ Check بتاع Price > 0 مش هيمنع العملية لأن النتيجة هتكون NULL

عشان كده لو عايز تمنع الـ NULL لازم تستخدم NOT NULL
****
## Not Null Constraint

ببساطة دي بتضمن أن الـ Column لازم يكون ليها Value ومينفعش تكون NULL

```sql
CREATE TABLE products (
    product_no integer NOT NULL,
    name text NOT NULL,
    price numeric
);
```

هنا مينفعش تعمل Insert من غير ما تبعت Value للـ `ProductNo` والـ `Name`

وممكن بردو تسمي الـ Constraint باسم معين

```
CREATE TABLE products (
    product_no integer,
    name text CONSTRAINT products_name_not_null NOT NULL,
    price numeric
);
```

في نقطة مهمة هنا أن الـ NOT NULL تعتبر أكثر كفاءة من أنك تعمل نفس الفكرة باستخدام CHECK

يعني الاتنين ممكن يحققوا نفس النتيجة وظيفيا

```sql
CHECK (name IS NOT NULL)
```

بس PostgreSQL بتتعامل مع الـ NOT NULL بشكل أكثر كفاءة

وممكن تحط أكتر من Constraint على نفس الـ Column عادي جدا

```sql
CREATE TABLE products (
    product_no integer NOT NULL,
    name text NOT NULL,
    price numeric NOT NULL CHECK (price > 0)
);
```

يعني هنا الـ Price لازم يكون موجود وفي نفس الوقت أكبر من صفر
****
## Unique Constraint

الـ **UNIQUE** بتضمن أن القيمة الموجودة في Column أو مجموعة Columns متتكررش في أي Row تاني

يعني مثلا لو عندك `ProductNo` وعايز كل Product يكون ليه رقم مميز

```sql
CREATE TABLE products (
    product_no integer UNIQUE,
    name text,
    price numeric
);
```

وممكن تكتبها كـ Table Constraint بردو

```sql
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric,
    UNIQUE (product_no)
);
```

طب لو عايز تعمل الـ UNIQUE على أكتر من Column؟

هنا بنستخدم **Composite Unique Constraint**

```sql
CREATE TABLE example (
    a integer,
    b integer,
    c integer,
    UNIQUE (a, c)
);
```

هنا الـ Combination بتاع A و C هو اللي مينفعش يتكرر

يعني ممكن A تتكرر عادي وممكن C تتكرر عادي لكن الاتنين مع بعض مينفعش نفس القيم يتكرروا في Row تاني

وفي حاجة مهمة جدا أن الـ UNIQUE في PostgreSQL بتعمل Automatically **Unique B-Tree Index** على الـ Column أو مجموعة الـ Columns اللي انت حددتها

يعني مش بس بتضمن عدم تكرار الداتا لكن كمان بيكون فيه Index بيساعد في عمليات البحث

طب والـ NULL؟

بشكل Default الـ PostgreSQL بتعتبر أن الـ NULL مش بيساوي NULL في الـ **UNIQUE Constraint**

يعني ممكن يكون عندك أكتر من Row بنفس الـ Column وقيمتها NULL ومش هيحصل Error

لكن تقدر تغير السلوك ده باستخدام

```sql
UNIQUE NULLS NOT DISTINCT (product_no)
```

هنا الـ NULL هتتعامل كأنها قيمة واحدة ومينفعش تتكرر
****
## Primary Key

الـ **Primary Key** من أهم الحاجات في الـ Relational Databases لأنه بيحدد الـ **Unique Identifier** لكل Row في الـ Table

يعني لو عندك Products كل Product لازم تقدر تميزه عن أي Product تاني

الـ Primary Key بيجمع بين حاجتين

- UNIQUE
- NOT NULL

يعني القيمة لازم تكون Unique ومينفعش تكون NULL

وعشان كده المثالين دول بيقبلوا نفس الداتا

```
CREATE TABLE products (
    product_no integer UNIQUE NOT NULL,
    name text,
    price numeric
);
```

```
CREATE TABLE products (
    product_no integer PRIMARY KEY,
    name text,
    price numeric
);
```

وممكن كمان تعمل **Composite Primary Key** يعني الـ Primary Key يكون مكون من أكتر من Column

```
CREATE TABLE example (
    a integer,
    b integer,
    c integer,
    PRIMARY KEY (a, c)
);
```

هنا الـ Combination بين A و C هو اللي بيميز كل Row

وفي شوية حاجات مهمة لازم تعرفها

الـ Primary Key بيعمل **Automatically Unique B-Tree Index** زي الـ UNIQUE

وبيجبر كل الـ Columns الموجودة فيه أنها تكون NOT NULL

والـ Table مينفعش يكون فيها أكتر من Primary Key واحد لكن ممكن يكون فيها أكتر من UNIQUE Constraint

كمان الـ Primary Key بيكون مهم جدا للـ Applications اللي بتحتاج تحدد Row معينة وتعملها Update أو Delete لأنه بيوفر طريقة مضمونة لتمييزها
****
## Foreign Key

الـ **Foreign Key** مسئول عن الحفاظ على الـ **Referential Integrity** بين الـ Tables المختلفة

يعني ببساطة لو عندك Table للـ Products وTable تانية للـ Orders وعايز كل Order يكون مرتبط بـ Product موجود فعلا

هنا بنستخدم الـ Foreign Key

```sql
CREATE TABLE products (
    product_no integer PRIMARY KEY,
    name text,
    price numeric
);

CREATE TABLE orders (
    order_id integer PRIMARY KEY,
    product_no integer REFERENCES products (product_no),
    quantity integer
);
```

هنا مينفعش تعمل Order بـ `ProductNo` مش موجود أصلا في الـ Products Table

الـ Orders هنا اسمها **Referencing Table** والـ Products اسمها **Referenced Table**

وممكن تختصر الـ REFERENCES وتكتب اسم الـ Table بس من غير ما تحدد الـ Column ووقتها PostgreSQL بتستخدم الـ Primary Key بتاع الـ Table دي بشكل تلقائي

```sql
product_no integer REFERENCES products
```

وممكن كمان تعمل Foreign Key على مجموعة Columns وده بيكون **Table Constraint**

```sql
FOREIGN KEY (b, c) REFERENCES other_table (c1, c2)
```

وطبعا عدد وأنواع الـ Columns لازم تكون متوافقة مع الـ Columns اللي بتعملها Reference

وفيه حاجة اسمها **Self-Referential Foreign Key** يعني الـ Table بتعمل Reference لنفسها

مثلا لو عندك Tree Structure وكل Node ممكن يكون ليها Parent Node

```sql
CREATE TABLE tree (
    node_id integer PRIMARY KEY,
    parent_id integer REFERENCES tree,
    name text
);
```

هنا الـ ParentId ممكن يشاور على Row تانية موجودة في نفس الـ Table
****
## ON DELETE & ON UPDATE

طب دلوقتي لو عندك Product موجود في الـ Products Table وعليه Orders بالفعل وقررت تمسحه

ايه اللي المفروض يحصل للـ Orders اللي مرتبطة بيه؟

الـ PostgreSQL بتوفرلك كذا **Behavior** تقدر تحدده باستخدام **ON DELETE**

**1. NO ACTION**
وده الـ Default Behavior

الـ Database هتسمح بعملية الحذف أنها تبدأ لكن في النهاية الـ Foreign **Key Constraint** لازم تفضل متحققة ولو لسه فيه Rows بتعمل Reference للـ Product المحذوف العملية هتعمل Error

وفي حالة الـ Transaction ممكن تأجل فحص الـ Constraint لوقت لاحق وتصلح الـ References قبل ما يتم الفحص

**2. RESTRICT**
هنا مينفعش تحذف Row أصلا طالما فيه Rows تانية بتعمل Reference ليها

والفرق بينها وبين NO ACTION أن RESTRICT مش بتسمح بتأجيل فحص الـ Constraint لوقت لاحق في الـ Transaction

**3. CASCADE**
هنا لو حذفت الـ Product كل الـ Rows المرتبطة بيه هتتمسح Automatically

**4. SET NULL**
لو حذفت الـ Row اللي معمولها Reference فالـ Foreign Key في الـ Rows التانية هيتحول لـ NULL

**5. SET DEFAULT**
نفس الفكرة لكن بدل NULL بيحط الـ Default Value بتاعة الـ Column

وطبعا لازم الـ Default Value نفسها تكون Valid وتحقق الـ Foreign Key Constraint وإلا العملية هتفشل

مثال عملي

```sql
CREATE TABLE order_items (
    product_no integer REFERENCES products ON DELETE RESTRICT,
    order_id integer REFERENCES orders ON DELETE CASCADE,
    quantity integer,
    PRIMARY KEY (product_no, order_id)
);
```

هنا لو حاولت تمسح Product لسه موجود في Order هتلاقي العملية اترفضت

لكن لو مسحت Order كل الـ OrderItems المرتبطة بيه هتتمسح تلقائيا

وده بيوضح أن اختيار الـ ON DELETE بيعتمد على طبيعة العلاقة بين الـ Tables

يعني لو الـ Table التانية بتمثل جزء من الـ Entity الأساسية ومينفعش تعيش لوحدها ممكن تستخدم CASCADE

لكن لو الاتنين Entities مستقلين عن بعض فغالبا RESTRICT أو NO ACTION هيكونوا أنسب

ونفس الكلام موجود مع **ON UPDATE** وبيشتغل لما القيمة اللي معمولها Reference تتغير بدل ما الـ Row تتمسح

والـ Options المتاحة هي نفسها
- NO ACTION
- RESTRICT
- CASCADE
- SET NULL
- SET DEFAULT

بس خلي بالك أن CASCADE هنا معناها أن القيمة الجديدة بتتنقل تلقائيا للـ Rows اللي بتعمل Reference للقيمة القديمة
****
## Important Notes

فيه شوية تفاصيل مهمة في الـ Foreign Keys

الـ Foreign Key لازم يعمل Reference على Columns تكون Primary Key أو عليها UNIQUE Constraint أو موجودة في Non-Partial Unique Index

كمان PostgreSQL مش بتعمل Automatically Index على الـ Foreign Key Columns نفسها

وده مهم لأن لما تعمل DELETE أو UPDATE للـ Row الموجودة في الـ Referenced Table الـ Database محتاجة تدور في الـ Referencing Table على الـ Rows المرتبطة بيها

فعشان كده ساعات بيكون من الأفضل تعمل Index على الـ Foreign Key Columns بنفسك حسب استخدامك والـ Queries بتاعتك

وفيه كمان حاجة اسمها **MATCH FULL**

بشكل Default لو أي Column من الـ Foreign Key Columns كانت NULL فالـ Row مش مطالبة تحقق الـ Foreign Key Constraint

لكن لو استخدمت MATCH FULL لازم كل الـ Columns تكون NULL مع بعض عشان الـ Row تتجاوز التحقق

يعني مينفعش يكون عندك Column بـ NULL والتانية فيها Value في الـ Composite Foreign Key

ولو مش عايز تسمح أصلا أن الـ Foreign Key يكون NULL استخدم NOT NULL