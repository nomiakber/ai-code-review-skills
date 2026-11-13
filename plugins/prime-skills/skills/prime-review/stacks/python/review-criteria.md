# Review criteria

Shared by the main agent and every slice sub-agent for Prime Review Python.

Also read `finding-contract.md` in this stack folder (What not to flag and Finding contract). Use the Category list below for every finding's Category field.

## Category list (section order)

Security, Correctness, Python, FastAPI, Performance, Simplicity, Tests

Omit the FastAPI section in `TODO_FILE` when Extra prep did not detect FastAPI.

## Review areas

Check what applies; skip what doesn't.

**Security**
- Injection: SQL, command, path, or template built from unsanitized input.
- Unsafe `pickle`, `yaml.load`, `eval`/`exec`, or deserialization of untrusted data.
- Secret leakage in logs, exceptions, or committed config; hardcoded credentials.
- Trusting client-supplied auth or role fields without server-side checks.

**Correctness**
- Swallowed exceptions; errors mapped to success; wrong return contracts.
- TOCTOU and partial writes across files/DB/network.
- Mutable default arguments that cause cross-call state bugs.
- Assumptions about cwd, env, or import order that break at runtime.

**Python**
- Resource leaks: files, sockets, DB connections, locks without reliable cleanup.
- Threading/async misuse: race on shared mutable state; blocking I/O on the event loop with concrete cost.
- Incorrect type or protocol usage that fails for real inputs the code accepts.
- Packaging/import errors that break runtime for in-scope entrypoints.

**FastAPI**
- Only when FastAPI is detected. Skip entirely otherwise.
- Missing `Depends` security / auth on mutating or privileged routes.
- `response_model` leakage of sensitive fields; returning ORM objects unsafely.
- Unvalidated body/query/path values used in privileged operations.
- CORS `*` (or overly broad origins) combined with credentials where the project intends restricted access.
- Background tasks that can fail silently with a concrete user-visible or data-loss cost.

**Performance**
- Unbounded loads or loops with a concrete cost in this code.
- Sync blocking work on async routes where latency impact is evident.
- Repeated expensive work that should be cached or moved off the request path (only with evidence).

**Simplicity**
- Delete: dead modules, unused exports, options and dependencies (search the whole repo first).
- Reuse: logic the codebase already has as a helper or service (name the path).
- Native: reimplementing what the stdlib or declared framework already provides.
- YAGNI: abstractions with one implementation; wrappers that only pass calls through.
- Merge: near-copies that must change together.
- Split: modules doing several unrelated jobs. Split by job, never by line count.
- Circular imports; dependencies that point the wrong way between layers.

**Tests**
- Using the existing test framework (pytest or project equivalent), find untested logic that carries real risk: the findings above, validation, and error paths.
- Tests in the audited folder that are broken, skipped or assert nothing.
