# Wave 2 Prime Review — Design (Vue, NestJS, Python, .NET)

Date: 2026-11-13  
Status: Approved; implementation plan at `docs/superpowers/plans/2026-11-13-wave2-prime-review.md`. Human owns commit/push.  
Prior art: `docs/superpowers/specs/2026-11-13-multi-framework-prime-review-design.md` (Phase 1 React/Angular + Phase 2 Laravel).  
Git: Do **not** commit or push from the agent; the human owns all git operations.

## Problem

Prime Review now covers WordPress PHP/JS plus React (Next when detected), Angular, and Laravel. Reviewers still need first-class packs for Vue (including Nuxt when present), NestJS (including microservices/GraphQL when present), general Python (FastAPI when present), and broader C# /.NET (ASP.NET Core when present), without changing the shared audit procedure or finding kernel.

## Goals

- Add four dedicated skills that reuse `shared/audit-procedure.md` and `shared/finding-contract.md`.
- Mirror the Phase 1–2 composition: profile `SKILL.md` + `review-criteria.md`, optional-layer detection with skippable Map/lens/category.
- Implement all four in one pass: Vue → NestJS → Python → .NET, then README + marketplace.
- Keep WordPress and existing multi-framework skills behaviorally unchanged.
- Preserve invocation: manual skills, `<folder> <todo-file>`, analysis-only, only `TODO_FILE` may be written.

## Non-goals

- A single auto-detecting “router” skill.
- Migrating WordPress criteria onto the shared kernel.
- Changing audit output format, ponytail prerequisite, or finding shape.
- Dedicated Express, Django, Flask, or Blazor-primary packs (only the optional layers listed below).
- Implementing code fixes as part of the skill.
- Agent commits or pushes.

## Decisions (locked)

| Topic | Choice |
|-------|--------|
| Packaging | Hybrid Approach 1: four new dedicated skills sharing procedure + finding-contract |
| Vue surface | Broad Options + Composition API; Nuxt when detected |
| NestJS surface | Standard NestJS + microservices/GraphQL when detected |
| Python surface | General Python + FastAPI when detected |
| .NET surface | Broader C# /.NET + ASP.NET Core when detected |
| Delivery | One pass: Vue → NestJS → Python → .NET → docs |
| Shared procedure/kernel | Unchanged (no new steps; no finding-shape edits) |
| Git | Human only; no agent commit/push |

## Architecture

```
plugins/prime-skills/
  shared/
    audit-procedure.md          # Unchanged
    finding-contract.md         # Unchanged
  skills/
    prime-review-vue/           # NEW
    prime-review-nestjs/        # NEW
    prime-review-python/        # NEW
    prime-review-dotnet/        # NEW
    (+ existing php/js/react/angular/laravel)
```

### Composition rules

- Every new skill has:
  - `SKILL.md` — profile (Arguments, Criteria, Finding contract, Source, Exclusions, Project files, Tools, Map, Extra scope, Cross-slice, Lenses, Example; Extra prep for optional-layer detection)
  - `review-criteria.md` — review areas and category order only
  - Explicit pointers to `../../shared/audit-procedure.md` and `../../shared/finding-contract.md`
- `disable-model-invocation: true` and `argument-hint: "<folder> <todo-file>"` for every new skill.
- Profile Map always wins over any residual WordPress phrasing in the shared procedure.
- Optional layers (Nuxt, Nest microservices, Nest GraphQL, FastAPI, ASP.NET Core): detect in Extra prep; when absent, skip that Map subset, lens, and criteria section entirely.
- Category names stay per-skill; do not centralize them in the finding-contract.

## Skill inventory

### `prime-review-vue`

- **When**: Read-only audit of Vue (Options + Composition) UI code in one folder → Pending tasks in a TODO file. Not for WordPress/`@wordpress/*` plugin UI — use `prime-review-js`.
- **Source**: `.vue .js .jsx .mjs .cjs .ts .tsx`
- **Exclusions**: build output, minified files, `.d.ts`, `node_modules`
- **Project files**: `package.json`, `vite.config.*` / `vue.config.*`, `nuxt.config.*` when present, `tsconfig.*`, lint/test scripts
- **Nuxt detection**: Treat as Nuxt when `package.json` lists a `nuxt` dependency or `nuxt.config.*` exists. When not detected, skip Nuxt-only Map items, the Nuxt criteria category, and the Nuxt lens.
- **Tools**: project JS/TS lint and type-check scripts (run if available; never claim clean if missing).
- **Map**:
  - Entry/mount points (bundler entries, root components, Nuxt `pages`/`layouts` when Nuxt).
  - SFC / module graph: shared components, composables, utilities.
  - State: Options `data`/`computed`/`watch`; Composition refs/reactive/computed; Pinia/Vuex when used.
  - Props, emits, provide/inject, and slots as contracts.
  - Client↔server data paths: every fetch/RPC call and its handler.
  - When Nuxt: pages/layouts, route middleware, server routes/API used by in-scope UI, auto-import impact on in-scope symbols.
- **Extra scope**: API/server handlers called from in-scope files (follow them when they are the contract).
- **Cross-slice**: duplication, dependency direction, bundle composition.
- **Lenses**:
  1. Data/async contract (payloads, error shapes, races, updates after unmount).
  2. Vue reactivity/lifecycle correctness (Options + Composition; watchers; keying; owned vs derived state).
  3. Nuxt boundaries (only when Nuxt detected): server/client splits, payload/data fetching mistakes, server-route auth gaps, leaking secrets to the client.
  4. Accessibility and XSS sinks (`v-html`, unsafe URL/`href` construction).
- **Criteria categories (order)**: Security → Correctness → Vue → Nuxt (omit if not Nuxt) → Performance → Accessibility → Simplicity → Tests

### `prime-review-nestjs`

- **When**: Read-only audit of NestJS application or module code in one folder → Pending tasks in a TODO file.
- **Source**: `.ts .js`
- **Exclusions**: `dist/`, `node_modules`, generated clients
- **Project files**: `package.json`, `nest-cli.json`, `tsconfig.*`, lint/test scripts
- **Optional detection**:
  - Microservices: `@nestjs/microservices` dependency and/or microservice transport bootstrap / `@MessagePattern` / `@EventPattern` in scope → enable microservices Map/lens/category; else skip.
  - GraphQL: `@nestjs/graphql` (or Apollo/Mercurius Nest GraphQL packages) and/or resolvers/`@Query`/`@Mutation` in scope → enable GraphQL Map/lens/category; else skip.
- **Tools**: project lint, type-check, and test scripts when present.
- **Map**:
  - Root/`AppModule` and feature module graph.
  - Controllers, providers, exports/imports.
  - Global and route pipes, guards, interceptors, filters.
  - DTO / ValidationPipe usage and serialization.
  - When microservices detected: message/event patterns, transport config used by in-scope handlers.
  - When GraphQL detected: resolvers, schema/code-first models, GraphQL-specific guards/interceptors used in scope.
- **Extra scope**: shared modules/guards/pipes imported by in-scope code; document assumed external HTTP/RPC contracts without auditing unrelated services.
- **Cross-slice**: duplication, circular module imports, dependency direction, repeated authz gaps across sibling controllers/resolvers.
- **Lenses**:
  1. Authn/authz and guard gaps (including public vs protected routes).
  2. Request → handler → response validation (DTOs, pipes, exception filters, partial writes).
  3. DI and module boundaries (provider scope, circular deps, leaking internals).
  4. Microservices correctness **or** GraphQL correctness when the corresponding layer is detected (run both lenses if both are detected).
- **Criteria categories (order)**: Security → Correctness → NestJS → Microservices (omit if not) → GraphQL (omit if not) → Performance → Simplicity → Tests

### `prime-review-python`

- **When**: Read-only audit of general Python code in one folder → Pending tasks in a TODO file. FastAPI checks when FastAPI is detected.
- **Source**: `.py`
- **Exclusions**: `venv/`, `.venv/`, `__pycache__/`, build/dist artifacts, generated code
- **Project files**: `pyproject.toml` and/or `requirements*.txt` / `Pipfile`, pytest/ruff/mypy (or equivalent) configs
- **FastAPI detection**: Treat as FastAPI when dependencies list `fastapi` or in-scope code constructs `FastAPI(` / `APIRouter`. When not detected, skip FastAPI-only Map items, category, and lens.
- **Tools**: project lint/type-check/test commands when present (ruff, mypy, pytest, etc.).
- **Map**:
  - Entrypoints (CLI, `__main__`, app factory).
  - Package layout and public modules.
  - I/O and side effects (files, DB, network, subprocess).
  - Concurrency model (async, threads, processes) used in scope.
  - When FastAPI: routes, dependency injection, Pydantic models, middleware, background tasks, security dependencies.
- **Extra scope**: imported modules that define the in-scope contract; do not audit entire third-party frameworks.
- **Cross-slice**: duplication, layering, circular imports, dual write paths.
- **Lenses**:
  1. Input validation, injection, and secrets handling.
  2. Correctness and error paths (partial failure, swallowed exceptions, wrong contracts).
  3. Resource and async safety (leaks, blocking calls on event loops, cancellation).
  4. FastAPI deps/security (only when FastAPI detected): dependency overrides, auth gaps, response model leakage, unsafe CORS.
- **Criteria categories (order)**: Security → Correctness → Python → FastAPI (omit if not) → Performance → Simplicity → Tests

### `prime-review-dotnet`

- **When**: Read-only audit of broader C# /.NET code in one folder → Pending tasks in a TODO file. ASP.NET Core checks when detected.
- **Source**: `.cs`; `.cshtml` / `.razor` only when they are the root cause of a verified issue (not style nitpicks).
- **Exclusions**: `bin/`, `obj/`, generated files, `node_modules`
- **Project files**: `*.csproj`, `*.sln`, `Directory.Build.props`, related test projects, `.editorconfig` / analyzer configs
- **ASP.NET Core detection**: Treat as ASP.NET Core when the project uses the Web SDK, references `Microsoft.AspNetCore.*`, or hosts via typical `Program`/`Startup` web host patterns. When not detected, skip ASP.NET-only Map items, category, and lens.
- **Tools**: `dotnet build` / test / analyzer commands when available for the solution or project under review.
- **Map**:
  - Project references and public API surface used in scope.
  - Types, nullability annotations, and async APIs.
  - DI registrations consumed by in-scope types.
  - I/O: files, DB, HTTP clients, messaging.
  - When ASP.NET Core: endpoints (controllers or minimal APIs), middleware pipeline, auth schemes/policies, model binding and validation.
- **Extra scope**: interfaces and implementations crossed by in-scope calls when they define the contract.
- **Cross-slice**: duplication, layering, DI lifetime mismatches across features, repeated authz gaps.
- **Lenses**:
  1. Authz and sensitive data (secrets, PII logging, over-exposure in APIs).
  2. Nullability, async, and disposal correctness.
  3. DI lifetimes and coupling (captive dependencies, god services).
  4. ASP.NET request pipeline (only when ASP.NET detected): middleware order, endpoint auth, model binding/validation gaps, CSRF on cookie auth.
- **Criteria categories (order)**: Security → Correctness → .NET → ASP.NET Core (omit if not) → Performance → Simplicity → Tests

## Criteria content outline

### Vue

- **Security**: `v-html` and other XSS sinks; unsafe URL construction; secrets in client bundles; client-only authz.
- **Correctness**: unhandled rejections; assumed response shapes; races; missing loading/error/empty states.
- **Vue**: Options vs Composition consistency bugs; reactivity pitfalls (lost reactivity, mutating props); watcher/lifecycle mistakes; list keys; component responsibility.
- **Nuxt** (when detected): server/client boundary mistakes; leaking server-only data; unsafe server routes; data-fetching/caching mistakes with concrete failure modes.
- **Performance / Accessibility / Simplicity / Tests**: same bar as React skill, adapted to Vue/Nuxt APIs.

### NestJS

- **Security**: missing guards; public route mistakes; injection via unsanitized inputs; secrets in logs; insecure microservice message trust.
- **Correctness**: ignored validation failures; wrong status/contracts; partial writes; exception filter gaps.
- **NestJS**: module import/export mistakes; provider scope errors; pipe/interceptor misuse that causes real bugs; deprecated Nest APIs for the project's Nest major.
- **Microservices** (when detected): unvalidated message payloads; missing ack/retry/idempotency where loss is concrete; trusting network identity without auth.
- **GraphQL** (when detected): resolver auth gaps; N+1 where cost is concrete; over-fetch / field exposure of sensitive data; validation gaps on inputs.
- **Performance / Simplicity / Tests**: blocking work on hot paths with evidence; dead modules; missing tests on risky handlers.

### Python

- **Security**: injection (SQL/command/path); unsafe `pickle`/`yaml`/`eval`; secret leakage; unsafe deserialization.
- **Correctness**: swallowed exceptions; wrong types/contracts; TOCTOU; partial writes.
- **Python**: resource leaks; threading/async misuse; mutable default args causing bugs; packaging/import errors that break runtime.
- **FastAPI** (when detected): missing Depends/security; response_model leakage; unvalidated body/query; CORS/`*` with credentials; background-task failure modes.
- **Performance / Simplicity / Tests**: unbounded work with concrete cost; dead code; missing tests on risky paths.

### .NET

- **Security**: missing authz; open redirects; insecure deserialization; secret logging; XSS in views when views are in scope.
- **Correctness**: null-forgiving misuse hiding real nulls; async sync-over-async; incorrect exception handling; partial writes.
- **.NET**: IDisposable/IAsyncDisposable gaps; DI lifetime bugs; API surface mistakes; nullable annotation lies that cause NREs.
- **ASP.NET Core** (when detected): endpoint auth gaps; middleware order bugs; model validation bypass; CSRF on cookie-auth forms; over-sharing DTOs.
- **Performance / Simplicity / Tests**: allocations/unbounded queries with evidence; dead endpoints; missing tests on risky paths.

## Delivery plan

1. Add `prime-review-vue` (`SKILL.md` + `review-criteria.md`).
2. Add `prime-review-nestjs` (`SKILL.md` + `review-criteria.md`).
3. Add `prime-review-python` (`SKILL.md` + `review-criteria.md`).
4. Add `prime-review-dotnet` (`SKILL.md` + `review-criteria.md`).
5. Update root `README.md` and both marketplace files (`.cursor-plugin/marketplace.json`, `.claude-plugin/marketplace.json`).
6. Optionally note in the Phase 1 design open-follow-ups that Wave 2 covers Vue/NestJS/Python/.NET (documentation only; do not rewrite Phase 1 history).
7. Stop. Human commits and pushes.

## Compatibility

- Existing review skills keep current names, criteria, and assumptions.
- New skills are additive.
- Ponytail remains a hard prerequisite for all review skills.
- Shared procedure and finding-contract stay byte-stable unless a defect is found; Wave 2 does not require edits to them.

## Success criteria

- An agent can run Vue, NestJS, Python, or .NET audits using only the new skill profile + shared procedure + shared finding kernel.
- Optional-layer checks (Nuxt, Nest microservices, Nest GraphQL, FastAPI, ASP.NET Core) run only when detected.
- Vue criteria cover Options and Composition; Python works without FastAPI; .NET works without ASP.NET Core.
- WordPress and Phase 1–2 skills are unchanged in behavior.
- No agent git commit or push as part of this work.

## Open follow-ups (explicitly deferred)

- Optional WP criteria migration onto the shared kernel — superseded by per-skill contracts: `docs/superpowers/specs/2026-11-13-per-skill-finding-contract-design.md`.
- Express / Django / Flask / Blazor-primary packs.
- A stubs/reference skill for non-WordPress stacks.
- Any auto-routing skill across frameworks.
