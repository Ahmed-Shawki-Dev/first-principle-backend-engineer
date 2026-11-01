لو سيبت الـ Model زي ما هو، هيشتغل بالإعدادات الإفتراضية بتاعة الفريمورك. فلو في حاجة عايز تعدلها، ببساطة هنستعمل الـ Data Annotation عشان نحدد الخصائص والحاجة اللي أحنا عايزنها، مش شرط تستعملها بس لازم تكون عارفها:

```cs
[Table("tbl_Students")] // Choose Table Name
public class Student
{
    [Key] // Primary Key To Table
    public Guid Id { get; set; }

    [Required]
    [MaxLength(100)]
    [Column("StudentName", TypeName = "varchar(100)")] // Choose Column Type And Name 
    public string Name { get; set; } = string.Empty;

    [Range(18, 60)] 
    public int Age { get; set; }

    [NotMapped] // Dont Include It As Property In Table
    public string FullInfo => $"{Name} - {Age}";
}
```

بس هكلمك بصراحة ده مش أحسن حل، كـ **separation of concerns** دي مش حاجة لطيفة.

طب ايه الحل يعم اللطيف؟ هنستعمل الـ `FluentApi` جوا Class الـ `ApplicationDbContext` بنحط الميثود دي

```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        
    }
```

وهنا بنعدل براحتنا بقى ونضيف كل الخصائص لكل الـ Models بشكل كامل 

بس مشكلتها أنها هتخلي الـ Class كبير ومعقد جدا، فأحنا هنعرف بردو أزاي نعملها في ملف مستقل أو أنك تعمل لكل Domain ملف مستقل هيكون أنضف اكتر
****
## Separate ModelBuilder Validation
ده مثال بسيط جدا للتعديلات اللي تعملها:

```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Student>().Property(s => s.Name).IsRequired();
    }
```

هنا أنت بتقول للـ DB أن تخلي الـ **Property** دي بـ `NOT NULL` 

بس في الشغل هتلاقي مثلا أنك لـ Student بس محتاج مثلا تعمل 10 ولا 20 تعديل تضيف **Indexes** بقى تعدل Property وهكذا 

فأنا عايز دلوقتي أفصل كل Model لذاته واقعد أعدل عليه وفي الآخر اضيفه في الـ `ModelBuilder` كسطر واحد ده أزاي؟

بص يسطا أنت لما تحب تأكسس على حاجة بتدخل بـ `Entity` فاللي هنعمله دلوقتي أننا هندخل عليها بشكل خاص ونلعب بقى نضيف اللي عايزينه بص:

```cs
public class StudentConfiguration:IEntityTypeConfiguration<Student>
{
    public void Configure(EntityTypeBuilder<Student> builder)
    {
        builder.Property(s => s.Name).IsRequired();
    }
}
```

وفي الآخر بتروح للـ `OnModelCreating` تنفذ بقى اللي أنت عملته بطريقة سهلة جدا لكل الـ Classes وكل التعديلات بالشكل ده:

```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{    modelBuilder.ApplyConfigurationsFromAssembly(typeof(ApplicationDbContext).Assembly);
}
```

طبعا المثال يبان بسيط بس هتيجي في وقت عايز تحط 20 تعديل جوا الـ Student وهتحتاجه جدا 