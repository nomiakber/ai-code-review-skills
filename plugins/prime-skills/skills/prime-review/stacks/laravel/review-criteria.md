# Review criteria

Shared by the main agent and every slice sub-agent for Prime Review Laravel.

Also read `finding-contract.md` in this stack folder (What not to flag and Finding contract). Use the Category list below for every finding's Category field.

## Category list (section order)

Security, Correctness, Laravel integration, Round trip & contracts, Performance, Blade/views, Simplicity, Tests

## Review areas

Check what applies; skip what doesn't.

**Security**
- Authorization: every state-changing or sensitive read uses policies, gates, or middleware the project relies on (`authorize`, `can`, `Gate`, route middleware). Flag sibling routes that skip a rule another route enforces. Client-only checks are never authoritative.
- Authentication assumptions: routes that mutate or expose private data without `auth` (or the project's equivalent) when siblings require it.
- Mass assignment: `create` / `update` / `fill` / `forceFill` with request input broader than `$fillable` / `$guarded` intends; `Model::unguard()` left on.
- CSRF: web form / cookie-session state changes without the framework CSRF protection the app uses; API token misuse that bypasses the project's auth scheme.
- SQL: raw queries or `whereRaw` / `orderByRaw` / `DB::raw` with untrusted input; string-built identifiers.
- Files: user-controlled paths, unsafe storage disks, missing validation of uploads (`mimes`, size) where the project otherwise validates them.
- Redirects: open redirects from user input (`redirect($request->input('url'))` and equivalents) without an allowlist or `url()->to` safety the project uses.
- Secrets: API keys, tokens or passwords logged, returned in JSON, or printed in Blade.

**Correctness**
- Validation failures ignored or answered with the wrong status/body; `$request->validate` / Form Request rules that don't match what the handler reads.
- Unchecked results that leave partial state (DB write succeeded, job/notification skipped, or the reverse).
- Timezone / serialization bugs where the project stores or compares datetimes inconsistently (`Carbon`, casts, `datetime` columns).
- Racey check-then-act on status transitions without locking, unique constraints, or queue exclusivity the runner makes reachable.
- Wrong HTTP contracts: success on failure, error payloads clients can't distinguish, pagination or envelope shape that breaks documented consumers.

**Laravel integration**
- Form Requests (or a project-standard validator) unused where the codebase already uses them for the same kind of input.
- Facades, helpers, or service-container usage that fights the project's architecture (for example business logic stuffed in routes/controllers when services already exist for that job).
- APIs newer than the minimum PHP or Laravel version from `composer.json`.
- Eloquent vs query builder misuse that causes a real bug (wrong soft-delete behavior, global scopes skipped unintentionally, `withTrashed` exposing deleted rows).
- Deprecated framework APIs for the project's Laravel version.

**Round trip & contracts**
- For every write path (HTTP, console, job), trace hostile values through validation → casts → DB → read-back → API or Blade: `\`, `&`, `<`, quotes, overlong strings, wrong types (array where string expected).
- Fields returned by API resources/transformers: name writers and consumers; flag hard-coded empties, read-never-written, write-never-read, and shape changes a client would drop.
- Invariants: when one entry point enforces a rule (ownership, status lock, tenancy), check sibling write routes and model methods for the same rule.
- Early returns: list what is already written, queued, mailed or logged at each exit; flag partial success and audit entries only some paths reach.

**Performance**
- N+1 relation loads in loops; missing `with` / `loadMissing` where the cost is concrete on this endpoint or job.
- Unbounded `get()` / `all()` on tables that grow; exports or admin lists without chunking, cursor, or pagination.
- Expensive work in middleware or service providers that every request pays for when only one route needs it.
- Cache that is never invalidated, or remote calls on hot paths without caching where failure/cost is concrete.

**Blade/views**
- Unescaped output (`{!! !!}`) with data that can be user- or stored-content-controlled without an explicit sanitizer.
- XSS via components or `@json` misuse in HTML attributes/scripts.
- Business logic, queries, or authorization decisions that belong in the action/service layer already used elsewhere.
- Views that leak sensitive attributes a policy would hide on the API path.

**Simplicity**
- Delete: dead routes, controllers, jobs, unused imports and dependencies. Search the whole repo (including tests, route lists and string references) before claiming anything is unused.
- Reuse: existing Form Requests, policies, actions, queries or helpers (name the path).
- Native: code or a dependency doing what Laravel already provides.
- YAGNI: abstractions with one implementation, wrappers that only pass calls through, config nobody sets.
- Merge: near-copies that must change together.
- Split: one class or method doing several unrelated jobs. Split by job, never by line count.
- Hidden dependencies and circular service references that make the code hard to test.

**Tests**
- Using the project's PHPUnit or Pest setup, find untested logic that carries real risk: the findings above, authz, validation, mass assignment, job failure paths. Say what to fake at the framework boundary (HTTP, queue, mail, storage).
- Tests in the audited folder that are broken, skipped or assert nothing.
