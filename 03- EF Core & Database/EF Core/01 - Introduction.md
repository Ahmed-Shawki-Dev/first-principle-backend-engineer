## Introduction And Setup
من الأول خالص EF Core معمول عشان يكون وسيط بين الـ DB والـ Code نفسه بتاع .Net وده هو تعريف أي **ORM** بالمناسبة

![](https://res.cloudinary.com/du7j80lol/image/upload/v1792833838/obsidian/upsbphfjylyz4njmkion.png)

عشان تبدأ تشتغل محتاج تنزل مكاتب الـ EF Core والـ DB اللي أنت هتشتغل بيها أحنا هنشتغل بـ **Postgres**:
```bash
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
dotnet add package Microsoft.EntityFrameworkCore.Design
```

هنعمل بعد كدا Class واحد في البروجيكت هيكون هو حلقة الوصل بين الأبلكيشن والـ DB هنسميه `ApplicationDbContext` ده الأسم الرسمي وهيورث من كلاس `DbContext` وأحنا حاليا شغالين Console فهنحط الـ Config جوا الكلاس بس ده خطر ومش صح

```cs
public class ApplicationDbContext:DbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseNpgsql(
            "Host=localhost;Port=5432;Database=EFCore_Learn;Username=postgres;Password=postgres;");
    }
}
```

حطينا الـ **Connection String** بتاع **Postgres** بالمناسبة أنت بتكتب بنظام Key Value Pairs وبين كل واحد والتاني **;** والفريمورك بيحولها لأوبجيكت ويتعامل هو دي طريقة الكتابة

بالمناسبة `OnConfiguring` دي فانكشن متعرفة في الـ `DbContext` وبتتنفذ في الكونستراكتور بتاعها فلما أنت عملت `override` انت كدا دخلت البوليمرفزم وخليته ينفذ الفانكشن بتاعتك اللي فيها الكونيكشن استرنج بتاعك
****
### Create Model And Push To DB
الـ Model هو Class بيمثل Entity في الـ DB بتكتبه مفرد وبيتحط في الداتا بيز بشكل تلقائي كجمع

```cs
public class Student
{
    public Guid Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public int Age { get; set; }
}
```

كدا أنت عملت **Model** بس ملوش أي لازمة حرفيا لازم تعرف الـ Ef Core أنه موجود فبتروح للـ `ApplicationDbContext` تحط جواه كدا:

```cs
public DbSet<Student> Students { get; set; }
```

هنا أنت بتقول أن كلاس الـ Students ده بيمثل جدول الداتا بيز عشان تقدر تتفاعل معاه في الكود

خلاص بعد ما ضبطنا الدنيا نعمل **Migration** وده هنشرحه بالكامل في الملف الجاي

وفي الآخر بتعمل Update Database عشان التغييرات تروح للداتا بيز بجد