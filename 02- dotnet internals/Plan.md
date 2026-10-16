## Phase 02 — ASP.NET Core Internals & Deep API Architecture
**Duration:** 3 Weeks
**Dates:** October 5 – October 25, 2026
****
## Week 1 — Hosting, DI & Application Composition
### Saturday — Hosting & Configuration 1
* [x] Study `WebApplication.CreateBuilder(args)`.
* [x] Understand `WebApplicationBuilder` and the Generic Host.
* [x] Understand Host configuration vs Application configuration.
* [x] Study configuration providers and configuration precedence.
* [x] Understand environment-specific configuration.
* [x] Study `IConfiguration`.
**Practical:** جرّب تشوف الـ configuration فعليًا في مشروع ASP.NET Core صغير.
---
### Sunday — Hosting & Configuration 2
* [x] Study `IOptions<T>`.
* [x] Study `IOptionsSnapshot<T>`.
* [x] Study `IOptionsMonitor<T>`.
* [x] Study Options validation.
* [x] Understand service registration during application startup.
* [x] Understand what `Build()` creates.
**Practical:** اعمل Options class واربطها بالـ configuration وجرب الـ 3 options patterns.
---
### Monday — Application Lifetime
* [x] Understand application lifetime and graceful shutdown.
* [x] Study `IHostApplicationLifetime`.
* [ ] Understand graceful shutdown and cancellation on Linux.
**Practical:** شغّل التطبيق، اقفله، وراقب lifecycle/cancellation behavior.
---
### Tuesday — DI Foundations
* [x] Study `IServiceCollection`.
* [x] Study `ServiceDescriptor`.
* [x] Study `IServiceProvider`.
* [x] Understand service registration vs service resolution.
* [x] Understand dependency graphs and object construction.
* [x] Study Singleton / Scoped / Transient lifetimes.
* [x] Understand Root Service Provider vs Service Scopes.
* [ ] Understand scope creation and disposal.
**Practical:**
* [x] Inspect registered `ServiceDescriptor`s.
* [x] Reproduce Singleton / Scoped / Transient behavior.
---
### Wednesday — DI Internals
* [ ] Study `IDisposable` / `IAsyncDisposable` disposal through DI.
* [ ] Understand captive dependencies.
* [ ] Study `ValidateScopes` and `ValidateOnBuild`.
* [ ] Study factory registrations.
* [ ] Study open generic registrations.
* [ ] Study keyed services in modern .NET.
**Practical:**
* [ ] Reproduce a captive dependency.
* [ ] Observe scope disposal with `IDisposable`.
* [ ] Experiment with open generics.
* [ ] Experiment with keyed services.
---
### Thursday — DI Construction & Final Experiment
* [ ] Study `ActivatorUtilities`.
* [ ] Understand Dependency Inversion vs Service Locator.
* [ ] Understand where factories and other design patterns appear inside the DI system.
**Practical:**
* [ ] Use `ActivatorUtilities` to construct objects.
* [ ] Trace a dependency from registration → resolution → construction → disposal.
---
## Week 2 — ASP.NET Core API Execution & Extensibility

### Day 1 — Endpoint & Metadata Model
- [x] Understand the modern Endpoint model.
- [x] Study `Endpoint`.
- [x] Study `EndpointMetadata`.
- [x] Understand how attributes become runtime metadata.
- [x] Study `[ApiController]`, `[Route]`, `[HttpGet]`, `[HttpPost]`.
- [x] Study `ControllerActionDescriptor`.
- [x] Understand how controllers and actions are represented by the framework.
- [x] Understand how framework components consume endpoint metadata.
- [x] Study Endpoint Filters.
- [x] Understand when Endpoint Filters are useful.
#### Practical
- [ ] Inspect endpoint metadata at runtime.
- [ ] Create custom attributes and inspect them through reflection.

### Day 2 — Model Binding & Validation
- [x] Study Model Binding internals.
- [x] Understand Route, Query, Headers, Form, and Body binding sources.
- [x] Understand how request data becomes .NET objects.
- [x] Study `ModelState`.
- [x] Study validation metadata.
- [x] Understand automatic API validation.

#### Practical
- [x] Build an endpoint with custom model binding or validation behavior.
- [x] Test valid requests, invalid values, missing data, and binding failures.

### Day 3 — Serialization & JSON
- [x] Study `System.Text.Json` in the API pipeline.
- [x] Understand serialization vs deserialization.
- [x] Understand source-generated JSON serialization.
- [ ] Compare reflection-based serialization with source generation at a practical level.
- [x] Understand how serialization fits into request and response execution.

#### Practical
- [x] Test JSON request deserialization and response serialization.
- [x] Compare reflection-based and source-generated serialization behavior.

### Day 4 — MVC Filter Pipeline
- [ ] Study Authorization Filters.
- [ ] Study Resource Filters.
- [x] Study Action Filters.
- [ ] Study Exception Filters.
- [ ] Study Result Filters.
- [x] Understand filter execution order.
- [ ] Understand filter scopes.
- [ ] Understand dependency injection inside filters.
- [ ] Compare Middleware, Endpoint Filters, and MVC Filters.
- [ ] Identify the correct extension point for common API concerns.

#### Practical
- [x] Build a custom Action Filter.
- [ ] Build a custom Exception Filter.
- [ ] Observe filter execution order with logging or breakpoints.

### Day 5 — Controller, Action & Result Execution
- [x] Understand controller activation.
- [x] Understand constructor dependency injection into controllers.
- [x] Study action invocation.
- [x] Understand action parameters.
- [ ] Study Action Results.
- [ ] Understand result execution.
- [ ] Study formatters and content negotiation at a practical level.
- [ ] Understand object → JSON → HTTP response.

#### Practical
- [ ] Trace controller activation and action execution with breakpoints.
- [ ] Observe how an action result becomes an HTTP response.

### Day 6 — Integration & Framework Tracing
- [ ] Build a custom Endpoint Filter.
- [ ] Trace a real endpoint with breakpoints.
- [ ] Inspect relevant ASP.NET Core framework source when behavior is unclear.
- [ ] Trace endpoint selection, metadata consumption, binding, validation, filter execution, action invocation, and response generation.
- [ ] Revisit unclear concepts from Days 1–5.

#### Practical completion checklist
- [ ] Inspect endpoint metadata at runtime.
- [ ] Create custom attributes and inspect them through reflection.
- [ ] Build a custom Endpoint Filter.
- [ ] Build a custom Action Filter.
- [ ] Build an endpoint with custom model binding/validation behavior.
- [ ] Trace a real endpoint with breakpoints and framework source.

### Day 7 — Review
- [ ] Explain the Endpoint and Metadata model without notes.
- [ ] Explain how model binding and validation work.
- [ ] Explain serialization vs deserialization and source generation.
- [ ] Explain the MVC filter execution order and scopes.
- [ ] Compare Middleware, Endpoint Filters, and MVC Filters.
- [ ] Trace object → JSON → HTTP response.
- [ ] Identify remaining gaps and record what needs further study.
---
#### Week 3 — Error Architecture, Outbound HTTP & Defensive Backend Design
##### Goal
Build a deeper understanding of production API behavior around failures, external services, connection management, and resilient application boundaries.
##### Tasks
###### Error Handling
* Study centralized exception handling.
* Study `IExceptionHandler`.
* Study `ProblemDetails`.
* Study `IProblemDetailsService`.
* Understand RFC 7807.
* Design a consistent API error contract for frontend consumers.
* Understand expected business failures vs exceptional failures.
* Study the Result Pattern.
* Understand when returning a Result is preferable to throwing an exception.
* Implement a lightweight Result type and apply it to a real endpoint.
###### Idempotency & Defensive API Design
* Understand idempotency in HTTP APIs.
* Study idempotency keys.
* Understand duplicate requests and retry scenarios.
* Identify operations that require idempotency.
* Design an idempotent API operation.
* Understand fail-fast validation at API boundaries.
* Study cancellation propagation through application layers.
###### Outbound HTTP
* Study modern `HttpClient`.
* Understand why repeatedly creating/discarding `HttpClient` instances can cause problems.
* Study `HttpMessageHandler`.
* Study `SocketsHttpHandler`.
* Understand connection pooling.
* Understand DNS resolution and connection lifetime.
* Understand socket exhaustion.
* Study `IHttpClientFactory`.
* Study Named Clients.
* Study Typed Clients.
* Study Delegating Handlers.
* Understand DelegatingHandler as a practical Decorator pattern.
* Implement outbound logging/authentication through a Delegating Handler.
* Understand timeout and cancellation behavior for outbound requests.
###### Practical Experiments
* Build centralized `IExceptionHandler` + `ProblemDetails`.
* Implement Result-based business error handling.
* Build an idempotent API operation.
* Configure a Typed `HttpClient`.
* Add a Delegating Handler for logging.
* Observe connection reuse and open sockets with Linux tools.
* Simulate an external service failure and observe application behavior.
---
#### Phase Completion
* Explain modern ASP.NET Core DI and service lifetimes.
* Explain how configuration and Options are composed.
* Explain how attributes become runtime metadata.
* Explain Endpoint Metadata and Filters.
* Explain model binding, validation, and serialization.
* Explain the difference between Middleware, Endpoint Filters, and MVC Filters.
* Implement centralized ProblemDetails error handling.
* Design idempotent API operations.
* Explain `HttpClient`, handlers, connection pooling, and `IHttpClientFactory`.
* Read relevant ASP.NET Core source code when framework behavior is unclear.
* Apply the concepts to an existing real API instead of building a toy application.