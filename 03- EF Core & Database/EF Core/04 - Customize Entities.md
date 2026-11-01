دي شوية Tweaks عامة موجودة للمعرفة بس ومختلفة
****
## Change Table Name
لو عايز تعمل اسم الـ Class بأسم معين بس عايزه يتحط في الداتا بيز بأسم Table مختلف
ببساطة يأما بالـ Data Annotation يأما بالـ FluentApi عن طريق
```cs
[Table("People")]
// Or in FluentApi
modelBuilder.Entity<Student>().ToTable("People");
```
****
## Change Schema Name
في Postgres بيبقى by default بتبقى **public** فأنت أزاي تعمل كذا Schema أو تحط Entity معينة في Schema مختلفة

```cs
[Table("Children",Schema ="students" )]
// Or in FluentApi
modelBuilder.Entity<Student>().ToTable("Children", schema: "students");
```
****
## Exclude Properties
لو في بروبرتي أنت ضايفها بس مش عايزها تظهر في الداتا بيز أو تكون Property فيها عايزها بس للـ EF Core ببساطة هتعمل
```cs
[NotMapped] public DateTime CreatedAt { get; set; }
// Or in FluentApi
modelBuilder.Entity<Student>().Ignore(s => s.CreatedAt);
```
****
## Change Column Name
لو عندك بروبرتي عايز تحطها في الكود بشكل وفي الداتا بيز بشكل مختلف
```cs
[Column("AddedOn")]
// Or in FluentApi
modelBuilder.Entity<Student>().Property(s => s.CreatedAt).HasColumnName("AddedOn");
```
****
## Change Column Data Types
لو عايز تعدل الـ Type مثلا بدل int في الكود يكون decimal أو تعدل الـ Decimal ذات نفسه تخليه مثلا ياخد شكل معين وهكذا

```cs
[Column(TypeName = "decimal(18,2)")]
// Or in FluentApi
modelBuilder.Entity<Student>().Property(s => s.Age).HasColumnType("decimal(18,2)");
```
****
## Set Primary Key
في البداية أي حاجة اسمها `Id` أو متضمنة كلمة `Id` بتبقى هي الـ Primary Key By Default
طب تخيل أنت مش عايزه
```cs
[Keyless]
public class Student
{
    public string Name { get; set; } = string.Empty;
    public int Age { get; set; }
}
```

بس كدا سهلة، طب لو عايز دلوقتي تخلي الـ Name هو الـ Primary Key:
```cs
[Key]
    public string Name { get; set; } = string.Empty;
    // Or in FluentApi
    modelBuilder.Entity<Student>().HasIndex(s => s.Name);
```
****
## Set Composite Key
دي مهمة خصوصا لو ريليشن Many to Many أنك يكون عندك 2 مثلا بيمثلوا اليونيك كي بتاع ال Row
أنا على طول كنت بعملها كدا ودي الطريقة الوحيدة بالـ FluentApi عن طريق **Anonymous Object**
```cs
modelBuilder.Entity<Student>().HasKey(s=>new {s.Name,s.Age});

```
****
## Set Default Value
أنت لما بتعمل كدا:
```cs
public string Name { get; set; } = string.Empty;
```

بتحط **Default Value** للـ Property بس دي بتبقى على مستوى الـ C# بس مش في الداتا بيز يعني لو حد كتب حاجة في الداتا بيز مش هيحط حاجة بل ممكن يخليها `null` مثلا

إنما في الـ `FluentApi` ممكن تحط بقى أنت الديفولت للداتا بيز نفسها 
```cs
modelBuilder.Entity<Student>().Property(s => s.Name).HasDefaultValue("Layla");
```
****
