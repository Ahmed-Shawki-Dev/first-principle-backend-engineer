بنحتاج أوقات كتير لما تشتغل يكون في بعض الداتا موجودة في الـ DB والـ Tables بتاعتك بمجرد الـ **Creation** اوتوماتيك، وده اللي اسمه **Data Seeding** 

عندك نوعين منه 
- **Migration-based Seeding**
بنستخدمه للبيانات الثابتة جدا اللي الأبلكيشن مينفعش يشتغل من غيرها زي مثلا قائمة الدول أو العملات
- **Runtime Seeding**
هنا بنستخدمه عشان لما السيرفر يشتغل في أي Environment يكون في داتا أولية تجرب فيها ومش كل مرة تقعد تحط داتا من الصفر
****
## Migration-based Seeding Using Fluent API
هنا هنتعلم أزاي نحط الـ Data ألاساسية جدا، زي ما قولنا مثلا عملات وهكذا دي الأبلكيشن مينفعش يكون موجود من غيرها:

```cs
modelBuilder.Entity<Currency>().HasData(
    new Currency { Id = 1, Code = "EGP", Name = "Egyptian Pound" },
    new Currency { Id = 2, Code = "USD", Name = "US Dollar" }
);
```
****
## Runtime Seeding
هنا أنت بتكون عايز تعمل Seed لداتا أول ما البروجيكت يفتح أو السيرفر يشتغل تتشيك وتشوف لو موجودة متضفهاش لو مش موجودة ضيفها ودي مفيدة جدا خصوصا في البرودكشن لأن مش هيكون ليك Access مباشر على كل حاجة فبتكتب والكود هو اللي بيضبط الدنيا لوحده

```cs
public static class DataSeeding
{
    public static async Task SeedDataAsync(ApplicationDbContext context)
    {
        // 1. Check All Migration Applied
        await context.Database.MigrateAsync();

        // 2. To Not Repeat Roles
        if (!await context.Roles.AnyAsync())
        {
            var roles = new List<Role>
            {
                new() { Id = Guid.NewGuid(), Title = "SuperAdmin" },
                new() { Id = Guid.NewGuid(), Title = "User" }
            };

            await context.Roles.AddRangeAsync(roles);
            await context.SaveChangesAsync();
        }
    }
}
```

ده مثلا مثال بيخليك تعمل رولز بس بيتأكد أن حصل Migration الاول ولا لا وبعدها يشوف لو مفيش رولز في الداتا بيز يقوم عامل هو وحافظ وألف مبروك

**بس كدا مش هيشتغل خالص**
لأزم تعرف الـ `Program.cs` أن الكلاس ده موجود

```cs
using (var scope = app.Services.CreateScope())
{
    var context = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
    await DataSeeding.SeedDataAsync(context);
}
```

طب هو ليه لازم نضيف حاجة زي كدا ليه مضيفنهاش كخدمة بـ `AddScoped` مثلا؟

أحنا عارفين أن الـ `DbContext` متسجلة كـ Scoped، فأحنا دلوقتي عايزين نـ **Seed** فيها داتا
بس أول ما السيرفر يفتح مش بيكون في أي `HttpContext` أو ريكوست جاي

فلو نادينا الـ DbContext بشكل مباشر هيقولك Error لأن مفيش أصلا لأن مفيش لسه HttpRequest جه 

عشان كدا بنعمل `app.Services.CreateScope()` بنخلق **Scope** وهمي بإيدينا ونطلب منه الـ `DbContext` عشان ينفذ عملية الـ **Seeding** وخلاص 

واستخدمنا `using` عشان أول ما يخلص الـ **Seeding** الـ **Scope** يتمسح تماما من الرام والأبلكيشن يرجع زي الأول فهو بتشتغل أول ما السيرفر يحصله **Run** بس