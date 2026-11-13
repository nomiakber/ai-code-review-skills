# Review criteria

Shared by the main agent and every slice sub-agent for Prime Review .NET.

Also read `finding-contract.md` in this stack folder (What not to flag and Finding contract). Use the Category list below for every finding's Category field.

## Category list (section order)

Security, Correctness, .NET, ASP.NET Core, Performance, Simplicity, Tests

Omit the ASP.NET Core section in `TODO_FILE` when Extra prep did not detect ASP.NET Core.

## Review areas

Check what applies; skip what doesn't.

**Security**
- Missing authz on privileged operations; trusting client-supplied user/role ids.
- Open redirects; path traversal; command injection via unsanitized input.
- Insecure deserialization; secret or PII logging; credentials in source or config committed to the repo.
- XSS in `.cshtml` / `.razor` when those files are in scope and output is unencoded.

**Correctness**
- Null-forgiving operator or nullable annotation lies that hide real nulls and cause NREs.
- Sync-over-async (`.Result` / `.Wait()`) on request or UI threads with concrete deadlock or thread-pool risk.
- Incorrect exception handling: swallowed faults, wrong status mapping, partial writes.
- Assumptions about config, environment, or startup order that break at runtime.

**.NET**
- `IDisposable` / `IAsyncDisposable` gaps for types that own unmanaged or scarce resources.
- DI lifetime bugs: captive dependencies, disposing objects the container owns, resolving scoped services from singletons incorrectly.
- API surface mistakes: public mutability or over-broad visibility that enables misuse with concrete harm.
- Incorrect async APIs (blocking, fire-and-forget without observation) where failure is lost.

**ASP.NET Core**
- Only when ASP.NET Core is detected. Skip entirely otherwise.
- Endpoint auth gaps (missing `[Authorize]`, incorrect policies, anonymous mutating endpoints).
- Middleware order bugs (exception handling, auth, CORS) with a concrete bypass or failure mode.
- Model validation bypass; binding over-posting into sensitive fields.
- CSRF gaps on cookie-auth form/API endpoints the project intends to protect.
- Over-sharing DTOs (returning entities with secrets or internal fields).

**Performance**
- Unbounded queries or allocations with a concrete cost in this code.
- Sync I/O on hot ASP.NET request paths when async APIs exist and latency impact is evident.
- Repeated expensive work per request that should be cached or moved (only with evidence).

**Simplicity**
- Delete: dead types, unused packages, options and facades (search the whole repo first).
- Reuse: logic the codebase already has as a service or helper (name the path).
- Native: reimplementing what BCL or ASP.NET already provide.
- YAGNI: abstractions with one implementation; wrappers that only pass calls through.
- Merge: near-copies that must change together.
- Split: types doing several unrelated jobs. Split by job, never by line count.
- Circular project references; dependencies that point the wrong way between layers.

**Tests**
- Using the existing test projects, find untested logic that carries real risk: the findings above, authz, validation, and error paths.
- Tests in the audited folder that are broken, skipped or assert nothing.
