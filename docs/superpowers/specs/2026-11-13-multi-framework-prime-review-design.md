# Multi-framework Prime Review — Design

Date: 2026-11-13  
Status: Approved; Phase 1 and Phase 2 (Laravel) implemented in working tree — human owns commit/push. Plan: `docs/superpowers/plans/2026-11-13-multi-framework-prime-review.md`  
Git: Do **not** commit or push from the agent; the human owns all git operations.

## Problem

Prime Review today covers WordPress-oriented PHP and JS/TS only (`prime-review-php`, `prime-review-js`), plus a separate `wp-core-reference` plugin. The shared audit procedure is already framework-agnostic enough to reuse, but profiles and criteria are WordPress-specific. We need first-class review skills for React (including Next.js App Router), Angular (standalone and NgModules), and Laravel, without breaking or rewriting the existing WordPress skills.

## Goals

- Add dedicated review skills for React, Angular, and Laravel that reuse `shared/audit-procedure.md`.
- Extract a shared finding kernel so new skills do not drift on severity, confidence, or “what not to flag”.
- Design all three stacks now; implement React + Angular first; build Laravel in a later phase from this same design.
- Keep WordPress PHP/JS skills behaviorally unchanged in Phase 1.
- Preserve the existing invocation model: manual skills, arguments `<folder> <todo-file>`, analysis-only, only `TODO_FILE` may be written.

## Non-goals

- A single auto-detecting “router” skill.
- Migrating WordPress criteria onto the shared kernel in Phase 1 (optional later).
- Vue, Svelte, Django, NestJS, Ionic, or other stacks.
- Changing audit output format or the ponytail prerequisite.
- Implementing code fixes as part of the skill (still TODO-file only).
- Agent commits or pushes.

## Decisions (locked)

| Topic | Choice |
|-------|--------|
| Packaging | Hybrid: keep WP PHP/JS; add dedicated Laravel / React / Angular skills sharing the procedure |
| Delivery order | Frontend first (React + Angular), Laravel build later |
| React vs `prime-review-js` | Parallel and separate: WP JS stays; React skill is general React (no WP assumptions) |
| React surface | Core React + Next.js App Router |
| Angular surface | Broad: standalone and NgModules-era patterns |
| Laravel in this round | Full design now; implement in Phase 2 |
| Shared structure | Shared kernel + framework packs |
| Git | Human only; no agent commit/push |

## Architecture

```
plugins/prime-skills/
  shared/
    audit-procedure.md          # Existing process (prepare → map → review → TODO)
    finding-contract.md         # NEW: severity, confidence, finding shape, what-not-to-flag
  skills/
    prime-review-php/           # Unchanged (WordPress)
    prime-review-js/            # Unchanged (WordPress)
    prime-review-react/         # NEW — Phase 1
    prime-review-angular/       # NEW — Phase 1
    prime-review-laravel/       # NEW — Phase 2 build; designed here
```

### Composition rules

- Every new skill has:
  - `SKILL.md` — profile (Arguments, Criteria, Source, Exclusions, Project files, Tools, Map, Extra scope, Cross-slice, Lenses, Example; Extra prep only if needed)
  - `review-criteria.md` — review areas and category order only
  - Explicit pointers to `../../shared/audit-procedure.md` and `../../shared/finding-contract.md`
- WordPress skills remain verbatim in Phase 1 (they keep an inline Finding contract).
- `disable-model-invocation: true` and `argument-hint: "<folder> <todo-file>"` for every new skill.
- Profile Map always wins over any residual WordPress phrasing in the shared procedure.

### Shared procedure tweaks (Phase 1, light touch)

Keep the six-step process identical. Only neutralize WordPress-only default phrasing in shared Map/Finish hints (for example load assumptions and finish suggestions that assume WP admin/frontend/cron) so non-WP skills are not steered wrong. Profiles continue to supply the real Map items.

### Shared finding kernel

`shared/finding-contract.md` holds:

- Finding shape: Location, Evidence (concrete failure scenario), Verification
- Severity: Critical, High, Medium, Low (same definitions as today)
- Confidence: High or Medium only; drop lower
- Fix + Fix check + Severity in context (ponytail ladder; keep public APIs unless the bug is the behavior)
- Global What not to flag: formatter/style preferences, speculative performance without concrete cost in this code, excluded paths, changes that break project rules, `shortcut:` / `ponytail:` decisions that name their limit

Framework-specific Fix-check notes (e.g. supported browsers, Laravel PHP version, Next runtime) live in each skill’s criteria or profile Example, not as WordPress-hardcoded text in the kernel.

Category names are **not** centralized: each skill’s criteria defines its Category list and order for TODO sections.

## Skill inventory

### `prime-review-react` (Phase 1)

- **When**: Read-only audit of React / Next.js UI code in one folder → Pending tasks in a TODO file.
- **Source**: `.js .jsx .mjs .cjs .ts .tsx`
- **Exclusions**: build output, minified files, `.d.ts`, `node_modules`
- **Project files**: `package.json`, Next config if present (`next.config.*`), tsconfig, lint/test scripts
- **Next detection**: If `next` is a dependency or App Router layout exists (`app/`), enable Next map items and the Next.js review category / lens; otherwise skip Next-only checks
- **Tools**: project lint and type-check scripts (same “run if available, never claim clean if missing” rule)
- **Map**: entry/mount points; component/module graph; state (local, context, stores); client↔server data paths; for Next: route segments, server vs client components, server actions / route handlers used by in-scope UI
- **Extra scope**: API route handlers / server actions called from in-scope files
- **Cross-slice**: duplication, dependency direction, bundle composition
- **Lenses**:
  1. Data/async contract (payloads, error shapes, races)
  2. React correctness (hooks, effects, state, keys)
  3. Next App Router boundaries and caching (when Next detected)
  4. Accessibility and XSS sinks
- **Criteria categories (order)**: Security → Correctness → React → Next.js (omit section if not Next) → Performance → Accessibility → Simplicity → Tests

### `prime-review-angular` (Phase 1)

- **When**: Read-only audit of Angular app code in one folder → Pending tasks in a TODO file.
- **Source**: `.ts` and component `.html` templates; styles only when they are the root cause of a verified issue (not style nitpicks)
- **Exclusions**: build output, `node_modules`, generated files
- **Project files**: `angular.json` or `project.json`, `package.json`, `tsconfig.*`, lint/test config
- **Tools**: project lint and type-check / test scripts as configured
- **Map**: NgModule and/or standalone bootstrap; routes; DI graph; services and HTTP; forms; template↔class bindings
- **Extra scope**: Document assumed API contracts for backends called from in-scope services; do not audit the whole backend unless it is inside `FOLDER`
- **Cross-slice**: duplication, architecture layers, circular DI, feature vs shared boundaries
- **Lenses**:
  1. Template security and accessibility
  2. Change detection and async correctness
  3. DI and architecture direction
  4. Forms and validation round-trip
- **Criteria categories (order)**: Security → Correctness → Angular → Templates & forms → Performance → Accessibility → Simplicity → Tests  
  (“Angular” covers both standalone and NgModules patterns.)

### `prime-review-laravel` (designed now; Phase 2 build)

- **When**: Read-only audit of Laravel PHP/Blade in one folder → Pending tasks in a TODO file.
- **Source**: `.php` and `.blade.php`
- **Exclusions**: `vendor/`, generated files, language dumps as appropriate
- **Project files**: `composer.json`, `artisan`, relevant `config/*`, PHPUnit/Pest, Pint / PHPStan / Larastan scripts
- **Tools**: project static analysis and test commands when present
- **Map**: routes → controllers/actions; middleware; Form Requests; Eloquent models/scopes; jobs/events/listeners; policies/gates; Blade entry points
- **Extra scope**: frontend callers of in-scope API routes when they are contract consumers
- **Cross-slice**: duplication, service boundaries, repeated authz gaps across sibling routes, dual write paths
- **Lenses**:
  1. Authn/authz and mass assignment
  2. Request → DB → response round trip
  3. Queues/jobs and failure modes
  4. N+1 and query cost
- **Criteria categories (order)**: Security → Correctness → Laravel integration → Round trip & contracts → Performance → Blade/views → Simplicity → Tests
- **Extra prep**: none required analogous to `wp-core-reference` unless a Laravel stubs skill exists later; do not invent one in Phase 1–2.

## Criteria content outline

### React review areas (summary)

- **Security**: XSS sinks (`dangerouslySetInnerHTML`, raw HTML, unsafe URLs); secrets in client bundles; unsafe reliance on client-only checks for authz
- **Correctness**: unhandled rejections; assumed response shapes; races; missing loading/error/empty states
- **React**: hook rules; effect dependencies; stale closures; derived state vs effects; list keys; controlled/uncontrolled; component responsibility
- **Next.js** (when detected): server/client boundary mistakes; leaking server-only data/modules to the client; incorrect caching/revalidation; unsafe server actions; route handler auth gaps
- **Performance**: duplicate fetches; expensive render work; missing cleanup of listeners/subscriptions; heavy imports in client bundles
- **Accessibility**: roles, labels, keyboard, focus, live regions
- **Simplicity / Tests**: same delete/reuse/YAGNI/split rules as JS skill; risk-bearing untested logic

### Angular review areas (summary)

- **Security**: template binding sinks, bypassSecurityTrust*, XSS via innerHTML equivalents; open redirects; token storage mistakes
- **Correctness**: subscription leaks; ignored HTTP errors; racey async pipe / switchMap misuse; router state assumptions
- **Angular**: NgModule vs standalone consistency; DI scope mistakes; providedIn / component providers; change detection pitfalls; signals vs RxJS misuse where the project mixes them
- **Templates & forms**: reactive/template-driven validation gaps; valueChanges races; disabled control submit gaps
- **Performance**: unnecessary change detection work; large default change detection surfaces; unbounded lists without virtualization where cost is concrete
- **Accessibility / Simplicity / Tests**: same bar as React skill adapted to Angular APIs

### Laravel review areas (summary)

- **Security**: missing policies/gates/middleware; mass assignment; raw SQL / query builder injection; insecure file storage; CSRF on web routes; secrets in logs/Blade
- **Correctness**: unchecked validation failures; wrong HTTP status/contracts; timezone/serialization bugs; partial writes across DB + jobs
- **Laravel integration**: Form Requests unused; facades/services fighting project architecture; deprecated APIs for the minimum PHP/Laravel versions; misuse of Eloquent vs query builder where it causes real bugs
- **Round trip & contracts**: hostile values through validation → cast → DB → API/Blade; invariant gaps across sibling routes
- **Performance**: N+1, unbounded `get()`, missing chunking on large jobs
- **Blade/views**: unescaped output (`{!! !!}`), XSS; excessive logic in views
- **Simplicity / Tests**: dead routes/controllers; missing feature/unit coverage on risky paths

## Delivery plan

### Phase 1 — implement now

1. Add `shared/finding-contract.md`.
2. Add `prime-review-react` (`SKILL.md` + `review-criteria.md`).
3. Add `prime-review-angular` (`SKILL.md` + `review-criteria.md`).
4. Light-touch neutralize WP-only defaults in `shared/audit-procedure.md` where they mislead non-WP skills.
5. Update root `README.md` (and marketplace copy if it lists skills) for the new skills.
6. Stop. Human commits and pushes.

### Phase 2 — later

1. Add `prime-review-laravel` from this design.
2. Optional: point WP PHP/JS criteria at `shared/finding-contract.md` (dedupe only; no behavior change).

## Compatibility

- Existing `/prime-review-php` and `/prime-review-js` keep current names, criteria, and WordPress assumptions.
- New skills are additive; installing an updated plugin adds commands without removing old ones.
- Ponytail remains a hard prerequisite for all review skills.

## Success criteria

- An agent can run React or Angular audits using only the new skill profile + shared procedure + shared finding kernel.
- Next-only checks run only when Next is detected.
- Angular criteria explicitly cover both standalone and NgModules.
- Laravel can be implemented later without redesigning architecture.
- WordPress audits produce the same class of findings as before Phase 1.
- No agent git commit or push as part of this work.

## Open follow-ups (explicitly deferred)

- Optional WP criteria migration onto the shared kernel — superseded by per-skill contracts: `docs/superpowers/specs/2026-11-13-per-skill-finding-contract-design.md`.
- Additional stacks beyond React, Angular, Laravel — **Wave 2** (Vue, NestJS, Python, .NET) is specified in `docs/superpowers/specs/2026-11-13-wave2-prime-review-design.md`.
- A Laravel-specific stubs/reference skill analogous to `wp-core-reference`.
