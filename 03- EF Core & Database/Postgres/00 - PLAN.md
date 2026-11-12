## PostgreSQL & Concurrency — 3-Week Roadmap
- **بيئة التطبيق:** جدول تجريبي فيه مليون صف (`generate_series`) عشان تشوف الفرق الحقيقي في الأداء.
****
### الأسبوع الأول: أساسيات التخزين ومحرك البحث (Storage & Indexing)

#### اليوم 1: مراجعة + Dates & Timezones
- [x] **Constraints** 
- [ ] **Dates & Timezones (`TIMESTAMPTZ`):**
  - **الهدف:** تفهم ليه `TIMESTAMP` العادي بيعمل كوارث في السيستمز، وإزاي بوستجريس بتخزن UTC وتعرض حسب الـ Timezone، وربط ده بـ `DateTimeOffset` في C#.

#### اليوم 2: JSONB
- [ ] **JSONB & GIN Indexes:**
  - **الهدف:** تعرف إمتى تخزن داتا مرنة جوه عمود `JSONB` (الفرق بينه وبين `JSON` في التخزين الثنائي)، وإزاي تسيرش جواه بسرعة باستخدام `GIN Index`.

#### اليوم 3: طريقة قراءة الداتابيز للداتا
- [ ] **Sequential Scans vs Index Scans:**
  - **الهدف:** تفهم إزاي بوستجريس بتجيب الصفوف من الـ Disk، وليه ساعات قراءة الجدول كله (Seq Scan) بتكون أسرع من استخدام الـ Index.

#### اليوم 4: الفهارس (B-Tree & Composite)
- [ ] **B-Tree Indexes & Composite Indexes:**
  - **الهدف:** تفهم فكرة شجرة الـ B-Tree، وإزاي تعمل Index على أكتر من عمود، وأهمية ترتيب الأعمدة جواه (Left-most prefix rule).

#### اليوم 5: اختيار الفهارس وتحليل الكويري
- [ ] **Index Selection:** إمتى الـ Query Planner بيختار Index وإمتى بيطنشه (زي استخدام دوال على العمود في الـ `WHERE`).
- [ ] **EXPLAIN & EXPLAIN ANALYZE & Query Plans:** الفرق بين التوقع والتنفيذ الفعلي، وإزاي تقرأ الخطة (Cost, Rows, Execution Time) من جوه لبره.

*(اليوم 6 و 7: راحة أو تطبيق عملي حر على جدول المليون صف)*

---
### الأسبوع الثاني: قلب الداتابيز والتزامن (Transactions & Concurrency)

#### اليوم 1: أساس المعاملات
- [ ] **Transactions & ACID:**
  - **الهدف:** تفهم كل حرف في `ACID` بيحصل إزاي عملياً جوه بوستجريس، وإزاي `COMMIT` و `ROLLBACK` بيحافظوا على سلامة الداتا.

#### اليوم 2: روح بوستجريس (MVCC)
- [ ] **PostgreSQL Concurrency Model & MVCC:**
  - **الهدف:** تفهم إزاي ناس بتقرأ وناس بتكتب في نفس اللحظة من غير ما يقفلوا على بعض (Multi-Version Concurrency Control)، وفكرة الـ Snapshots، ودور الـ `VACUUM` في مسح النسخ القديمة.

#### اليوم 3: مستويات العزل
- [ ] **Isolation Levels:**
  - **الهدف:** تفهم المشاكل اللي بتحصل في التزامن (`Dirty Read`, `Non-Repeatable Read`, `Phantom Read`) وإزاي المستويات (`Read Committed`, `Repeatable Read`, `Serializable`) بتمنعها.

#### اليوم 4: الأقفال والاختناقات
- [ ] **Locks & Deadlocks:**
  - **الهدف:** تفرق بين قفل الصف (Row-level) وقفل الجدول (Table-level)، وتجرب بإيدك تعمل Deadlock بين اتنين Terminals وتشوف بوستجريس بتكسره إزاي.

#### اليوم 5: استراتيجيات التزامن + التطبيق في .NET
- [ ] **Optimistic vs Pessimistic Concurrency:** الفرق بين قفل الصف بدري (`SELECT ... FOR UPDATE`) أو كشف التضارب وقت الحفظ.
- [ ] **EF Core Concurrency Handling:** ربط كل اللي فات بالكود؛ استخدام عمود السيستم `xmin` في بوستجريس كـ Concurrency Token جوه EF Core وهندلة `DbUpdateConcurrencyException`.

*(اليوم 6 و 7: راحة أو تجربة سيناريو Concurrency على مشروع قديم عندك)*

---
### الأسبوع الثالث: التشغيل والتوسع (Scaling & Operations Basics)

#### اليوم 1: إدارة الاتصالات
- [ ] **Connection Pooling:**
  - **الهدف:** تفهم تكلفة فتح اتصال جديد للداتابيز، والفرق بين الـ Pooling في `Npgsql` (على مستوى التطبيق) و `PgBouncer` (على مستوى السيرفر).

#### اليوم 2: تقسيم الجداول الضخمة
- [ ] **Partitioning Basics:**
  - **الهدف:** تفهم إزاي جدول فيه 100 مليون صف بيتقسم في الخلفية لجداول أصغر (بالتاريخ أو الـ Tenant ID) مع الحفاظ على التعامل معاه كجدول واحد في الكود.

#### اليوم 3: النسخ الاحتياطي اللحظي والتوزيع
- [ ] **Replication Basics:**
  - **الهدف:** تفهم الفرق بين السيرفر الأساسي (Primary) وسيرفرات القراءة (Read Replicas)، وفكرة الـ `WAL` (Write-Ahead Logging) اللي بتنقل الداتا بينهم.

#### اليوم 4: الحماية والاسترجاع من الكوارث
- [ ] **Backup & Recovery Basics:**
  - **الهدف:** تفرق بين النسخ المنطقي (`pg_dump`) والنسخ الفيزيائي، وتفهم مفهوم الاسترجاع لنقطة زمنية محددة (Point-in-Time Recovery - PITR).

#### اليوم 5: تجميع الخيوط (Review & Synthesis)
- [ ] **مراجعة شاملة:** مراجعة نوتس Obsidian، واختيار حتة واحدة من اللي اتعلمته تطبقها كـ Refactoring في مشروع من مشاريعك.