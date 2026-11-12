## Phase 03: EF Core & Database
**Dates:** Oct 26 – Nov 22, 2026
**Duration:** 4 weeks
### Goal
Understand the complete chain:
C# → EF Core → SQL → PostgreSQL Planner → Indexes → Locks → Storage
****
## EF Core 
هتراجع على الـ Loading Strategies
#### Day 1: Setup & Modeling Attributes
- **Videos:** 01 to 23 (23 videos)
- **Duration:** ~125 mins
- **Topics:** DbContext, Migrations, Fluent API vs Data Annotations, Key Configurations, Default Values.
- [x] Watch & Apply (Videos 01-23)
---
#### Day 2: Relationships, Indexes & Seeding
- **Videos:** 24 to 37 (14 videos)
- **Duration:** ~105 mins
- **Topics:** One-to-One, One-to-Many, Many-to-Many, Indexes, Seeding, Scaffolding.
- [x] Watch & Apply (Videos 24-37)
---
#### Day 3: LINQ Queries & Aggregations
- **Videos:** 38 to 53 (16 videos)
- **Duration:** ~92 mins
- **Topics:** Find, First, Single, Where, Sorting, Projections, GroupBy, Joins.
- [x] Watch & Apply (Videos 38-53)
---
#### Day 4: Tracking, Loading Strategies & Performance
- **Videos:** 54 to 62 (9 videos)
- **Duration:** ~67 mins
- **Topics:** Tracking vs NoTracking, Eager/Explicit/Lazy Loading, Split Queries, Global Filters.
- [x] Watch & Apply (Videos 54-62)
---
#### Day 5: CUD, Transactions & Modern EF Features
- **Videos:** 63 to 70 (8 videos)
- **Duration:** ~65 mins
- **Topics:** Save Related Data, Cascade Delete, Transactions, Raw SQL, Bulk Delete/Update.
- [ ] Watch & Apply (Videos 63-70)
---
### 🛠️ Deep Topics (Not in Playlist - To Cover Separately)
These are internal/advanced concepts from Phase 3 roadmap that the playlist doesn't cover:
- [ ] **Change Tracker Internals** *(Snapshot vs Notification, DetectChanges mechanics)*
- [ ] **LINQ Translation Mechanics** *(Expression Trees to SQL translation)*
- [ ] **The N+1 Problem** *(Diagnosis & real-world profiling)*
- [ ] **Value Converters** *(Complex types mapping)*
- [ ] **Interceptors** *(DbCommand logging & auditing)*
- [ ] **DbContext Pooling** *(`AddDbContextPool` for performance)*
****
## PostgreSQL
- Constraints ✅
- B-Tree indexes
- Dates, JSONB
- Composite indexes
- Index selection
- EXPLAIN
- EXPLAIN ANALYZE
- Query plans
- Sequential scans
- Transactions
- ACID
- Isolation levels
- Locks
- Deadlocks
- MVCC
- Connection pooling
- Partitioning basics
- Replication basics
- Backup & recovery basics

## Concurrency
- Optimistic concurrency
- Pessimistic concurrency
- PostgreSQL concurrency model
- EF Core concurrency handling

## Milestone
- [ ] Explain what happens when LINQ becomes SQL
- [ ] Diagnose an N+1 query
- [ ] Choose between tracking, no-tracking, and projection
- [ ] Read an EXPLAIN ANALYZE plan
- [ ] Design useful single and composite indexes
- [ ] Explain locks and MVCC
- [ ] Reproduce and diagnose a deadlock
- [ ] Explain transaction isolation levels
- [ ] Implement optimistic concurrency in a real feature
- [ ] Analyze at least 3 real queries from an existing project