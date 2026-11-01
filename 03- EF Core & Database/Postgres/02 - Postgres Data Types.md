دي أغلب الـ Data Types اللي موجودة في Postgres هنفهم أغلبها بإذن الله

![](https://res.cloudinary.com/du7j80lol/image/upload/v1793554275/obsidian/oilnnienbnd0yaorbmkd.png)
****
## TYPES 
1. **SERIAL**
هي Integer عادي بس Incremental يعني كل لما تضيف قيمة بيزيد واحد 
2. **INTEGER, BIGINT, SMALLINT**
انتجرز عاديين بس الميموري وكدا والحجم
3. **DECIMAL, NUMERIC**
هما تقريبا متطابقين مفيش فرق كبير بينهم بنستخدمها للـ Float مثلا
`DECIMAL(10,2)` 
يعني بتقول أنه بيشيل 10 ارقام جمب بعض من ضمنهم 2 بعد البوينت زي **12345678.96** لو هتخزن Price أو حاجة لازم تستعملهم
4. **CHAR, VARCHAR, TEXT**
الـ Text ببساطة تكتب اللي أنت عايزه بدون أي Length وده شيء مودرن وكويس *دايما استعمل Text في Postgres* 

متستعملش CHAR خالص إلا لو حاجة الـ Length فيها ثابت وعمره ما يتغير
****
