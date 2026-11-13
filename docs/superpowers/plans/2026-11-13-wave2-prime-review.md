# Wave 2 Prime Review (Vue, NestJS, Python, .NET) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add `prime-review-vue`, `prime-review-nestjs`, `prime-review-python`, and `prime-review-dotnet` skills that reuse the existing shared audit procedure and finding contract, then update README and marketplace copy.

**Architecture:** Same hybrid composition as React/Angular/Laravel: each skill is `SKILL.md` (profile) + `review-criteria.md` (categories and review areas), pointing at `../../shared/audit-procedure.md` and `../../shared/finding-contract.md`. Optional layers (Nuxt, Nest microservices/GraphQL, FastAPI, ASP.NET Core) use Extra prep detection with skippable Map/lens/category. Shared procedure and finding-contract stay unchanged.

**Tech Stack:** Markdown Cursor/Claude skills under `plugins/prime-skills/`; no application runtime code.

## Global Constraints

- Spec: `docs/superpowers/specs/2026-11-13-wave2-prime-review-design.md`
- Do **not** commit or push; the human owns all git operations
- Do **not** modify `prime-review-php`, `prime-review-js`, `prime-review-react`, `prime-review-angular`, or `prime-review-laravel` skill bodies unless a Wave-2 doc cross-link requires a one-line note outside those folders
- Do **not** edit `shared/audit-procedure.md` or `shared/finding-contract.md` in this plan
- Invocation: `disable-model-invocation: true`, `argument-hint: "<folder> <todo-file>"`
- Analysis-only audits; only `TODO_FILE` may change when a skill runs
- New skills must not assume WordPress APIs
- Delivery order: Vue → NestJS → Python → .NET → docs → verification

## File structure

| Path | Responsibility |
|------|----------------|
| `plugins/prime-skills/skills/prime-review-vue/SKILL.md` | Vue/Nuxt profile |
| `plugins/prime-skills/skills/prime-review-vue/review-criteria.md` | Vue/Nuxt review areas + category order |
| `plugins/prime-skills/skills/prime-review-nestjs/SKILL.md` | NestJS (+ optional microservices/GraphQL) profile |
| `plugins/prime-skills/skills/prime-review-nestjs/review-criteria.md` | Nest review areas + category order |
| `plugins/prime-skills/skills/prime-review-python/SKILL.md` | Python/FastAPI profile |
| `plugins/prime-skills/skills/prime-review-python/review-criteria.md` | Python/FastAPI review areas + category order |
| `plugins/prime-skills/skills/prime-review-dotnet/SKILL.md` | .NET/ASP.NET Core profile |
| `plugins/prime-skills/skills/prime-review-dotnet/review-criteria.md` | .NET/ASP.NET review areas + category order |
| `README.md` | Document new skills |
| `.claude-plugin/marketplace.json` | Broader marketplace description |
| `.cursor-plugin/marketplace.json` | Broader marketplace description |
| `docs/superpowers/specs/2026-11-13-wave2-prime-review-design.md` | Status line after plan exists |

---

### Task 1: prime-review-vue

**Files:**
- Create: `plugins/prime-skills/skills/prime-review-vue/SKILL.md`
- Create: `plugins/prime-skills/skills/prime-review-vue/review-criteria.md`

**Interfaces:**
- Consumes: `../../shared/audit-procedure.md`, `../../shared/finding-contract.md`
- Produces: Vue skill with Nuxt optional layer (skip when not detected)

- [ ] **Step 1: Confirm the skill folder does not exist**

Run:

```bash
test ! -d plugins/prime-skills/skills/prime-review-vue && echo MISSING
```

Expected: `MISSING`

- [ ] **Step 2: Create `SKILL.md`**

Write this full content to `plugins/prime-skills/skills/prime-review-vue/SKILL.md`:

```markdown
---
name: prime-review-vue
description: Use when the user asks for a read-only audit of Vue (Options or Composition API) or Nuxt UI code in one folder, with the findings recorded as Pending tasks in a TODO markdown file.
argument-hint: "<folder> <todo-file>"
disable-model-invocation: true
---

# Prime Review Vue

Audit one folder of Vue (Options and Composition API, and Nuxt when present) and record verified, actionable findings in a TODO file. Analysis only. Not for WordPress/`@wordpress/*` plugin UI — use `prime-review-js` for that.

Read `../../shared/audit-procedure.md` (relative to this skill's folder) and follow it with this profile.

## Profile

- **Arguments**: `$ARGUMENTS`
- **Criteria**: `review-criteria.md` in this skill's folder
- **Finding contract**: `../../shared/finding-contract.md` (relative to this skill's folder)
- **Source**: `.vue .js .jsx .mjs .cjs .ts .tsx`
- **Exclusions**: build output (directories from the build config and `.gitignore`), minified files, `.d.ts` files, `node_modules`.
- **Project files**: `package.json`, `vite.config.*` / `vue.config.*` when present, `nuxt.config.*` when present, `tsconfig.*`, the test config, to learn framework detection (`nuxt` dependency or `nuxt.config.*`), build output directories, the JS/TS test framework, and the lint and type-check scripts.
- **Extra prep**: detect Nuxt. Treat the project as Nuxt when `package.json` lists a `nuxt` dependency or `nuxt.config.*` exists. When Nuxt is not detected, skip Nuxt-only Map items, the Nuxt criteria category, and the Nuxt lens.
- **Tools**: the project's JS/TS lint and type-check scripts.
- **Map**:
  - Entry points and how they mount (bundler entries, root components, Nuxt `pages`/`layouts` when Nuxt).
  - SFC / module graph: shared components, composables, and utilities.
  - State: Options `data`/`computed`/`watch`; Composition refs/reactive/computed; Pinia or Vuex when used; what writes to state.
  - Props, emits, provide/inject, and slots as contracts between components.
  - Client↔server data paths: every fetch/RPC call, its endpoint, and the handler that serves it.
  - When Nuxt is detected: pages/layouts, route middleware, server routes/API used by in-scope UI, and auto-import impact on in-scope symbols.
- **Extra scope**: API and server handlers called from in-scope files (follow them even if outside `FOLDER` when they are the contract).
- **Cross-slice**: duplication across modules, architecture, dependency direction and bundle composition.
- **Lenses**:
  1. Data/async contract: every request's payload and every response field the code reads, including error shapes; races, double submits, updates after unmount, swallowed rejections, UI left inconsistent after failure.
  2. Vue reactivity/lifecycle correctness: Options and Composition pitfalls, watchers, keying, owned vs derived state, mutating props, component responsibility.
  3. Nuxt boundaries (only when Nuxt detected): server/client splits, payload/data-fetching mistakes, server-route auth gaps, leaking secrets or server-only modules to the client.
  4. Accessibility and XSS sinks: interactive semantics, labels, focus, live regions; `v-html` and unsafe URL/`href` construction.
- **Example**:

  > `FilterBar.vue:58` (`useFilters`) watches the whole `filters` reactive object and refetches on every keystroke, so three field edits send three requests and an older response can overwrite a newer one. Trigger the request from the submit handler instead. Keep the request parameters and initial-load behavior unchanged. Verify with tests for initial load, submit, reset and API failure.
```

- [ ] **Step 3: Create `review-criteria.md`**

Write this full content to `plugins/prime-skills/skills/prime-review-vue/review-criteria.md`:

```markdown
# Review criteria

Shared by the main agent and every slice sub-agent for Prime Review Vue.

Also read `../../shared/finding-contract.md` (What not to flag and Finding contract). Use the Category list below for every finding's Category field.

## Category list (section order)

Security, Correctness, Vue, Nuxt, Performance, Accessibility, Simplicity, Tests

Omit the Nuxt section in `TODO_FILE` when Extra prep did not detect Nuxt.

## Review areas

Check what applies; skip what doesn't.

**Security**
- XSS sinks: `v-html`, `innerHTML` equivalents, and `href`/`src` built from data (`javascript:` URLs). Trace whether the value can be user- or server-controlled and whether it is escaped or sanitized.
- Secrets or privileged tokens shipped in client bundles or public env vars (`NUXT_PUBLIC_*`, `VITE_*` and equivalents).
- Authz enforced only in the client UI while a callable API/server route lacks the same check (follow Extra scope handlers).
- Third-party or dynamically loaded scripts without a clear trust boundary.

**Correctness**
- Unhandled promise rejections; request errors swallowed or shown as success.
- Response shapes assumed without guards where the server can return errors or empty data.
- Races: older responses overwriting newer state, double submits, state updates after unmount.
- Missing loading, error or empty states; UI left inconsistent after a failure.
- Assumptions about globals, env, DOM elements or script/module load order.

**Vue**
- Options vs Composition consistency bugs that cause real failures (duplicate sources of truth, mixed patterns fighting each other).
- Reactivity pitfalls: lost reactivity, mutating props, incorrect `ref`/`reactive` usage, watchers that should be computed or event handlers.
- List `key` usage; component responsibility; provide/inject contracts that silently break consumers.
- Lifecycle mistakes: missing cleanup for listeners/timers; work that belongs in an event handler living in `onMounted`/`watch` with concrete cost.

**Nuxt**
- Only when Nuxt is detected. Skip entirely otherwise.
- Server/client boundary mistakes; server-only data, modules or secrets reaching the client.
- Data-fetching/caching mistakes with a concrete failure mode (stale privileged data, missing await, wrong keying).
- Server routes or middleware that mutate state without authz/authn the project relies on.
- Route params / query trusted without validation before use in queries or redirects.

**Performance**
- Duplicate or unnecessary requests; requests inside loops.
- Expensive work during render with a real cost; unthrottled scroll, resize or input handlers.
- Listeners, timers, observers and subscriptions without cleanup.
- Heavy imports: whole libraries for one function; large modules pulled into client bundles that only a small leaf needs.

**Accessibility**
- Clickable `div`/`span` instead of buttons or links; keyboard access; visible focus.
- Inputs without labels; icon buttons without accessible names.
- Modals that don't trap and return focus; dynamic messages not announced (live regions).
- Incorrect or redundant ARIA.

**Simplicity**
- Delete: dead code, unused exports, options and dependencies. Search the whole repo, including tests, config and string or dynamic references, before claiming anything is unused.
- Reuse: logic the codebase already has as a helper, composable or component (name the path).
- Native: code or a dependency doing what the platform, Vue, or Nuxt already does.
- YAGNI: abstractions with one implementation, wrappers that only pass calls through, options nobody sets.
- Merge: near-copies that must change together.
- Split: one component or function doing several unrelated jobs. Split by job, never by line count.
- Circular imports; dependencies that point the wrong way between layers.

**Tests**
- Using the existing JS/TS test framework, find untested logic that carries real risk: the findings above, data transforms, async and error paths. Don't ask for tests of trivial presentational components.
- Tests in the audited folder that are broken, skipped or assert nothing.
```

- [ ] **Step 4: Verify Vue skill**

Run:

```bash
rg -n "^name:|Finding contract:|Nuxt|Options|Composition" plugins/prime-skills/skills/prime-review-vue/SKILL.md
rg -n "Category list|Omit the Nuxt|v-html|WordPress" plugins/prime-skills/skills/prime-review-vue/review-criteria.md
```

Expected: profile names Nuxt detection and Options/Composition; Category list includes Nuxt with omit rule; `v-html` present; WordPress only as a “use prime-review-js” redirect in SKILL, not as assumptions in criteria beyond that.

- [ ] **Step 5: Human git reminder**

Do not commit.

---

### Task 2: prime-review-nestjs

**Files:**
- Create: `plugins/prime-skills/skills/prime-review-nestjs/SKILL.md`
- Create: `plugins/prime-skills/skills/prime-review-nestjs/review-criteria.md`

**Interfaces:**
- Consumes: shared procedure + finding-contract
- Produces: NestJS skill with independent microservices and GraphQL optional layers

- [ ] **Step 1: Confirm the skill folder does not exist**

Run:

```bash
test ! -d plugins/prime-skills/skills/prime-review-nestjs && echo MISSING
```

Expected: `MISSING`

- [ ] **Step 2: Create `SKILL.md`**

Write this full content to `plugins/prime-skills/skills/prime-review-nestjs/SKILL.md`:

```markdown
---
name: prime-review-nestjs
description: Use when the user asks for a read-only audit of NestJS application or module code in one folder, with the findings recorded as Pending tasks in a TODO markdown file.
argument-hint: "<folder> <todo-file>"
disable-model-invocation: true
---

# Prime Review NestJS

Audit one folder of NestJS code (including microservices and GraphQL when present) and record verified, actionable findings in a TODO file. Analysis only.

Read `../../shared/audit-procedure.md` (relative to this skill's folder) and follow it with this profile.

## Profile

- **Arguments**: `$ARGUMENTS`
- **Criteria**: `review-criteria.md` in this skill's folder
- **Finding contract**: `../../shared/finding-contract.md` (relative to this skill's folder)
- **Source**: `.ts .js`
- **Exclusions**: `dist/`, `node_modules`, generated clients and build output from the project config / `.gitignore`.
- **Project files**: `package.json`, `nest-cli.json`, `tsconfig.*`, the test config, to learn Nest major, optional microservices/GraphQL packages, and the lint, type-check and test scripts.
- **Extra prep**: detect optional layers independently.
  - Microservices: enable when `package.json` lists `@nestjs/microservices` and/or in-scope code uses microservice transports, `@MessagePattern`, or `@EventPattern`. Otherwise skip microservices-only Map items, the Microservices criteria category, and the microservices lens.
  - GraphQL: enable when `package.json` lists `@nestjs/graphql` (or Nest GraphQL driver packages such as Apollo/Mercurius integrations used by the project) and/or in-scope code defines resolvers / `@Query` / `@Mutation`. Otherwise skip GraphQL-only Map items, the GraphQL criteria category, and the GraphQL lens.
- **Tools**: the project's lint, type-check, and test scripts.
- **Map**:
  - Root / `AppModule` and feature module graph (imports, exports, providers).
  - Controllers and route handlers in scope.
  - Providers/services and their injected dependencies.
  - Global and route pipes, guards, interceptors, and filters that apply to in-scope handlers.
  - DTO / `ValidationPipe` usage and response serialization.
  - When microservices detected: message/event patterns and transport config used by in-scope handlers.
  - When GraphQL detected: resolvers, schema or code-first models, and GraphQL-specific guards/interceptors used in scope.
- **Extra scope**: shared modules, guards, pipes, and filters imported by in-scope code; document assumed external HTTP/RPC contracts without auditing unrelated services end-to-end.
- **Cross-slice**: duplication, circular module imports, dependency direction, repeated authz gaps across sibling controllers or resolvers.
- **Lenses**:
  1. Authn/authz and guard gaps: public vs protected routes, missing guards, trusting client-supplied identity.
  2. Request → handler → response validation: DTOs, pipes, exception filters, partial writes, wrong status or contract.
  3. DI and module boundaries: provider scope mistakes, circular deps, leaking internals across modules.
  4. Optional layers: when microservices detected, message payload validation, ack/retry/idempotency gaps with concrete loss, trusting network identity without auth; when GraphQL detected, resolver auth gaps, N+1 with concrete cost, field exposure of sensitive data, input validation gaps. Run both halves of this lens when both layers are detected.
- **Example**:

  > `orders.controller.ts:41` (`OrdersController.create`) is `@Public()` while `OrdersService.create` writes `userId` from the body, so any caller can create orders for another user. Require the authenticated user (or an admin guard) and ignore client-supplied `userId`. Leave the success response shape unchanged. Verify with e2e tests for anonymous, wrong-user, and admin paths.
```

- [ ] **Step 3: Create `review-criteria.md`**

Write this full content to `plugins/prime-skills/skills/prime-review-nestjs/review-criteria.md`:

```markdown
# Review criteria

Shared by the main agent and every slice sub-agent for Prime Review NestJS.

Also read `../../shared/finding-contract.md` (What not to flag and Finding contract). Use the Category list below for every finding's Category field.

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
```

- [ ] **Step 4: Verify NestJS skill**

Run:

```bash
rg -n "^name:|Finding contract:|microservices|GraphQL|Extra prep" plugins/prime-skills/skills/prime-review-nestjs/SKILL.md
rg -n "Category list|Omit the Microservices|Omit the GraphQL|WordPress" plugins/prime-skills/skills/prime-review-nestjs/review-criteria.md || true
```

Expected: independent microservices and GraphQL detection; Category omit rules for both; no WordPress.

- [ ] **Step 5: Human git reminder**

Do not commit.

---

### Task 3: prime-review-python

**Files:**
- Create: `plugins/prime-skills/skills/prime-review-python/SKILL.md`
- Create: `plugins/prime-skills/skills/prime-review-python/review-criteria.md`

**Interfaces:**
- Consumes: shared procedure + finding-contract
- Produces: general Python skill with FastAPI optional layer

- [ ] **Step 1: Confirm the skill folder does not exist**

Run:

```bash
test ! -d plugins/prime-skills/skills/prime-review-python && echo MISSING
```

Expected: `MISSING`

- [ ] **Step 2: Create `SKILL.md`**

Write this full content to `plugins/prime-skills/skills/prime-review-python/SKILL.md`:

```markdown
---
name: prime-review-python
description: Use when the user asks for a read-only audit of Python code in one folder (including FastAPI when present), with the findings recorded as Pending tasks in a TODO markdown file.
argument-hint: "<folder> <todo-file>"
disable-model-invocation: true
---

# Prime Review Python

Audit one folder of Python code (and FastAPI when present) and record verified, actionable findings in a TODO file. Analysis only.

Read `../../shared/audit-procedure.md` (relative to this skill's folder) and follow it with this profile.

## Profile

- **Arguments**: `$ARGUMENTS`
- **Criteria**: `review-criteria.md` in this skill's folder
- **Finding contract**: `../../shared/finding-contract.md` (relative to this skill's folder)
- **Source**: `.py`
- **Exclusions**: `venv/`, `.venv/`, `__pycache__/`, build/dist artifacts, generated code, and paths ignored by the project VCS ignore files.
- **Project files**: `pyproject.toml` and/or `requirements*.txt` / `Pipfile`, pytest / ruff / mypy (or equivalent) configs, to learn Python version, FastAPI detection, and lint/type-check/test commands.
- **Extra prep**: detect FastAPI. Treat the project as FastAPI when dependencies list `fastapi` or in-scope code constructs `FastAPI(` / `APIRouter`. When FastAPI is not detected, skip FastAPI-only Map items, the FastAPI criteria category, and the FastAPI lens.
- **Tools**: the project's lint, type-check, and test commands when present (for example ruff, mypy, pytest).
- **Map**:
  - Entrypoints (CLI, `__main__`, app factory, scripts).
  - Package layout and public modules used in scope.
  - I/O and side effects: files, DB, network, subprocess.
  - Concurrency model used in scope (async, threads, processes).
  - When FastAPI is detected: routes, dependency injection, Pydantic models, middleware, background tasks, security dependencies.
- **Extra scope**: imported modules that define the in-scope contract; do not audit entire third-party frameworks.
- **Cross-slice**: duplication, layering, circular imports, dual write paths.
- **Lenses**:
  1. Input validation, injection, and secrets handling.
  2. Correctness and error paths: partial failure, swallowed exceptions, wrong contracts.
  3. Resource and async safety: leaks, blocking calls on event loops, cancellation gaps with concrete cost.
  4. FastAPI deps/security (only when FastAPI detected): dependency overrides, auth gaps, response model leakage, unsafe CORS.
- **Example**:

  > `api/users.py:77` (`update_user`) builds SQL with an f-string from `user_id`, so a crafted id can alter the query. Use a parameterized query or the project's query API. Keep the response shape unchanged. Verify with tests for valid update, missing user, and hostile `user_id` values.
```

- [ ] **Step 3: Create `review-criteria.md`**

Write this full content to `plugins/prime-skills/skills/prime-review-python/review-criteria.md`:

```markdown
# Review criteria

Shared by the main agent and every slice sub-agent for Prime Review Python.

Also read `../../shared/finding-contract.md` (What not to flag and Finding contract). Use the Category list below for every finding's Category field.

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
```

- [ ] **Step 4: Verify Python skill**

Run:

```bash
rg -n "^name:|Finding contract:|FastAPI|Extra prep" plugins/prime-skills/skills/prime-review-python/SKILL.md
rg -n "Category list|Omit the FastAPI|pickle|WordPress" plugins/prime-skills/skills/prime-review-python/review-criteria.md || true
```

Expected: FastAPI detection and omit rule; security mentions pickle; no WordPress.

- [ ] **Step 5: Human git reminder**

Do not commit.

---

### Task 4: prime-review-dotnet

**Files:**
- Create: `plugins/prime-skills/skills/prime-review-dotnet/SKILL.md`
- Create: `plugins/prime-skills/skills/prime-review-dotnet/review-criteria.md`

**Interfaces:**
- Consumes: shared procedure + finding-contract
- Produces: broader .NET skill with ASP.NET Core optional layer

- [ ] **Step 1: Confirm the skill folder does not exist**

Run:

```bash
test ! -d plugins/prime-skills/skills/prime-review-dotnet && echo MISSING
```

Expected: `MISSING`

- [ ] **Step 2: Create `SKILL.md`**

Write this full content to `plugins/prime-skills/skills/prime-review-dotnet/SKILL.md`:

```markdown
---
name: prime-review-dotnet
description: Use when the user asks for a read-only audit of C# /.NET code in one folder (including ASP.NET Core when present), with the findings recorded as Pending tasks in a TODO markdown file.
argument-hint: "<folder> <todo-file>"
disable-model-invocation: true
---

# Prime Review .NET

Audit one folder of C# /.NET code (and ASP.NET Core when present) and record verified, actionable findings in a TODO file. Analysis only.

Read `../../shared/audit-procedure.md` (relative to this skill's folder) and follow it with this profile.

## Profile

- **Arguments**: `$ARGUMENTS`
- **Criteria**: `review-criteria.md` in this skill's folder
- **Finding contract**: `../../shared/finding-contract.md` (relative to this skill's folder)
- **Source**: `.cs`; `.cshtml` / `.razor` only when they are the root cause of a verified issue (not style nitpicks).
- **Exclusions**: `bin/`, `obj/`, generated files, `node_modules`, and build outputs from the project / `.gitignore`.
- **Project files**: `*.csproj`, `*.sln`, `Directory.Build.props` when present, related test projects, `.editorconfig` / analyzer configs, to learn TFMs, ASP.NET Core detection, and build/test/analyzer commands.
- **Extra prep**: detect ASP.NET Core. Treat the project as ASP.NET Core when the project uses the Web SDK, references `Microsoft.AspNetCore.*`, or hosts via typical `Program`/`Startup` web host patterns. When ASP.NET Core is not detected, skip ASP.NET-only Map items, the ASP.NET Core criteria category, and the ASP.NET lens.
- **Tools**: `dotnet build`, `dotnet test`, and analyzer commands when available for the solution or project under review.
- **Map**:
  - Project references and the public API surface used in scope.
  - Types, nullability annotations, and async APIs in scope.
  - DI registrations consumed by in-scope types.
  - I/O: files, DB, HTTP clients, messaging.
  - When ASP.NET Core is detected: endpoints (controllers or minimal APIs), middleware pipeline, auth schemes/policies, model binding and validation.
- **Extra scope**: interfaces and implementations crossed by in-scope calls when they define the contract.
- **Cross-slice**: duplication, layering, DI lifetime mismatches across features, repeated authz gaps.
- **Lenses**:
  1. Authz and sensitive data: secrets, PII logging, over-exposure in APIs or logs.
  2. Nullability, async, and disposal correctness: NRE risks, sync-over-async, missing dispose.
  3. DI lifetimes and coupling: captive dependencies, god services, wrong-direction references.
  4. ASP.NET request pipeline (only when ASP.NET detected): middleware order, endpoint auth, model binding/validation gaps, CSRF on cookie auth.
- **Example**:

  > `OrdersController.cs:88` (`Create`) binds `CreateOrderRequest` including `UserId` and passes it to the store with no authz check, so any authenticated caller can create orders for another user. Set `UserId` from the authenticated principal (or require an admin policy) and stop accepting client `UserId`. Leave the success response shape unchanged. Verify with tests for anonymous, wrong-user, and admin paths.
```

- [ ] **Step 3: Create `review-criteria.md`**

Write this full content to `plugins/prime-skills/skills/prime-review-dotnet/review-criteria.md`:

```markdown
# Review criteria

Shared by the main agent and every slice sub-agent for Prime Review .NET.

Also read `../../shared/finding-contract.md` (What not to flag and Finding contract). Use the Category list below for every finding's Category field.

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
```

- [ ] **Step 4: Verify .NET skill**

Run:

```bash
rg -n "^name:|Finding contract:|ASP.NET|Extra prep" plugins/prime-skills/skills/prime-review-dotnet/SKILL.md
rg -n "Category list|Omit the ASP.NET|IDisposable|WordPress" plugins/prime-skills/skills/prime-review-dotnet/review-criteria.md || true
```

Expected: ASP.NET detection and omit rule; IDisposable covered; no WordPress.

- [ ] **Step 5: Human git reminder**

Do not commit.

---

### Task 5: Docs and marketplace copy

**Files:**
- Modify: `README.md`
- Modify: `.claude-plugin/marketplace.json`
- Modify: `.cursor-plugin/marketplace.json`
- Modify: `docs/superpowers/specs/2026-11-13-wave2-prime-review-design.md` (status line)

**Interfaces:**
- Consumes: new skill names and invocation patterns from Tasks 1–4
- Produces: discoverable docs that match installed skills

- [ ] **Step 1: Update README intro and skills table**

Replace the opening line with:

```markdown
A Claude Code and Cursor plugin marketplace with read-only code-review skills for WordPress PHP/JS, React/Next, Angular, Laravel, Vue/Nuxt, NestJS, Python/FastAPI, and .NET/ASP.NET Core.
```

Add these rows to the skills table (keep existing PHP, JS, React, Angular, Laravel, WP Core Reference rows):

| Skill | Claude Code | Cursor | What it does |
|-------|-------------|--------|--------------|
| Prime Review Vue | `/prime-skills:prime-review-vue <folder> <todo-file>` | `/prime-review-vue <folder> <todo-file>` | Read-only audit of Vue (Options/Composition) / Nuxt UI in one folder. Records verified findings as Pending tasks in a TODO file. |
| Prime Review NestJS | `/prime-skills:prime-review-nestjs <folder> <todo-file>` | `/prime-review-nestjs <folder> <todo-file>` | Read-only audit of NestJS (plus microservices/GraphQL when present) in one folder. Records verified findings as Pending tasks in a TODO file. |
| Prime Review Python | `/prime-skills:prime-review-python <folder> <todo-file>` | `/prime-review-python <folder> <todo-file>` | Read-only audit of Python (plus FastAPI when present) in one folder. Records verified findings as Pending tasks in a TODO file. |
| Prime Review .NET | `/prime-skills:prime-review-dotnet <folder> <todo-file>` | `/prime-review-dotnet <folder> <todo-file>` | Read-only audit of C# /.NET (plus ASP.NET Core when present) in one folder. Records verified findings as Pending tasks in a TODO file. |

Update the sentence under the table so React, Angular, Laravel, Vue, NestJS, Python, and .NET use `shared/finding-contract.md`; WordPress PHP/JS still embed their own Finding contract.

- [ ] **Step 2: Broaden marketplace descriptions**

In both `.claude-plugin/marketplace.json` and `.cursor-plugin/marketplace.json`, set the top-level `description` to:

```text
Prime code-review skills for WordPress, React/Next, Angular, Laravel, Vue/Nuxt, NestJS, Python/FastAPI, and .NET/ASP.NET Core
```

Leave individual plugin entry short descriptions as generic audit / WP stubs text unless they currently list frameworks — then align with the new top-level description. Do not remove `wp-core-reference`.

- [ ] **Step 3: Mark the Wave-2 design spec status**

In `docs/superpowers/specs/2026-11-13-wave2-prime-review-design.md`, set the status line to:

```markdown
Status: Approved; implementation plan at `docs/superpowers/plans/2026-11-13-wave2-prime-review.md`. Human owns commit/push.
```

- [ ] **Step 4: Verify docs consistency**

Run:

```bash
rg -n "prime-review-vue|prime-review-nestjs|prime-review-python|prime-review-dotnet" README.md
rg -n "Vue/Nuxt|NestJS|Python/FastAPI|\\.NET/ASP.NET" .claude-plugin/marketplace.json .cursor-plugin/marketplace.json
```

Expected: all four skills documented in README; marketplace descriptions list the new stacks.

- [ ] **Step 5: Human git reminder**

Do not commit or push.

---

### Task 6: Wave-2 verification gate

**Files:**
- Read-only checks across the plugin tree

**Interfaces:**
- Consumes: all Wave-2 outputs
- Produces: pass/fail checklist before human commit

- [ ] **Step 1: File inventory**

Run:

```bash
ls plugins/prime-skills/skills/prime-review-vue/SKILL.md plugins/prime-skills/skills/prime-review-vue/review-criteria.md
ls plugins/prime-skills/skills/prime-review-nestjs/SKILL.md plugins/prime-skills/skills/prime-review-nestjs/review-criteria.md
ls plugins/prime-skills/skills/prime-review-python/SKILL.md plugins/prime-skills/skills/prime-review-python/review-criteria.md
ls plugins/prime-skills/skills/prime-review-dotnet/SKILL.md plugins/prime-skills/skills/prime-review-dotnet/review-criteria.md
ls plugins/prime-skills/shared/finding-contract.md plugins/prime-skills/shared/audit-procedure.md
```

Expected: all listed paths exist.

- [ ] **Step 2: Shared files and prior skills unchanged by Wave-2 body edits**

Run:

```bash
rg -n "disable-model-invocation: true" plugins/prime-skills/skills/prime-review-vue/SKILL.md plugins/prime-skills/skills/prime-review-nestjs/SKILL.md plugins/prime-skills/skills/prime-review-python/SKILL.md plugins/prime-skills/skills/prime-review-dotnet/SKILL.md
rg -n "finding-contract.md" plugins/prime-skills/skills/prime-review-vue/SKILL.md plugins/prime-skills/skills/prime-review-nestjs/SKILL.md plugins/prime-skills/skills/prime-review-python/SKILL.md plugins/prime-skills/skills/prime-review-dotnet/SKILL.md
rg -n "Omit the Nuxt|Omit the Microservices|Omit the FastAPI|Omit the ASP.NET" plugins/prime-skills/skills/prime-review-vue/review-criteria.md plugins/prime-skills/skills/prime-review-nestjs/review-criteria.md plugins/prime-skills/skills/prime-review-python/review-criteria.md plugins/prime-skills/skills/prime-review-dotnet/review-criteria.md
```

Expected: all four skills disable model invocation, point at finding-contract, and have omit rules for their optional layers (NestJS has Microservices; GraphQL omit is separate — also confirm):

```bash
rg -n "Omit the GraphQL" plugins/prime-skills/skills/prime-review-nestjs/review-criteria.md
```

Expected: match.

- [ ] **Step 3: WP and Phase 1–2 skill folders not rewritten for Wave-2**

Run:

```bash
git diff -- plugins/prime-skills/skills/prime-review-php plugins/prime-skills/skills/prime-review-js plugins/prime-skills/skills/prime-review-react plugins/prime-skills/skills/prime-review-angular plugins/prime-skills/skills/prime-review-laravel plugins/prime-skills/shared
```

Expected: empty diff for those paths from Wave-2 work (pre-existing uncommitted Phase 1–2 changes may appear if never committed — if so, confirm Wave-2 did not add further edits there; `shared/` must show no new Wave-2 edits).

- [ ] **Step 4: Optional plugin validate**

If Claude Code CLI is available:

```bash
claude plugin validate .
```

Expected: success, or note "CLI unavailable" and rely on Steps 1–3 plus README grep.

- [ ] **Step 5: Stop for human**

Summarize created/modified paths. Do not commit. Do not push.

---

## Plan self-review

1. **Spec coverage:** Vue, NestJS, Python, .NET profiles + criteria, optional-layer detection, README, both marketplaces, verification, no shared-kernel edits, no agent git — each has a task/step.
2. **Placeholders:** none; full file bodies inlined.
3. **Consistency:** Category names and omit rules match the Wave-2 design; Nest microservices and GraphQL are independent; delivery order Vue → NestJS → Python → .NET → docs → gate.
