أوقات بنكون عاملين شرط غالبا لازم يتطبق على كله ومش عايز تكرره كل مرة تكتب نفس الحاجة
مثلا زي أنه متعرضش الـ `IsDeleted` وده جزء من حوارات الـ Soft Delete

فأزاي نعمل **Filter** عام يتعامل هو من غير تدخل منك وأنك تكرر الـ Query كل مرة وكمان أزاي تلغيه في مكان أو كويري معينة
****
لو شغال عادي هتعمل كدا كل مرة:

```cs
var students = await 
context.Students.Where(s=>!s.IsDeleted).ToListAsync();
```

إنما في الجلوبل فلتر هتروح للـ `modelBuilder` تعمل التالي:

```cs
modelBuilder.Entity<Student>().HasQueryFilter(s => !s.IsDeleted);
```

بكدا هيتطبق على كل الجدول وأي كويري هتعملها هيتشال منها تلقائي أي حاجة `IsDeleted = true` 
ولو في API معين عايز تلغي الشرط ده فيه

```cs
var students = await context.Students.IgnoreQueryFilters().ToListAsync();
```

بكدا هيلغي كل الـ Filters الموجودة
****
