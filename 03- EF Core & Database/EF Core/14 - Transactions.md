أوقات كتير بتحتاج تنفذ أكتر من عملية على الـ DB بس بتكون عايز أن الكل يتنفذ مع بعضه، يعني يا الكل يتنفذ يا مفيش أي حاجة تتنفذ هنا هتسمع مصطلح الـ **Transactions** وده اللي هنتكلم عنه 
وكمان هنتكلم لو يعم دلوقتي أنت عندك **Transactions** طويلة وحابب لو حاجة ضربت ينفذ جزء معين بس ويوقف الباقي بمعنى انه مثلا لو في 3 خطوات ينفذ واحدة ويخلي الاتنين الأخرانين ميتنفذوش 
****
## Transactions
زي ما قولنا فوق وطريقة تنفيذها في دوت نت سهلة

```cs
private static void Main(string[] args)
    {
        var context = new ApplicationDbContext();
        // Start Coding
        using var transaction = context.Database.BeginTransaction();
        try
        {
            context.Students.Add(new Student { Name = "ayman" });
            context.SaveChanges();
            context.Students.Add(new Student { Name = "shaaban" });
            context.SaveChanges();

            transaction.Commit();
        }
        catch (Exception e)
        {
            transaction.Rollback();
        }
    }
```

هنا بتفتح **transaction** ب `using` عشان يضمن إنه يقفل الموارد والـ **Connection** ويعمل **Dispose** تلقائيا أول ما نخلص أو لو حصل **Exception** وبعدها بتعمل `trycatch` بتحط اللي أنت عايزه ولو نجح يعمل **Commit** لو فشل حاجة منهم مش هينفذ الكل وهيروح يعمل `Rollback` يعني يفشل
****
## SavePoint
دي ببساطة بتحط زي نقطة لو الفشل حصل بعدها ينفذ اللي قبلها عادي وكأنك بتقولي احفظلي دي أنا عايزها بس اللي تحت لو حاجة منهم باظت وقف اللي تحت كله

```cs
private static void Main(string[] args)
    {
        var context = new ApplicationDbContext();
        // Start Coding
        using var transaction = context.Database.BeginTransaction();
        try
        {
            context.Students.Add(new Student { Name = "ayman" });
            context.SaveChanges();

            transaction.CreateSavepoint("Stop");

            context.Students.Add(new Student { Name = "shaaban" });
            context.SaveChanges();

            transaction.Commit();
        }
        catch (Exception e)
        {
            transaction.RollbackToSavepoint("Stop");
            transaction.Commit();
        }
    }
```