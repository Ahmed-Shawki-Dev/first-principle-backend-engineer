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
* [ ] Study `IOptions<T>`.
* [ ] Study `IOptionsSnapshot<T>`.
* [ ] Study `IOptionsMonitor<T>`.
* [ ] Study Options validation.
* [ ] Understand service registration during application startup.
* [ ] Understand what `Build()` creates.
**Practical:** اعمل Options class واربطها بالـ configuration وجرب الـ 3 options patterns.
---
### Monday — Application Lifetime
* [ ] Understand application lifetime and graceful shutdown.
* [ ] Study `IHostApplicationLifetime`.
* [ ] Understand graceful shutdown and cancellation on Linux.
**Practical:** شغّل التطبيق، اقفله، وراقب lifecycle/cancellation behavior.
---
### Tuesday — DI Foundations
* [ ] Study `IServiceCollection`.
* [ ] Study `ServiceDescriptor`.
* [ ] Study `IServiceProvider`.
* [ ] Understand service registration vs service resolution.
* [ ] Understand dependency graphs and object construction.
* [ ] Study Singleton / Scoped / Transient lifetimes.
* [ ] Understand Root Service Provider vs Service Scopes.
* [ ] Understand scope creation and disposal.
**Practical:**
* [ ] Inspect registered `ServiceDescriptor`s.
* [ ] Reproduce Singleton / Scoped / Transient behavior.
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
### Week 2 — ASP.NET Core API Execution & Extensibility
#### Goal
Understand the framework machinery behind the API features you already use, especially attributes, metadata, filters, binding, validation, and endpoint execution.
#### Tasks
##### Endpoint & Metadata Model
* Understand the modern Endpoint model.
* Study `Endpoint`.
* Study `EndpointMetadata`.
* Understand how attributes become runtime metadata.
* Study:
    * `[ApiController]`
    * `[Route]`
    * `[HttpGet]`
    * `[HttpPost]`
* Study `ControllerActionDescriptor`.
* Understand how controllers and actions are represented by the framework.
* Understand how framework components consume endpoint metadata.
* Study Endpoint Filters.
* Understand when Endpoint Filters are useful.
##### Model Binding & Serialization
* Study Model Binding internals.
* Understand binding sources:
    * Route
    * Query
    * Headers
    * Form
    * Body
* Understand how request data becomes .NET objects.
* Study `ModelState`.
* Study validation metadata.
* Understand automatic API validation.
* Study `System.Text.Json` in the API pipeline.
* Understand serialization vs deserialization.
* Understand source-generated JSON serialization.
* Compare reflection-based serialization with source generation at a practical level.
##### Filters
* Study the MVC filter pipeline.
* Study:
    * Authorization Filters
    * Resource Filters
    * Action Filters
    * Exception Filters
    * Result Filters
* Understand filter execution order.
* Understand filter scopes.
* Understand dependency injection inside filters.
* Compare:
    * Middleware
    * Endpoint Filters
    * MVC Filters
* Build custom Action and Exception Filters.
* Identify the correct extension point for common API concerns.
##### Action & Result Execution
* Understand controller activation.
* Understand constructor dependency injection into controllers.
* Study action invocation.
* Understand action parameters.
* Study Action Results.
* Understand result execution.
* Study formatters and content negotiation at a practical level.
* Understand object → JSON → HTTP response.
##### Practical Experiments
* Inspect endpoint metadata at runtime.
* Create custom attributes and inspect them through reflection.
* Build a custom Endpoint Filter.
* Build a custom Action Filter.
* Build an endpoint with custom model binding/validation behavior.
* Trace a real endpoint with breakpoints and framework source.
---
### Week 3 — Error Architecture, Outbound HTTP & Defensive Backend Design
#### Goal
Build a deeper understanding of production API behavior around failures, external services, connection management, and resilient application boundaries.
#### Tasks
##### Error Handling
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
##### Idempotency & Defensive API Design
* Understand idempotency in HTTP APIs.
* Study idempotency keys.
* Understand duplicate requests and retry scenarios.
* Identify operations that require idempotency.
* Design an idempotent API operation.
* Understand fail-fast validation at API boundaries.
* Study cancellation propagation through application layers.
##### Outbound HTTP
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
##### Practical Experiments
* Build centralized `IExceptionHandler` + `ProblemDetails`.
* Implement Result-based business error handling.
* Build an idempotent API operation.
* Configure a Typed `HttpClient`.
* Add a Delegating Handler for logging.
* Observe connection reuse and open sockets with Linux tools.
* Simulate an external service failure and observe application behavior.
---
### Phase Completion
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