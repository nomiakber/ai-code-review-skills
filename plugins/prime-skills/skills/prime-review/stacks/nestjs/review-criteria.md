# Review criteria

Shared by the main agent and every slice sub-agent for Prime Review NestJS.

Also read `finding-contract.md` in this stack folder (What not to flag and Finding contract). Use the Category list below for every finding's Category field.

## Category list (section order)

Security, Correctness, NestJS, Microservices, GraphQL, Performance, Simplicity, Tests

Omit the Microservices section in `TODO_FILE` when Extra prep did not detect microservices. Omit the GraphQL section when Extra prep did not detect GraphQL.

## Review areas

Check what applies; skip what doesn't.

**Security**
- Missing or bypassable guards; routes marked public that mutate privileged state.
- Injection via unsanitized query/body/param values reaching ORM raw queries, child processes, or file paths.
- Secrets or tokens in logs, error responses, or committed config.
- Trusting microservice message identity or GraphQL context without the project's auth checks.

**Correctness**
- Validation failures ignored or mapped to success; wrong HTTP/RPC status or response contracts.
- Partial writes across DB + events/queues; swallowed exceptions in filters or interceptors.
- Racey concurrent handlers where shared state or idempotency keys are missing and loss is concrete.
- Assumptions about env, config modules, or load order that break at runtime.

**NestJS**
- Module import/export mistakes that leave providers undefined or accidentally global.
- Provider scope errors (singleton vs request) that cause cross-request data leaks or stale state.
- Pipe/interceptor/filter misuse that drops errors or transforms data incorrectly.
- Deprecated Nest APIs relative to the project's Nest major when they cause real breakage risk.

**Microservices**
- Only when microservices are detected. Skip entirely otherwise.
- Unvalidated message/event payloads reaching privileged handlers.
- Missing ack, retry, or idempotency where message loss or double-processing has a concrete cost.
- Trusting transport peer identity without authz the project requires.

**GraphQL**
- Only when GraphQL is detected. Skip entirely otherwise.
- Resolver auth gaps (query/mutation reachable without the same checks as HTTP equivalents).
- N+1 or unbounded field resolution with a concrete cost in this schema.
- Over-exposure of sensitive fields; input types accepted without validation.
- Context pollution (shared mutable context across requests) when evidence shows it.

**Performance**
- Blocking CPU or sync I/O on hot request paths with evidence.
- Unbounded `find`/`findMany` style loads; missing pagination where lists can grow.
- Redundant module re-instantiation or repeated expensive work per request.

**Simplicity**
- Delete: dead modules, unused providers, options and dependencies (search the whole repo first).
- Reuse: logic the codebase already has as a service, pipe, or guard (name the path).
- Native: reimplementing what Nest pipes, guards, or the platform already provide.
- YAGNI: abstractions with one implementation; wrappers that only pass calls through.
- Merge: near-copies that must change together.
- Split: controllers/services doing several unrelated jobs. Split by job, never by line count.
- Circular module imports; dependencies that point the wrong way between layers.

**Tests**
- Using the existing test framework, find untested logic that carries real risk: the findings above, guards, validation, and error paths.
- Tests in the audited folder that are broken, skipped or assert nothing.
