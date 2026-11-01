في السيكشن ده هنتكلم عن حاجات تخص البيرفورمنس أكتر وطرق تحميل الداتا وتتبعها
****
## Tracking 
دي ببساطة أنت لما تجيب داتا من الداتا بيز الـ EF Core باي ديفولت بيتراك الداتا دي يشوف هل عملت فيها تغييرات ولا لا مثلا لو كدا:

```cs
var studentId = Guid.Parse("00a0d72a-de29-4f50-ad4d-bcf402b64fc9");

var targetStudent = await context.Students
    .FirstOrDefaultAsync(s => s.Id == studentId);

if (targetStudent != null)
{
    targetStudent.Name = "Layla Ahmed";
    await context.SaveChangesAsync();
}
```

هنا الداتا اتغيرت لليلى عشان الـ Tracking شغال By Default 
إنما لو عملت 

```cs
var studentId = Guid.Parse("00a0d72a-de29-4f50-ad4d-bcf402b64fc9");

var targetStudent = await context.Students
    .AsNoTracking()
    .FirstOrDefaultAsync(s => s.Id == studentId);
```

هنا بقى بنستخدم دي للقراءة فقط لما تكون هتجيب داتا من الداتا بيز ومش هتعدل عليها يعني هتعمل GET API مثلا 

**الميزة هنا أنك هتاخد أداء أقوى بكتير لما متعملش تراك للداتا فلازم تستعملها لو مش هتعدل في الداتا اللي جايالك وهتعرضها على طول**
****
## Loading Strategies
هي الطرق اللي بتقول بيها للـ `DbContext` أزاي يسحب الداتا المرتبطة ببعض في الداتا بيز

تخيل عندك جدول *Student* و *Group* العلاقة بينهم `1 - M`
### Eager Loading 
هنا أنت بتحمل مباشرة بتقول للـ `EF Core` وأنت بتجيب الطالب هات معاه الجروب بتاعه في نفس الـ **Query** من الداتا بيز مرة واحدة وهنا بيترجم في SQL لـ `LEFT JOIN` و `INNER JOIN` بنستخدم هنا بقى الـ `Include()` و `ThenInclude()` 

```cs
var studentWithGroup = await context.Students
    .Include(s => s.Group)
    .FirstOrDefaultAsync(s => s.Id == studentId);
```

### Explicit Loading
اسمه التحميل الصريح أنت بتروح تجيب الطالب من الداتا بيز وبعديها قدام لو احتاجت الجروب بتقول للـ EF Core يروح للداتا بيز تاني مخصوص يجيب الجروب

الطريقة دي تقريبا مش بنستعملها كتير بس هي موجودة بتبقى كدا:

```cs
private static async Task Main(string[] args)
    {
        var context = new ApplicationDbContext();

        var enrollments = await context.Enrollments.ToListAsync();
        foreach (var enrollment in enrollments)
            await context.Entry(enrollment)
                .Reference(e => e.Student)
                .LoadAsync();

        foreach (var enrollment in enrollments)
            Console.WriteLine($"{enrollment.Student.Name}");
    }
```

### Lazy Loading
في الـ Eager Loading كنا بنعمل Include لأي حاجة لو عايزين نخليها معانا
هنا بقى لا هي بتيجي لما تطلب
يعني طول ما أنت عامل مثلا
`student.Name`
مش هيجيب غير الطلاب
لما تكتب بقى
`student.Group.Name`
هيجيبلك اسم الجروب 

بس الطريقة دي مش بتشتغل عادي لازملها شروط ومكتبة
هي دي:
```
Microsoft.EntityFrameworkCore.Proxies
```

ولازم نخلي كل الـ **Navigation Proerties** تكون **virtual** عشان المكتبة تشتغل

طبعا دي مش أحسن طريقة عشان بتسبب مشاكل `N+1` لو شغال على أراي وهكذا أو اكتر من داتا راجعة من الداتا بيز والداتا دي راجع منها داتا
****
