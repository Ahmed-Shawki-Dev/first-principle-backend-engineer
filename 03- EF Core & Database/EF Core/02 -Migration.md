![](https://res.cloudinary.com/du7j80lol/image/upload/v1793016938/obsidian/ozp7e9vfpqoennnwahqf.png)
****
## What Is Migration?
هي **Version Controll** للـ **Database Schema** زي الـ Git للأكواد بتاعتنا

بتاخد الكود بتاعك اللي كاتبه بـ C# وبتكتب جواها الـ SQL Code الخاص بي اللي بيتنفذ في الداتا بيز

الكود ده بيكون متقسم لحاجتين `Up` و `Down`
```cs
public partial class Initial : Migration
    {
        protected override void Up(MigrationBuilder migrationBuilder)
        {
     
        }

        protected override void Down(MigrationBuilder migrationBuilder)
        {
        
        }
    }
```

- الـ Up بيتحط فيها الأوامر اللي بتطبق في الداتا بيز اللي بتشتغل فعليا وتطبق في الـ DB
- الـ Down بتتنفذ لما تعمل Remove أو بمعنى أصح عايز تمسح الـ Migration فهو بيقوم رايح للـ Down ومنفذ الحاجات اللي فيه اللي بتكون عكس أي حاجة في الـ Up

لما بتعمل أول **Migration** الـ EFCore بيعمل جدول في الـ DB اسمه `__EFMigrationsHistory` بيتحط فيه أسامي كل الماجريشن عشان يعرف الداتا بيز هي واصلة لحد فين

**عشان كدا في قاعدة مهمة جدا جدا جدا أنك أياك تمسح الـ Migration بإيدك**

لأن هي بتكون مربوطة بالداتا بيز وبملف Snapshot فلما تمسح مانيول أنت مش بتعرفهم بالتغييرات
****
## ModelSnapshot
في ملف `ApplicationDbContextModelSnapshot.cs` بيتكريت مرة واحدة بس بيكون جواه الـ **Blueprint** لكل الـ **Schema** بتاعتك ببساطة هو موجود عشان لما تعمل Add Migration الـ EF Core ميضطرش يروح للداتا بيز عشان يقرأ الجدول ويشوف هل الـ Migration ده عمل حاجة زيادة ولا لا عشان يقرر أنه ده تغيير ولا لا 
****
## Custom Migration To Seed Data
ممكن تكتب أوامر SQL بسيطة تتنفذ بمجرد ما تستحدم الـ DB ودي حاجة مفيدة جدا عشان لما تنقل من داتا بيز للتانية متفضلش تحط داتا من الصفر كل مرة 

أنت هتعمل Migration فاضي وتكتب SQL زي كدا:

```cs
public partial class custommigration : Migration
    {
        protected override void Up(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.Sql(@"
                INSERT INTO ""Students"" (""Id"", ""Name"", ""Age"") 
                VALUES (gen_random_uuid(), 'Ahmed', 20);
            ");
        }
        
        protected override void Down(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.Sql(@"DELETE FROM ""Students"" WHERE ""Name"" = 'Ahmed';");
        }
    }
}
```

كدا لما تعمل Update Database الـ Row ده هيتضاف بشكل تلقائي
****
## Generate SQL Scripts
في الـ Dev بنعمل `database update` وخلاص.

بس في الـ Production مسئولين الـ DBAs والمطورين بيخافوا يخلو الـ EF Core يتعامل في داتا بيز البرودكشن أوتوماتيك لسببين

1. عايزين يراجعوا كل سطر SQL بايديهم عشان مفيش حاجة تقع في الداتا بيز.
2. سيرفر البرودكشن مش بيكون عليه .NET SDK أصلا عشان يرن أوامر EF Core.

طب ايه الحل؟
بنخلي الـ EF Core يطلع ملف `.sql`  بناخده نترنه في الـ DB Tool أو الـ Pipeline.

الأمر العادي:

```bash
dotnet ef migrations script
```

الأمر في البرودكشن (**Idempotent**):

```bash
dotnet ef migrations script --idempotent
```

وده أصلح واحد، لأنه بيكتب شروط `IF NOT EXISTS` جوه السكريبت ويشيك على جدول `__EFMigrationsHistory`؛ يعني لو المايجريشن مطبقة هيعديها، ولو جديدة هيطبقها من غير ما يضرب ايرور. 
****
## Database Scaffolding
لو عندك DB هي أصلا موجودة يعني **Database First** أنت عايز تحول الـ DB دي بقى لكود EF Core عشان تقدر تتعامل معاه في الكود ده الموضوع ببساطة

ودي عملية الـ **Scaffolding** أو ممكن تتسمى **Reverse Engineering**
![](https://res.cloudinary.com/du7j80lol/image/upload/v1793017688/obsidian/f6uzv8aboqgmkh6ppa3y.png)
