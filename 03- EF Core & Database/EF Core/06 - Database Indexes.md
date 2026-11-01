ايه هو الـ **Index**؟
هو الفهرس، هنتكلم عنه في الـ Phase دي بشكل تفصيلي في الـ DB Section بس حاليا هنشوف الإستعمال المباشر، الأندكس من الآخر بيسرعلك عملية البحث وبيخلي الداتا بيز تجيبلك الحاجة في وقت أقصر كتير وده بخوارزميات وطرق معينة

هو مفيد جدا للـ Read بس ممكن يكون سيء للـ Write لو كترت منه يعني مينفعش تستخدم الـ Index في كل الأماكن

الـ EF Core في الماجريشن By Default بيعمل Index على الـ Foreign Keys

```cs
migrationBuilder.CreateIndex(
                name: "IX_Enrollment_GroupId",
                table: "Enrollment",
                column: "GroupId");
        }
```
****
## Create Index
عشان تعمل **Index** سهلة جدا بالطريقتين:

```cs
[Index(nameof(GroupCode))]
public class Group
{
    public Guid Id { get; init; }
    public int GroupCode { get; set; }
    public string Name { get; init; } = string.Empty;

    // Relation M-M
    public ICollection<Enrollment> Enrollments { get; set; }
}
// OR
modelBuilder.Entity<Group>().HasIndex(g => g.GroupCode);
```
****
## Create Composite Index
لو عايز الـ Index يكون مكون من 2 بروبرتي:

```cs
[Index(nameof(GroupCode), nameof(Name))]
// OR
modelBuilder.Entity<Group>().HasIndex(g => new{g.GroupCode,g.Name});
```
****
## Custom Index Name
لو عايز تحط أسم انت عايزه للـ Index:

```cs
[Index(nameof(GroupCode), Name = "Index_GroupCode")]
// OR
modelBuilder.Entity<Group>().HasIndex(g => g.GroupCode).HasDatabaseName("Index_GroupCode");
```