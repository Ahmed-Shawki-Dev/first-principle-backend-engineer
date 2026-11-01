## PostgreSQL Basics — Pre-Midterm Week

الهدف: تفهم إزاي PostgreSQL بتحافظ على صحة البيانات، وبتتعامل مع أنواعها، وبتوصلها بكفاءة من خلال الـ Indexes.

### خطة الأسبوع

#### Day 1 & 2 — Constraints & Primary Keys

الهدف: تفهم إزاي الداتابيز بتضمن صحة البيانات.

- Primary Key & Foreign Key
- Unique & Check Constraints
- NOT NULL
- UUID vs Auto-increment IDs

#### Day 3 — Dates & Timezones

الهدف: تفهم إزاي تخزن التواريخ وتتعامل مع اختلاف التوقيت.

- TIMESTAMP vs TIMESTAMPTZ
- UTC Handling
- Timezone Conversion

#### Day 4 — JSONB Basics

الهدف: تفهم إزاي PostgreSQL بتتعامل مع البيانات المرنة.

- JSON vs JSONB
- JSONB Storage
- Basic JSONB Queries
- JSONB vs Relational Columns

#### Day 5 — B-Tree & Composite Indexes

الهدف: تفهم إزاي PostgreSQL بتوصل للبيانات بسرعة وإمتى تستخدم Index.

- B-Tree Mechanics
- Single vs Composite Index
- Selectivity
- Index Selection
- Index Scan vs Sequential Scan

### التغطية من الـ Roadmap الكبيرة

|الموضوع|التغطية|
|---|---|
|Constraints|أساسي|
|Dates|أساسي|
|UUIDs vs Auto-increment|أساسي|
|JSONB Basics|أساسي|
|B-Tree Indexes|مقدمة|
|Composite Indexes|مقدمة|
|Index Selection|مقدمة|

### بعد الميدتيرم

هتكمل باقي PostgreSQL:

- Query Performance: EXPLAIN, EXPLAIN ANALYZE, Query Plans.
- Transactions: ACID, Isolation Levels.
- Concurrency: Locks, Deadlocks, MVCC.
- Database Operations: Connection Pooling, Partitioning, Replication, Backup & Recovery.
- EF Core Concurrency: Optimistic & Pessimistic Concurrency.

الصورة الكبيرة: Data Integrity → Data Types → Indexing → Query Execution → Transactions & Concurrency.