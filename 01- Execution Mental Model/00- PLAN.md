## Week 01 — Execution Mental Model

**Sat → Thu**
**Target:** 2–3 hours/day

### Saturday — Request Lifecycle

- Learn: HTTP Request / Response
- Learn: Client vs Server
- Learn: DNS + TCP basics
- Learn: Kestrel
- Draw the full request lifecycle

**Done when:** You can explain what happens when you call `GET /users`.

---

### Sunday — ASP.NET Core Pipeline

- Learn: Middleware
- Learn: Request Pipeline
- Learn: Routing
- Learn: Endpoint
- Connect it to yesterday's lifecycle

**Task:** Draw:

`Request → Middleware → Routing → Endpoint → Response`

**Done when:** You can explain what happens between Kestrel and your Controller/Endpoint.

---

### Monday — Process / Thread / CPU / I/O

- Learn: Process
- Learn: Thread
- Learn: CPU-bound
- Learn: I/O-bound
- Learn: Why DB calls are I/O
- Learn: async/await conceptually

**Task:** Classify:

- DB query
- HTTP request
- Image processing
- Hashing
- JSON serialization

**Done when:** You can explain CPU-bound vs I/O-bound without notes.

---

### Tuesday — Latency / Throughput

- Learn: Latency
- Learn: Throughput
- Learn: Requests per second
- Learn: Bottleneck
- Learn: p95 latency — basic concept

**Task:** Write the difference between latency and throughput in your own words.

**Done when:** You understand why a system can have low latency but still have limited throughput.

---

### Wednesday — Boundaries / Resources / Failures

- Learn: Network boundary
- Learn: Process boundary
- Learn: Database boundary
- Learn: CPU / Memory / Connections
- Learn: Timeouts / Exceptions / DB failures

**Task:** Take:

`Client → Kestrel → ASP.NET Core → EF Core → PostgreSQL`

For every part, ask:

- What resource does it use?
- How can it fail?
- What can become a bottleneck?

**Done when:** You can point at the architecture and identify major failure/resource boundaries.

---

### Thursday — Final Integration

No new topics.

Do these 3 things:

**1. Draw**

`Client → Network → Kestrel → ASP.NET Core → Middleware → Endpoint → Application → EF Core → PostgreSQL → Response`

**2. Explain**

Take:

`GET /api/users/10`

and explain the entire execution flow from request to response.

**3. Write**

One page:

> **My Backend Execution Mental Model**

Include:

- Request lifecycle
- Process / Thread
- CPU vs I/O
- Latency vs Throughput
- Resources
- Failure boundaries

**Done = You can explain the whole flow without looking at notes.**

---

### Weekly Output

By Thursday you should have only **3 things**:

- One request lifecycle diagram
- One-page Execution Mental Model
- Ability to explain the whole flow verbally

**Don't build a project this week. Don't go deep into ASP.NET internals. Don't study everything you encounter.**