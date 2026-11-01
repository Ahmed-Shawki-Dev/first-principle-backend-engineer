في الجزء ده هنتكلم عن الطريقة الحديثة والسريعة اللي نزلت في دوت نت 7 عشان نعمل عمليات تعديل أو مسح لجزء كبير من البيانات في الداتا بيز بضربة واحدة ومن غير ما نحمل الداتا في الميموري ولا نستهلك الـ Change Tracker

زمان لما كنت بتعوز تمسح مجموعة طلاب كنت بتضطر تعمل كويري تجيب الطلاب كلهم الأول في الميموري وبعدين تنادي `RemoveRange` وبعدها `SaveChangesAsync` وده كان بيبعت أمر **DELETE** منفصل لكل سطر في الداتا بيز وبيستهلك رامات وقت كبير أوي
****
## ExecuteDeleteAsync
أول حاجة الـ `ExecuteDeleteAsync` دي بنستعملها لما نكون عايزين نمسح داتا مباشرة في الداتا بيز بـ SQL Command واحد أسرع بكتير ومن غير ما نجيب حاجة في الميموري

```cs
private static async Task Main(string[] args)
{
    var context = new ApplicationDbContext();
    // Start Coding

    await context.Students
        .Where(s => s.IsDeleted)
        .ExecuteDeleteAsync();
}
```

هنا الـ EF Core بياخد الكويري وبيترجمها لـ **SQL DELETE** مباشر وبيروح ينفذها على طول في الداتا بيز ومن غير ما تحتاج تنادي `SaveChangesAsync` لأن الأداة دي بتضرب في الداتا بيز فوري
****
## ExecuteUpdateAsync
تاني حاجة الـ `ExecuteUpdateAsync` وهنا بدل ما تروح تجيب الاوبجيكت وتعدل الخواص بتاعته وبعدها تنادي `SaveChangesAsync` عشان يعمل **Tracking** أنت بتعمل تعديل مباشر لخاصية معينة أو مجموعة خواص لجميع السطور اللي بتنطبق عليها الشروط بـ **SQL UPDATE** مباشر

```cs
private static async Task Main(string[] args)
{
    var context = new ApplicationDbContext();
    // Start Coding
    await context.Students
        .Where(s => s.IsDeleted)
        .ExecuteUpdateAsync(s => s.SetProperty(student => student.Name, "Archived Student"));
}
```

طبعا الميزة الكبيرة هنا إن التعديل بيحصل بـ Query واحدة مباشرة من غير ما نسحب داتا لرامات الجهاز ولا نستهلك الميموري بس لازم تاخد بالك إن الطرق دي بتتخطي الـ **Change Tracker** ومبتاخدش بالها من التغييرات اللي في الميموري وبتبعت للداتا بيز على طول وما بتحتاجش تنادي معاه `SaveChangesAsync` خالص