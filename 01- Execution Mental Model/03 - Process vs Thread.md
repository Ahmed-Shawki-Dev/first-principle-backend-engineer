الـ **Process** هي unit of execution يعني حاجة بتتنفذ في الـ OS بتكون independent و self-contained وبيكون ليها الـ **Memory Space** والـ **Resources**
المستقلة بتاعتها

إنما الـ **Thread** فهي أصغر unit of execution بتتم جوا الـ Process ذات نفسها
كل Process بيكون جواها Threads كتير وبيشيروا كلهم نفس الـ Resources ونفس الـ Memory Space

![](https://res.cloudinary.com/du7j80lol/image/upload/v1790577766/obsidian/gytpagj2jwcgmykbcf6l.png)
### Resource Allocation
الـ Process بتاخد System Resources مستقلة حاجات زي CPU Time و File Handles و Memory وكله مستقل وفي حاله وملوش دعوة بغيره 

الـ Threads بتشارك الـ Heap والـ Resources بتاعة الـ Process، بس كل Thread ليه الـ Call Stack المستقل بتاعه عشان يسجل مسار الدوال والـ Local Variables بتاعته.

### Communication
الـ Processes صعب يتواصلوا مع بعض بتحتاج ميكانيزم وسيط اسمه **IPC (Inter-Process Communication)**. 

إنما الـ Threads بيتواصلوا مباشرة عبر الـ Shared Memory، بس ضريبة ده إننا بنحتاج Synchronization عشان نمنع تضارب الداتا (Race Conditions).

### Isolation
الـ Process بتبقى Isolated بشكل كامل يعني لو حصل failure لبروسيس البروسيس التانيين مش هيتأثروا نهائي

إنما الـ Threads مش Fully Isolated يعني لو في مشكلة في one thread ممكن الـ Process كلها تتأثر

### Creation and Termination
عشان تعمل Process دي عملية معقدة وبتـ time-consuming أكتر بكتير من الـ Thread 
بينما الـ Thread فسهل انك تعمل وتنهي ثريد بسهولة
****

