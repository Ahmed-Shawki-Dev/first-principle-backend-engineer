هنتكلم في الصفحة دي عن العمليات الخاصة بالإضافة والتعديل والمسح يعني أزاي بتبعت للداتا بيز وكدا
****
## Add Data To DB
ببساطة دي الطريقة
```cs
private static async Task Main(string[] args)
    {
        var context = new ApplicationDbContext();
        // Start Coding
        var student = new Student { Name = "Ali" };
        await context.Students.AddAsync(student);
        await context.SaveChangesAsync();
    }
```

ولو عايز تضيف **List** يعني أكتر من طالب أو **Entity** بتبقى كدا مش بتعمل For Loop
```cs
private static async Task Main(string[] args)
    {
        var context = new ApplicationDbContext();
        // Start Coding
        List<Student> students = [new() { Name = "Ali" }, new() { Name = "Ahmed" }];
        await context.Students.AddRangeAsync(students);
        await context.SaveChangesAsync();
    }
    }
```

في نقطة في الـ Add مهمة جدا جدا وهي أنك لو عندك جدول جواه **Navigation** لجدول تاني مثلا الـ `Enrollment` جواه نافيجيشن للـ `Student` تقدر تضيف `Student` بشكل تلقائي وأنت بتضيف الـ `Enrollment` أو تضيف مجموعة طلاب لو بيسمح

والميزة هنا أنه الـ **Enrollment** هياخد `Id` الـ `Student` اللي هيطلع بشكل تلقائي من غير تدخل منك
```cs
private static async Task Main(string[] args)
    {
        var context = new ApplicationDbContext();
        // Start Coding
        var group = await context.Groups.FirstAsync();
        var enrollment = new Enrollment
        {
            Group = group,
            Student = new Student { Name = "Momen" }
        };
        await context.Enrollments.AddAsync(enrollment);
        await context.SaveChangesAsync();
    }
```
****
## Update Data
في كذا طريقة للـ Update بس الطريقة الأحسن هي انك تجيب الداتا من الـ DB وتعدل فيها وتنادي `SaveChangesAsync`
```cs
var student = await context.Students.FindAsync(studentId);

if (student != null)
{
    student.Name = "Ali Updated"; // In-Memory Update
    await context.SaveChangesAsync(); // Tracking The Changes
}
```

في طريقة أنك تستعمل الـ `Update` فكرة الـ **Update** دي إنك بتستعمل ميثود `Update()` من غير ما تجيب الأوبجيكت من الداتا بيز بـ Query لأنك أنت اللي بتعمله بنفسك بـ `new`:
```cs
private static async Task Main(string[] args)
    {
        var context = new ApplicationDbContext();
        // Start Coding
        var student = new Student
        {
            Id = 1,
            Name = "Hazem Ahmed"
        };

        context.Update(student);
        await context.SaveChangesAsync();
    }
```

هنا بقى الطريقة دي محدودة جدا وفيها مشاكل لأن EF Core بيعمل `UPDATE` لكل العواميد اللي في الجدول بلا استثناء فأي حاجة أنت مش بتحطها في الأوبجيكت بتتحول لـ Null أو بترجع للـ **Default Value** بتاع نوعها وتتمسح البيانات القديمة اللي كانت في الداتا بيز فلازم تركز جدا وأنت بتكتبها
****
## Remove Data
دي حاجة سهلة جدا هتختار الـ Row من الـ DB وبعدها تعمله Remove
```cs
private static async Task Main(string[] args)
    {
        var context = new ApplicationDbContext();
        // Start Coding
        var student = await context.Students.FindAsync(1);

        context.Remove(student);
        await context.SaveChangesAsync();
    }
```

ولو عايز تمسح أكتر من حاجة مثلا هتعمل:

```cs
private static async Task Main(string[] args)
    {
        var context = new ApplicationDbContext();
        // Start Coding
        var students = await context.Students.Where(s=>s.Id>10).ToListAsync();

        context.RemoveRange(students);
        await context.SaveChangesAsync();
    }
```