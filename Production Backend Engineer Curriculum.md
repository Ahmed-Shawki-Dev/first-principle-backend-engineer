A Pragmatic, Time-Boxed Roadmap
.NET / PostgreSQL / Distributed Systems / DevOps Knowledge
## Curriculum Rules
1. The curriculum defines WHAT to learn, not WHERE to learn it.
   Sources are interchangeable: official documentation, books, videos, articles, AI, experiments, and real projects.

2. Every phase has a fixed timebox.
   The timebox exists to prevent wandering, not to force artificial speed.

3. This is NOT a beginner curriculum.
   The following are assumed knowledge or existing experience:
   - Programming fundamentals
   - OOP
   - C#
   - Basic ASP.NET Core
   - Controllers / Services
   - REST APIs
   - HTTP fundamentals
   - SQL fundamentals
   - EF Core fundamentals
   - Git
   - Linux fundamentals
   - Computer Networks fundamentals
   - Operating Systems fundamentals
   - Basic cybersecurity
   - Ethical hacking fundamentals
   - Docker fundamentals

4. Do not spend curriculum time relearning university-level foundations.

5. Do not build toy projects just to satisfy the curriculum.
   Use existing projects whenever practical.

6. Project-driven learning and curriculum-driven learning run in parallel.
   If a project requires a topic before its scheduled phase, learn enough to unblock the project.
   When that topic appears in the curriculum, study it systematically.

7. The target is:

   Strong Junior → Mid-level Backend Engineer

8. DevOps is included as engineering knowledge, not as a DevOps career track.

9. Security is included as backend/application security, not as a second Cybersecurity course.

10. The objective is not to become an expert in every topic.
    The objective is to understand the important mechanisms, trade-offs, and practical usage well enough to design, implement, debug, and discuss backend systems confidently.

---
# Timeline Overview

| Dates                      | Phase                                         | Duration |
| -------------------------- | --------------------------------------------- | -------: |
| Sep 28 – Oct 4, 2026       | Phase 01 — Execution Mental Model             |   1 week |
| Oct 5 – Oct 25, 2026       | Phase 02 — ASP.NET Core Internals             |  3 weeks |
| Oct 26 – Nov 22, 2026      | Phase 03 — EF Core & Database Engineering     |  4 weeks |
| Nov 23 – Dec 13, 2026      | Phase 04 — Backend Architecture               |  3 weeks |
| Dec 14 – Dec 20, 2026      | Phase 05 — Backend Security                   |   1 week |
| Dec 21, 2026 – Jan 3, 2027 | Phase 06 — Async & Concurrency                |  2 weeks |
| Jan 4 – Jan 24, 2027       | Phase 07 — Testing Engineering                |  3 weeks |
| Jan 25 – Feb 7, 2027       | Phase 08 — Background Processing & Resilience |  2 weeks |
| Feb 8 – Feb 14, 2027       | Phase 09 — Caching & Redis                    |   1 week |
| Feb 15 – Feb 21, 2027      | Phase 10 — Observability                      |   1 week |
| Feb 22 – Feb 28, 2027      | Phase 11 — Docker & DevOps Foundations        |   1 week |
| Mar 1 – Mar 14, 2027       | Phase 12 — Infrastructure & Deployment        |  2 weeks |
| Mar 15 – Apr 11, 2027      | Phase 13 — Distributed Systems                |  4 weeks |
| Apr 12 – May 2, 2027       | Phase 14 — System Design                      |  3 weeks |
| May 3 – May 16, 2027       | Phase 15 — Networking & Real-Time             |  2 weeks |
| May 17 – May 30, 2027      | Phase 16 — Microservices                      |  2 weeks |
| May 31 – Jun 13, 2027      | Phase 17 — .NET Runtime & Performance         |  2 weeks |

**Total: 37 weeks**

---
# Phase 01 — Execution Mental Model

**Dates:** Sep 28 – Oct 4, 2026  
**Duration:** 1 week
## Goal
Build one coherent mental model of how a backend system executes:

Network → OS → Kestrel → ASP.NET Core → Middleware → Endpoint → Application → EF Core → PostgreSQL → Response
## Topics
- Request/response lifecycle
- Process vs thread
- CPU-bound vs I/O-bound work
- Latency vs throughput
- Statelessness
- Network boundaries
- Failure boundaries
- Resource limits
- Basic scalability mental model

## Milestone
- [ ] Draw the complete request lifecycle
- [ ] Explain CPU-bound vs I/O-bound work
- [ ] Explain latency vs throughput
- [ ] Identify major resource and failure boundaries
- [ ] Write a one-page Execution Mental Model

---****
# Phase 02 — ASP.NET Core Internals

**Dates:** Oct 5 – Oct 25, 2026  
**Duration:** 3 weeks

## Goal

Understand what ASP.NET Core actually does between receiving an HTTP request and producing a response.

## Topics

### Hosting

- Kestrel
- Generic Host
- Application startup
- Server vs application
- Reverse proxy relationship

### Request Pipeline

- Middleware pipeline
- Middleware ordering
- Short-circuiting
- HttpContext
- Request lifetime
- Cancellation

### Dependency Injection

- Service provider
- Singleton / Scoped / Transient
- Scope creation
- Captive dependencies
- Disposal

### Routing & Endpoints

- Routing
- Endpoint selection
- Endpoint metadata
- Endpoint execution

### MVC/API Pipeline

- Model binding
- Validation
- Filters
- Action execution
- Result execution

### Error Handling

- Exception handling
- Problem Details
- Centralized API error handling

### Outbound HTTP

- HttpClient lifecycle
- IHttpClientFactory
- Connection pooling
- Socket exhaustion
- Named / typed clients

## Milestone

- [ ] Trace a request through the complete ASP.NET Core pipeline
- [ ] Explain DI lifetimes and scope creation
- [ ] Explain middleware ordering
- [ ] Explain routing vs endpoint execution
- [ ] Explain model binding and validation
- [ ] Implement centralized Problem Details handling
- [ ] Explain IHttpClientFactory and connection management

---

# Phase 03 — EF Core & Database Engineering

**Dates:** Oct 26 – Nov 22, 2026  
**Duration:** 4 weeks

## Goal

Understand the complete chain:

C# → EF Core → SQL → PostgreSQL Planner → Indexes → Locks → Storage

## EF Core

- DbContext lifecycle
- Change Tracker
- Entity states
- Tracking vs NoTracking
- Projection
- Loading strategies
- N+1
- LINQ translation
- Query compilation
- Global Query Filters
- Value Converters
- Interceptors
- EF Core performance

## PostgreSQL

- Constraints
- B-Tree indexes
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

---

# Phase 04 — Backend Architecture

**Dates:** Nov 23 – Dec 13, 2026  
**Duration:** 3 weeks

## Goal

Understand architectural boundaries and make deliberate trade-offs instead of following architecture templates blindly.

## Topics

### Foundations

- Coupling
- Cohesion
- Dependency direction
- Boundaries
- Separation of concerns

### Architectures

- Layered Architecture
- Clean Architecture
- Vertical Slice
- Modular Monolith
- Domain Model

### Application Design

- Domain Services
- Application Services
- Commands / Queries
- CQRS
- Mediator
- Dependency Inversion
- Repository trade-offs

### Architecture Decisions

- When abstraction helps
- When abstraction becomes ceremony
- When Clean Architecture is useful
- When Vertical Slice is useful
- When a simpler Layered Architecture is better
- Modular boundaries

## Milestone

- [ ] Explain why dependencies should point in specific directions
- [ ] Compare Layered, Clean, Vertical Slice, and Modular Monolith
- [ ] Identify architectural boundaries in an existing project
- [ ] Refactor one meaningful module
- [ ] Make an explicit architecture decision and document its trade-offs

---

# Phase 05 — Backend Security

**Dates:** Dec 14 – Dec 20, 2026  
**Duration:** 1 week

## Assumption

Cybersecurity and Ethical Hacking fundamentals are already covered.

This phase is only about integrating security knowledge into backend engineering.

## Topics

- Authentication vs Authorization
- Claims
- Policies
- Authorization handlers
- JWT lifecycle
- Access vs refresh tokens
- Refresh token rotation
- Token theft/reuse detection
- Cookie authentication
- HttpOnly
- Secure
- SameSite
- CSRF
- CORS
- Password hashing
- Secrets management
- ASP.NET Core Data Protection
- Rate limiting
- OWASP API Security fundamentals

## Milestone

- [ ] Explain the complete authentication lifecycle
- [ ] Implement policy-based authorization
- [ ] Explain refresh-token rotation and reuse detection
- [ ] Explain Cookie + HttpOnly + Secure + SameSite + CSRF together
- [ ] Apply rate limiting to a real endpoint
- [ ] Perform a security review of one existing API

---

# Phase 06 — Async & Concurrency

**Dates:** Dec 21, 2026 – Jan 3, 2027  
**Duration:** 2 weeks

## Goal

Understand how .NET handles asynchronous and concurrent work.

## Topics

- async/await mechanics
- State machines
- Task
- I/O-bound vs CPU-bound
- Thread Pool
- Thread starvation
- Blocking vs asynchronous execution
- Task.WhenAll
- Cancellation
- Race conditions
- Synchronization
- SemaphoreSlim
- Concurrent collections
- Deadlocks
- Optimistic vs pessimistic concurrency

## Milestone

- [ ] Explain what happens at an await
- [ ] Explain why blocking .Result/.Wait() can cause starvation
- [ ] Use Task.WhenAll correctly for independent I/O
- [ ] Reproduce a race condition
- [ ] Fix a concurrency problem using appropriate synchronization
- [ ] Use cancellation correctly in background/request work

---

# Phase 07 — Testing Engineering

**Dates:** Jan 4 – Jan 24, 2027  
**Duration:** 3 weeks

## Goal

Become comfortable testing backend systems without turning testing into a separate career track.

## Topics

### Testing Model

- Testing pyramid
- Unit vs integration vs E2E
- What should and should not be mocked
- Test boundaries

### Unit Testing

- xUnit
- FluentAssertions
- Business-rule testing
- Dependency isolation
- Mocking with Moq or NSubstitute

### Integration Testing

- WebApplicationFactory
- Real HTTP requests
- Real application pipeline
- Database integration

### Testcontainers

- PostgreSQL container
- Real migrations
- Real database behavior
- Test isolation

### Architecture Testing

- Dependency rules
- Architecture tests
- NetArchTest or equivalent

## Milestone

- [ ] Write meaningful unit tests for business rules
- [ ] Write integration tests against the API
- [ ] Run PostgreSQL through Testcontainers
- [ ] Test migrations against a real database
- [ ] Add architecture tests for dependency boundaries
- [ ] Decide when a dependency should be mocked and when it should be real

---

# Phase 08 — Background Processing & Resilience

**Dates:** Jan 25 – Feb 7, 2027  
**Duration:** 2 weeks

## Goal

Understand work that should happen outside the HTTP request and how systems behave when dependencies fail.

## Background Processing

- HostedService
- BackgroundService
- Worker Services
- Producer/Consumer
- System.Threading.Channels
- Queues
- Scheduled jobs
- Hangfire
- Quartz basics

## Resilience

- Timeouts
- Retry
- Exponential backoff
- Jitter
- Circuit breaker
- Graceful degradation
- Graceful shutdown
- Cancellation
- Idempotency basics

## Milestone

- [ ] Move one suitable operation outside the HTTP request
- [ ] Build a small producer/consumer flow
- [ ] Implement a scheduled background job
- [ ] Understand when retry is safe and when it is dangerous
- [ ] Add timeout/retry/circuit-breaker behavior to an external dependency
- [ ] Handle graceful shutdown correctly

---

# Phase 09 — Caching & Redis

**Dates:** Feb 8 – Feb 14, 2027  
**Duration:** 1 week

## Goal

Understand caching as a consistency/performance trade-off.

## Topics

- Why caching exists
- Cache-aside
- Read-through / write-through concepts
- TTL
- Absolute vs sliding expiration
- Cache invalidation
- Cache stampede
- Cache consistency
- In-memory caching
- Distributed caching
- Redis
- Relevant Redis data structures
- Pub/Sub basics

## Milestone

- [ ] Identify appropriate cache candidates in a real system
- [ ] Implement cache-aside with Redis
- [ ] Define expiration and invalidation strategy
- [ ] Explain what happens when Redis is unavailable
- [ ] Explain why caching can make a system incorrect

---

# Phase 10 — Observability

**Dates:** Feb 15 – Feb 21, 2027  
**Duration:** 1 week

## Goal

Know what the system is doing without relying on guesswork.

## Topics

- Structured logging
- Serilog
- Log levels
- Correlation IDs
- Health checks
- Liveness vs readiness
- Metrics
- OpenTelemetry
- Distributed tracing basics
- Error tracking
- Production diagnostics

## Milestone

- [ ] Add structured logging to a real API
- [ ] Trace one request using a correlation/trace ID
- [ ] Add liveness and readiness checks
- [ ] Expose basic application metrics
- [ ] Trace a request across API → database → external dependency

---

# Phase 11 — Docker & DevOps Foundations

**Dates:** Feb 22 – Feb 28, 2027  
**Duration:** 1 week

## Goal

Understand the DevOps concepts a backend engineer needs to work effectively with production infrastructure.

## Docker

- Images
- Containers
- Layers
- Dockerfile
- Multi-stage builds
- Volumes
- Networks
- Environment variables
- Docker Compose
- Container lifecycle

## DevOps Knowledge

- CI vs CD
- Build pipelines
- Automated tests
- Artifacts
- Container image lifecycle
- Environment promotion
- Configuration management
- Secrets
- Basic deployment flow
- GitHub Actions

## Milestone

- [ ] Containerize an ASP.NET Core API
- [ ] Write a multi-stage Dockerfile
- [ ] Run API + PostgreSQL + Redis with Compose
- [ ] Understand container networking
- [ ] Create a basic GitHub Actions pipeline
- [ ] Explain CI/CD from commit to deployed application

---

# Phase 12 — Infrastructure & Deployment

**Dates:** Mar 1 – Mar 14, 2027  
**Duration:** 2 weeks

## Goal

Understand how a backend actually reaches production.

## Assumed Knowledge

Linux and networking fundamentals are already known.

No dedicated Linux fundamentals phase is required.

## Topics

### Production Linux

- Processes
- Signals
- Services
- Ports
- Environment variables
- Permissions
- Process inspection
- Basic server troubleshooting

### Reverse Proxy & Deployment

- Nginx
- Kestrel behind Nginx
- HTTPS / TLS in deployment
- DNS basics in deployment context
- Docker deployment
- PostgreSQL deployment considerations
- Redis deployment considerations
- Production configuration
- Secrets
- Logs
- Health checks

### CI/CD

- Build
- Test
- Package
- Deploy
- Rollback concept

## Milestone

- [ ] Deploy a containerized API
- [ ] Put Nginx in front of Kestrel
- [ ] Configure HTTPS
- [ ] Configure production environment variables/secrets
- [ ] Understand the complete production request path
- [ ] Understand deployment and rollback flow
- [ ] Diagnose basic production/container failures

---

# Phase 13 — Distributed Systems

**Dates:** Mar 15 – Apr 11, 2027  
**Duration:** 4 weeks

## Goal

Understand what changes when one process becomes multiple independent processes communicating over a network.

## Distributed Systems Mental Model

- Network failures
- Partial failure
- Timeouts
- Retries
- Ordering
- Duplicate messages
- Eventual consistency
- CAP
- Clock issues

## Messaging

- Message queues
- RabbitMQ
- Event streams
- Kafka concepts
- Pub/Sub
- Consumer groups
- Delivery semantics

## Reliability Patterns

- Idempotent consumers
- Outbox Pattern
- Inbox Pattern
- Dual-write problem
- At-least-once delivery
- Distributed transactions
- Saga
- Compensating transactions
- Failure isolation

## Milestone

- [ ] Explain why distributed systems fail differently from monoliths
- [ ] Build a basic producer/consumer flow
- [ ] Implement idempotent message processing
- [ ] Understand and implement a basic Outbox Pattern
- [ ] Explain Saga and compensating transactions
- [ ] Analyze failure scenarios in a distributed workflow

---

# Phase 14 — Pragmatic System Design

**Dates:** Apr 12 – May 2, 2027  
**Duration:** 3 weeks

## Goal

Learn to design systems from requirements and constraints instead of starting with technologies.

## Requirements

- Functional requirements
- Non-functional requirements
- Constraints
- Availability
- Reliability
- Consistency

## Capacity

- Requests per second
- Concurrent users
- Storage estimation
- Bandwidth estimation
- Growth estimation

## Architecture

- Load balancing
- Caching
- Database scaling
- Read replicas
- Partitioning
- Sharding concepts
- Queues
- Async processing
- Failure handling
- Disaster recovery basics

## Practice

Design:

- URL shortener
- Notification system
- File upload system
- Rate limiter
- Booking system
- Real-time system

## Milestone

- [ ] Analyze requirements before choosing technology
- [ ] Perform basic capacity estimation
- [ ] Design a complete backend architecture from scratch
- [ ] Identify bottlenecks
- [ ] Identify failure points
- [ ] Explain major trade-offs

---

# Phase 15 — Networking & Real-Time

**Dates:** May 3 – May 16, 2027  
**Duration:** 2 weeks

## Goal

Go beyond networking fundamentals and understand the protocols/backend mechanisms used by modern systems.

## Topics

- HTTP/1.1 behavior
- HTTP/2
- HTTP/3
- TLS fundamentals
- WebSockets
- SignalR
- Connection lifecycle
- Connection management
- Scaling real-time connections
- gRPC
- REST vs gRPC
- Webhooks
- Streaming
- File uploads/downloads
- Object storage
- Presigned URLs

## Milestone

- [ ] Explain HTTP/1.1 vs HTTP/2 vs HTTP/3
- [ ] Explain WebSocket connection lifecycle
- [ ] Build a basic SignalR feature
- [ ] Explain how real-time systems scale
- [ ] Compare REST and gRPC for concrete use cases
- [ ] Design a reliable file-upload flow

---

# Phase 16 — Microservices

**Dates:** May 17 – May 30, 2027  
**Duration:** 2 weeks

## Goal

Understand microservices as a set of trade-offs, not as a default architecture.

## Topics

- Service boundaries
- Service ownership
- Database per service
- Service communication
- Synchronous vs asynchronous communication
- API Gateway
- Message broker
- Service discovery concepts
- Distributed configuration
- Distributed observability
- Failure isolation
- Data consistency
- Deployment independence
- Monolith vs Modular Monolith vs Microservices

## Milestone

- [ ] Identify reasonable service boundaries
- [ ] Explain why database-per-service exists
- [ ] Design synchronous and asynchronous service communication
- [ ] Understand API Gateway responsibilities
- [ ] Explain the operational cost of microservices
- [ ] Convert one hypothetical modular-monolith boundary into services

---

# Phase 17 — .NET Runtime & Performance

**Dates:** May 31 – Jun 13, 2027  
**Duration:** 2 weeks

## Goal

Understand the runtime enough to reason about memory, allocations, threads, and performance.

## CLR

- CLR responsibilities
- Assemblies
- JIT
- Runtime execution

## Memory

- Stack vs Heap
- Managed memory
- Allocations
- Garbage Collection
- Gen 0 / Gen 1 / Gen 2
- LOH
- GC pressure

## Performance Primitives

- Span<T>
- Memory<T>
- ArrayPool<T>
- Allocation reduction
- Boxing
- String allocations

## Diagnostics

- Runtime counters
- Profiling
- BenchmarkDotNet
- Basic runtime diagnostics

## Milestone

- [ ] Explain managed memory and GC generations
- [ ] Identify unnecessary allocations
- [ ] Explain when Span<T>/Memory<T> are useful
- [ ] Benchmark two implementations
- [ ] Interpret basic runtime performance measurements
- [ ] Explain the difference between optimizing code and measuring code

---

# Master Execution Checklist

## Foundation

- [ ] Phase 01 — Execution Mental Model
- [ ] Phase 02 — ASP.NET Core Internals
- [ ] Phase 03 — EF Core & Database Engineering
- [ ] Phase 04 — Backend Architecture
- [ ] Phase 05 — Backend Security
- [ ] Phase 06 — Async & Concurrency

## Production Backend

- [ ] Phase 07 — Testing Engineering
- [ ] Phase 08 — Background Processing & Resilience
- [ ] Phase 09 — Caching & Redis
- [ ] Phase 10 — Observability
- [ ] Phase 11 — Docker & DevOps Foundations
- [ ] Phase 12 — Infrastructure & Deployment

## Advanced Backend

- [ ] Phase 13 — Distributed Systems
- [ ] Phase 14 — System Design
- [ ] Phase 15 — Networking & Real-Time
- [ ] Phase 16 — Microservices
- [ ] Phase 17 — .NET Runtime & Performance

---

# Explicitly Outside the Curriculum

These are prerequisites, existing background, or supporting knowledge.

They do NOT receive dedicated weeks:

- Basic C#
- Basic OOP
- Basic SQL
- Basic REST
- Basic HTTP
- Basic Linux
- Basic TCP/IP
- Basic Networking
- Basic Operating Systems
- Basic Cybersecurity
- Ethical Hacking fundamentals
- Basic Git
- Basic Docker usage
- Basic ASP.NET Core CRUD
- Basic EF Core CRUD
- Basic JWT implementation

They can be revisited whenever a later topic requires deeper understanding.

---

# Definition of Completion

A phase is complete when:

- [ ] The mental model is understood.
- [ ] The important mechanisms can be explained without relying on memorized terminology.
- [ ] The relevant technology has been used or experimentally reproduced.
- [ ] The concept has been applied to a real project where appropriate.
- [ ] The phase milestones are complete.
- [ ] A concise personal summary has been written.

Do not extend a phase simply because more material exists.

Do not rush a phase simply because the checklist is short.

The curriculum is a map of deliberate learning, not a list of everything that exists in backend engineering.

The objective is to finish the year with a structured mental model connecting:

Computer Systems
→ Networking
→ Runtime
→ ASP.NET Core
→ Application Architecture
→ EF Core
→ PostgreSQL
→ Concurrency
→ Background Processing
→ Caching
→ Observability
→ Docker
→ DevOps
→ Deployment
→ Distributed Systems
→ System Design
→ Microservices

and to be able to reason about these layers as one backend engineering system.
