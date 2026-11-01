في أي Relational Database بيكون في Relationships بين الجداول وبعضها عشان نمنع الـ **Redundancy** ونحافظ على الـ **Data Integrity**

الـ EF Core بيفهم ويترجم العلاقات دي بين الـ Classes بـ **Foreign Keys** و **Constrains** في الداتا بيز عن طريق الـ **Navigation Properties** فهنا هنتكلم بشكل تفصيلي عن أنواع العلاقات كلها ونفهم الدنيا ماشية أزاي، هنتكلم عن:
- One To One
- One To Many
- Many To Many
حابب بس أذكر قبل ما نبدأ أن في الـ Relationships الأفضل نعتمد على الـ `Fluent API`
****
## One To One Relationship
عشان تعمل الريليشن صح محتاج تفهم أن الداتا بيز لازم تفرق بين الـ **Principal Entity** والـ **Dependent Entity** يعني تشوف مين الأساسي ومين التابع

لو عندك جدول `Student` و جدول `StudentIdentity` المنطقي جدا أن الطالب يكون لي ايدنتتي وممكن ميكنش له، بس مستحيل يكون في ايدنتتي من غير طالب فهنا الأساسي هو الطالب

وهنا في قاعدة أننا بنحط الـ **Foreign Key** في الـ **Dependent** بس مش بنحطه من الناحيتين عشان نمنع الـ **Circular Dependency**

وده كود توضيحي يبين الطريقة:
```cs
// Principal Entity 
public class Student
{
    public Guid Id { get; set; }
    public string Name { get; set; } = string.Empty;

    // Navigation Property Only !
    public StudentIdentity? StudentIdentity { get; set; }
}

// Dependent Entity
public class StudentIdentity
{
    public Guid Id { get; set; }
    public string StudentCode { get; set; } = string.Empty;

    // Foreign Key + Navigation Property
    public Guid StudentId { get; set; }
    public Student Student { get; set; } = null!;
}
```

وطبعا في الـ `FluentApi` لو مثلا مش مسمي أسم `StudentId` أو عايز تعتمد على Foreign Key مختلف أو ايا كان ممكن تعمل كدا:
```cs
builder.HasOne(s => s.StudentIdentity)
       .WithOne(si => si.Student)
       .HasForeignKey<StudentIdentity>(si => si.StudentId);
```

زي ما قولنا التابع بيعتمد على الأساسي مش العكس
****
## One To Many
بردو معتمدة على نفس فكرة الـ **Principal Entity** والـ **Dependent Entity** يعني تشوف مين الأساسي ومين التابع

مثلا عندنا جدول **Student** وجدول **Attendance** :

```cs
public class Student
{
    public Guid Id { get; init; }
    public string FirstName { get; init; } = string.Empty;
    public string LastName { get; init; } = string.Empty;

    // Relations
    public List<Attendance> Attendances { get; set; } = null!;
}

public class Attendance
{
    public Guid Id { get; set; }
    public bool IsAttend { get; set; } = false;

    // Relations
    public Guid StudentId { get; set; }
    public Student Student { get; set; } = null!;
}
```

لو مسمي أسم الـ Foreign Key مختلف أو مش ماشي على الـ Conventions وعايز تظبط الـ One To Many بالـ Fluent API فبتكتبها كدا:

```cs
builder.HasOne(a => a.Student)
       .WithMany(s => s.Attendances)
       .HasForeignKey(a => a.StudentId);
```

وهنا بتحدد مين اللي عنده الـ List ومين اللي شايل الـ Foreign Key صراحة والـ EF Core بيفهم أن الطالب لي كذا غياب والغياب دا تبع طالب واحد بس

**Notes**
- في الـ One To Many لما تمسح الأساسي اللي هو الطالب سلوك المسح بيعتمد على نوع الفورين كي لو الكي اجباري زي Guid عادية الداتا بيز بتمسح كل غيابات الطالب دا أوتوماتيك بـ **Cascade Delete** ولو الكي اختياري Nullable الغيابات بتفضل موجودة في الداتا بيز وبتحول الفورين كي لـ Null
- ولو شيلت الفورين كي والنافجيشن من كلاس الغياب وسبت اللستة بس عند الطالب الـ EF Core بيعمل العمود في الداتا بيز كـ **Shadow Property** والعلاقة بتفضل شغالة في السيرش والـ Include عادي جدا من غير ما تكتب الكي بإيدك
****
## Many To Many
زي مثلا الـ `Enrollment` بيبقى عندك حاجة معتمدة على كذا حاجة من Entity والـ Entity التانية معتمدة على كذا حاجة من الـ Entity الأولى 

مثلا الـ Student والـ Group ممكن الطالب يكون في كذا Group وأكيد الـ Group بيتكون من كذا Student

في الحالة دي بنعمل جدول وسيط أو **Junction Table** يتحط في الـ PK بتاع ده والـ PK بتاع ده

طب أزاي ده في EF Core؟

```cs
public class Student
{
    public Guid Id { get; init; }
    public string Name { get; init; } = string.Empty;

    // Relations
    public ICollection<Group> Groups { get; set; }
}

public class Group
{
    public Guid Id { get; init; }
    public string Name { get; init; } = string.Empty;

    // Relation M-M
    public ICollection<Student> Students { get; set; }
}
```

لو عملت كدا بس الـ EF Core ضمنيا هيعمل هو Junction Table ويعملك كل حاجة 
بس أحنا مش بنستعمل الطريقة دي وأحنا الأفضل أننا نعمل الجدول التالت بإيدنا وهوريك أزاي

بص أحنا هنعمل جدول تالت اسمه Enrollment

```cs
public class Enrollment
{
    public Student Student { get; set; }
    public Guid StudentId { get; set; }

    public Group Group { get; set; }
    public Guid GroupId { get; set; }
}
```

وهنخليه يكون Many في الـ Student والـ Group

```cs
public class Group
{
    public Guid Id { get; init; }
    public string Name { get; init; } = string.Empty;

    // Relation M-M
    public ICollection<Enrollment> Enrollments { get; set; }
}

public class Student
{
    public Guid Id { get; init; }
    public string Name { get; init; } = string.Empty;

    // Relations
    public ICollection<Enrollment> Enrollments { get; set; }
}
```

وفي الآخر هنروح للـ `Fluent API` نقوله:

```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Enrollment>().HasKey(e => new { e.StudentId, e.GroupId }); // Select Composite Key

        modelBuilder.Entity<Enrollment>().HasOne(e => e.Group).WithMany(s => s.Enrollments)
            .HasForeignKey(e => e.GroupId); // One-Many With Group

        modelBuilder.Entity<Enrollment>().HasOne(e => e.Student).WithMany(s => s.Enrollments)
            .HasForeignKey(e => e.StudentId); // 1-M With Student
    }
```