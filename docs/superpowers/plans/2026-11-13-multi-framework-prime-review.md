# Multi-framework Prime Review (Phase 1) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a shared finding kernel plus `prime-review-react` and `prime-review-angular` skills that reuse the existing audit procedure, without changing WordPress PHP/JS skill behavior.

**Architecture:** Keep `shared/audit-procedure.md` as the process. Add `shared/finding-contract.md` for severity/confidence/what-not-to-flag. New skills are profile + review-areas only, pointing at both shared files. Neutralize a few WordPress-only defaults in the procedure so non-WP profiles are not misled. Laravel is designed in the spec but **not** built in this plan.

**Tech Stack:** Markdown Cursor/Claude skills under `plugins/prime-skills/`; no application runtime code.

## Global Constraints

- Spec: `docs/superpowers/specs/2026-11-13-multi-framework-prime-review-design.md`
- Do **not** commit or push; the human owns all git operations
- Do **not** modify `prime-review-php` or `prime-review-js` skill files in Phase 1
- Do **not** create `prime-review-laravel` in this plan (Phase 2)
- Invocation: `disable-model-invocation: true`, `argument-hint: "<folder> <todo-file>"`
- Analysis-only audits; only `TODO_FILE` may change when a skill runs
- New skills must not assume WordPress APIs

## File structure

| Path | Responsibility |
|------|----------------|
| `plugins/prime-skills/shared/finding-contract.md` | Shared finding shape, severity, confidence, Fix/Fix check, What not to flag |
| `plugins/prime-skills/shared/audit-procedure.md` | Shared process; light neutralization + how to load finding contract |
| `plugins/prime-skills/skills/prime-review-react/SKILL.md` | React/Next profile |
| `plugins/prime-skills/skills/prime-review-react/review-criteria.md` | React/Next review areas + category order |
| `plugins/prime-skills/skills/prime-review-angular/SKILL.md` | Angular profile |
| `plugins/prime-skills/skills/prime-review-angular/review-criteria.md` | Angular review areas + category order |
| `README.md` | Document new skills |
| `.claude-plugin/marketplace.json` | Broader marketplace description |
| `.cursor-plugin/marketplace.json` | Broader marketplace description |

---

### Task 1: Shared finding contract

**Files:**
- Create: `plugins/prime-skills/shared/finding-contract.md`
- Test: structural grep/read checklist below

**Interfaces:**
- Consumes: existing Finding contract wording from `prime-review-js/review-criteria.md` (framework-neutral rewrite)
- Produces: shared kernel that new skills reference; WP skills do not import it yet

- [ ] **Step 1: Confirm the file does not exist**

Run:

```bash
test ! -f plugins/prime-skills/shared/finding-contract.md && echo MISSING
```

Expected: `MISSING`

- [ ] **Step 2: Create `finding-contract.md`**

Write this full content:

```markdown
# Finding contract

Shared by prime-review skills that point here. The calling skill's `review-criteria.md` supplies Review areas and the Category list. Sub-agents read this file together with that criteria file.

## What not to flag

- Style or formatting that the project's lint or formatter governs, or a preference between equivalent patterns. Breaks of the project's own rules are not style; flag them.
- Speculative performance ("could be slow at scale") without a concrete cost in this code. Judge cost against how the code actually runs (for example one-user admin vs every visitor request vs background workers).
- Anything in the excluded paths.
- Changes that would break project rules.
- Code marked with a `shortcut:` or `ponytail:` comment that names its limit; that is a decision, unless the real load already crosses the limit.

## Finding contract

Every finding has:

- **Location**: `path:line` plus the function, method, component, or class name that owns the behavior.
- **Evidence**: what the code does, and a concrete scenario where it fails or costs something (inputs or state, then the outcome). "Could" or "might" without a scenario is not a finding.
- **Verification**: you read the relevant code, including callers and consumers where relevant (including server or API handlers when the skill's Extra scope says to follow them).
- **Severity**:
  - Critical: exploitable security issue or data loss.
  - High: functional bug, broken flow, or a severe performance or resource problem users will hit.
  - Medium: accessibility barrier, reliability gap, or a maintenance problem with real cost.
  - Low: minor improvement, test gap on low-risk code, optional optimization.
- **Confidence**: High (the failure follows directly from the code) or Medium (the code is verified, but the impact depends on runtime conditions you can't confirm). Drop anything lower.
- **Fix**: the smallest change that resolves it, chosen with the ponytail ladder: does it need to exist → already in the codebase → platform or framework API → installed dependency → one line → minimum code. Keep behavior and public APIs unless the behavior itself is the bug; if a breaking change is unavoidable, list the affected consumers. State impact in words unless you measured it.
- **Fix check**: the fix works on the project's declared runtime targets (language, framework, and browsers or platforms the project supports), for every caller and consumer, and against real response or data shapes. A fix that needs a change outside the audited folder names that change.
- **Severity in context**: if another open path already allows the same thing, lower the severity and say why. If nothing reaches the path today, call it hardening.
- **Category**: one category from the calling skill's Category list in `review-criteria.md`, using those names verbatim.
```

- [ ] **Step 3: Verify kernel contents**

Run:

```bash
rg -n "What not to flag|Severity|Confidence|Fix check|Category" plugins/prime-skills/shared/finding-contract.md
rg -n "WordPress|@wordpress|wpdb" plugins/prime-skills/shared/finding-contract.md || true
```

Expected: headings/fields match; second command prints nothing (no WP-specific leaks).

- [ ] **Step 4: Human git reminder**

Do not commit. Leave changes unstaged/uncommitted for the human.

---

### Task 2: Neutralize shared audit procedure

**Files:**
- Modify: `plugins/prime-skills/shared/audit-procedure.md`
- Test: diff + grep checklist

**Interfaces:**
- Consumes: profile fields as today; optionally a Finding contract path from the profile
- Produces: WP-neutral Map load wording; Finish suggestion that is not WP-only; Criteria prep that loads shared finding contract when the profile names it

- [ ] **Step 1: Update Hard rules line about the Finding contract**

In `plugins/prime-skills/shared/audit-procedure.md`, replace:

```markdown
- Every recorded finding meets the Finding contract in the Criteria file. Recording nothing for an area is a valid outcome. Never pad.
```

with:

```markdown
- Every recorded finding meets the Finding contract (in the Criteria file, or in the shared finding-contract file when the profile names one). Recording nothing for an area is a valid outcome. Never pad.
```

- [ ] **Step 2: Update Prepare step 3**

Replace:

```markdown
3. Read the Criteria file. Resolve its absolute path now; sub-agents need it.
```

with:

```markdown
3. Read the Criteria file. Resolve its absolute path now; sub-agents need it. If the profile names a Finding contract file (for example `../../shared/finding-contract.md`), resolve that absolute path too and read it; sub-agents need both.
```

- [ ] **Step 3: Neutralize Map “Load” bullet**

Replace:

```markdown
- Load: where each entry point runs. Admin code runs for one user at a time, frontend code on every visitor page view, background code in cron or WP-CLI.
```

with:

```markdown
- Load: where each entry point runs and how often (for example one-user admin screens, every visitor page view, background workers, CLI, or scheduled jobs). Prefer the profile's Map wording when it is more specific.
```

- [ ] **Step 4: Neutralize Finish security suggestion**

Replace:

```markdown
2. Reply with the `TODO_FILE` path, counts by severity, areas reviewed with no findings, excluded paths, assumptions made, the Tools summary, prior-audit candidates confirmed, corrected and refuted (with reasons), and the coverage gaps the sub-agents reported. If the folder has security-sensitive code, suggest running `/claude-security` on `FOLDER` for deeper analysis.
```

with:

```markdown
2. Reply with the `TODO_FILE` path, counts by severity, areas reviewed with no findings, excluded paths, assumptions made, the Tools summary, prior-audit candidates confirmed, corrected and refuted (with reasons), and the coverage gaps the sub-agents reported. If the folder has security-sensitive code, suggest a deeper security pass with the project's preferred security skill or checklist on `FOLDER`.
```

- [ ] **Step 5: Update Slice and Lens briefs to mention finding contract**

In both the Slice brief and the Lens brief, replace:

```markdown
> Read `<absolute Criteria file path>` and apply its Review areas, What not to flag and Finding contract.
```

(and the Lens variant that only mentions What not to flag and Finding contract) with wording that includes an optional finding-contract path:

Slice brief line:

```markdown
> Read `<absolute Criteria file path>` (and `<absolute Finding contract path>` when the profile names one) and apply the Review areas, What not to flag and Finding contract.
```

Lens brief line:

```markdown
> Read `<absolute Criteria file path>` (and `<absolute Finding contract path>` when the profile names one) and apply the What not to flag and Finding contract.
```

- [ ] **Step 6: Verify WP skills still work with the procedure**

Confirm `prime-review-php` and `prime-review-js` profiles still only name `review-criteria.md` (no Finding contract field), so Prepare step 3's new sentence is a no-op for them.

Run:

```bash
rg -n "Finding contract|Criteria" plugins/prime-skills/skills/prime-review-php/SKILL.md plugins/prime-skills/skills/prime-review-js/SKILL.md
rg -n "WP-CLI|visitor page view" plugins/prime-skills/shared/audit-procedure.md || true
```

Expected: WP skills still point only at their criteria files; procedure no longer hard-codes the old WP-CLI/visitor sentence.

- [ ] **Step 7: Human git reminder**

Do not commit.

---

### Task 3: `prime-review-react` skill

**Files:**
- Create: `plugins/prime-skills/skills/prime-review-react/SKILL.md`
- Create: `plugins/prime-skills/skills/prime-review-react/review-criteria.md`
- Test: structure checklist below

**Interfaces:**
- Consumes: `../../shared/audit-procedure.md`, `../../shared/finding-contract.md`
- Produces: React + Next App Router audit profile and criteria

- [ ] **Step 1: Create the skill directory**

```bash
mkdir -p plugins/prime-skills/skills/prime-review-react
```

- [ ] **Step 2: Write `SKILL.md`**

```markdown
---
name: prime-review-react
description: Use when the user asks for a read-only audit of React or Next.js App Router UI code in one folder, with the findings recorded as Pending tasks in a TODO markdown file.
argument-hint: "<folder> <todo-file>"
disable-model-invocation: true
---

# Prime Review React

Audit one folder of React (and Next.js App Router when present) and record verified, actionable findings in a TODO file. Analysis only. Not for WordPress/`@wordpress/*` plugin UI — use `prime-review-js` for that.

Read `../../shared/audit-procedure.md` (relative to this skill's folder) and follow it with this profile.

## Profile

- **Arguments**: `$ARGUMENTS`
- **Criteria**: `review-criteria.md` in this skill's folder
- **Finding contract**: `../../shared/finding-contract.md` (relative to this skill's folder)
- **Source**: `.js .jsx .mjs .cjs .ts .tsx`
- **Exclusions**: build output (directories from the build config and `.gitignore`), minified files, `.d.ts` files, `node_modules`.
- **Project files**: `package.json`, `next.config.*` when present, `tsconfig.*`, the test config, to learn framework detection (`next` dependency or `app/` App Router), build output directories, the JS/TS test framework, and the lint and type-check scripts.
- **Extra prep**: detect Next.js. Treat the project as Next when `package.json` lists a `next` dependency or an App Router tree exists (`app/` with `layout.*` / `page.*`). When Next is not detected, skip Next-only Map items, the Next.js criteria category, and the Next lens.
- **Tools**: the project's JS/TS lint and type-check scripts.
- **Map**:
  - Entry points and how they mount (bundler entries, root components, route `page`/`layout` files).
  - Module graph: shared hooks, components and utilities.
  - State: where it lives (local, context, external stores) and what writes to it.
  - Client↔server data paths: every fetch/RPC/server-action call, its endpoint or action, and the handler that serves it.
  - When Next is detected: route segments, server vs client components, caching/revalidation touchpoints, server actions and route handlers used by in-scope UI.
- **Extra scope**: API route handlers and server actions called from in-scope files (follow them even if outside `FOLDER` when they are the contract).
- **Cross-slice**: duplication across modules, architecture, dependency direction and bundle composition.
- **Lenses**:
  1. Data/async contract: every request's payload and every response field the code reads, including error shapes; races, double submits, updates after unmount, swallowed rejections, UI left inconsistent after failure.
  2. React correctness: hooks, effect dependencies, stale closures, derived state vs effects, list keys, controlled/uncontrolled, component responsibility.
  3. Next App Router (only when Next detected): server/client boundaries, leaking server-only modules or secrets to the client, caching/revalidation mistakes, unsafe server actions, route-handler auth gaps.
  4. Accessibility and XSS sinks: interactive semantics, labels, focus, live regions; `dangerouslySetInnerHTML` and unsafe URL/`href` construction.
- **Example**:

  > `UserTable.tsx:42` (`UserTable`) refetches on every filter-field change because the effect depends on the whole `filters` object, so changing three fields sends three requests and an older response can overwrite a newer one. Trigger the request from the filter-submit handler instead. Keep the request parameters and initial-load behavior unchanged. Verify with tests for initial load, submit, reset and API failure.
```

- [ ] **Step 3: Write `review-criteria.md`**

```markdown
# Review criteria

Shared by the main agent and every slice sub-agent for Prime Review React.

Also read `../../shared/finding-contract.md` (What not to flag and Finding contract). Use the Category list below for every finding's Category field.

## Category list (section order)

Security, Correctness, React, Next.js, Performance, Accessibility, Simplicity, Tests

Omit the Next.js section in `TODO_FILE` when Extra prep did not detect Next.

## Review areas

Check what applies; skip what doesn't.

**Security**
- XSS sinks: `dangerouslySetInnerHTML`, `innerHTML`, `outerHTML`, `insertAdjacentHTML`, and `href`/`src` built from data (`javascript:` URLs). Trace whether the value can be user- or server-controlled and whether it is escaped or sanitized.
- Secrets or privileged tokens shipped in client bundles or public env vars (`NEXT_PUBLIC_*` and equivalents).
- Authz enforced only in the client UI while a callable API/server action lacks the same check (follow Extra scope handlers).
- Third-party or dynamically loaded scripts without a clear trust boundary.

**Correctness**
- Unhandled promise rejections; request errors swallowed or shown as success.
- Response shapes assumed without guards where the server can return errors or empty data.
- Races: older responses overwriting newer state, double submits, state updates after unmount.
- Missing loading, error or empty states; UI left inconsistent after a failure.
- Assumptions about globals, env, DOM elements or script/module load order.

**React**
- Hook rules; missing or wrong effect dependencies; stale closures.
- Effects that should be derived state or event handlers.
- `key` usage in lists; switching between controlled and uncontrolled.
- Components with several unrelated responsibilities; prop drilling that an existing store already solves.
- Memoization missing where there is a visible cost, or present where it buys nothing.

**Next.js**
- Only when Next is detected. Skip entirely otherwise.
- Server Components importing client-only APIs (or missing `'use client'`) in a way that breaks or silently widens the client bundle.
- Server-only modules, secrets or privileged data reaching Client Components.
- Incorrect caching or revalidation for data that must be fresh (or the reverse: disabling cache with a concrete cost and no need).
- Server actions or route handlers that mutate state without authz/authn or CSRF protections the project relies on.
- `searchParams` / route params trusted without validation before use in queries or redirects.

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
- Reuse: logic the codebase already has as a helper, hook or component (name the path).
- Native: code or a dependency doing what the platform, React, or Next already does.
- YAGNI: abstractions with one implementation, wrappers that only pass calls through, options nobody sets.
- Merge: near-copies that must change together.
- Split: one component or function doing several unrelated jobs. Split by job, never by line count.
- Circular imports; dependencies that point the wrong way between layers.

**Tests**
- Using the existing JS/TS test framework, find untested logic that carries real risk: the findings above, data transforms, async and error paths. Don't ask for tests of trivial presentational components.
- Tests in the audited folder that are broken, skipped or assert nothing.
```

- [ ] **Step 4: Verify skill structure**

Run:

```bash
rg -n "^name:|^disable-model-invocation:|Finding contract:|Extra prep:|Next" plugins/prime-skills/skills/prime-review-react/SKILL.md
rg -n "Category list|WordPress|@wordpress" plugins/prime-skills/skills/prime-review-react/review-criteria.md
test -f plugins/prime-skills/shared/finding-contract.md && echo KERNEL_OK
```

Expected: frontmatter present; Finding contract + Extra prep present; criteria has Category list; no `@wordpress` / WordPress in React criteria; `KERNEL_OK`.

- [ ] **Step 5: Human git reminder**

Do not commit.

---

### Task 4: `prime-review-angular` skill

**Files:**
- Create: `plugins/prime-skills/skills/prime-review-angular/SKILL.md`
- Create: `plugins/prime-skills/skills/prime-review-angular/review-criteria.md`
- Test: structure checklist below

**Interfaces:**
- Consumes: `../../shared/audit-procedure.md`, `../../shared/finding-contract.md`
- Produces: Angular (standalone + NgModules) audit profile and criteria

- [ ] **Step 1: Create the skill directory**

```bash
mkdir -p plugins/prime-skills/skills/prime-review-angular
```

- [ ] **Step 2: Write `SKILL.md`**

```markdown
---
name: prime-review-angular
description: Use when the user asks for a read-only audit of Angular application code in one folder, with the findings recorded as Pending tasks in a TODO markdown file.
argument-hint: "<folder> <todo-file>"
disable-model-invocation: true
---

# Prime Review Angular

Audit one folder of Angular (standalone and NgModules-era patterns) and record verified, actionable findings in a TODO file. Analysis only.

Read `../../shared/audit-procedure.md` (relative to this skill's folder) and follow it with this profile.

## Profile

- **Arguments**: `$ARGUMENTS`
- **Criteria**: `review-criteria.md` in this skill's folder
- **Finding contract**: `../../shared/finding-contract.md` (relative to this skill's folder)
- **Source**: `.ts` and component `.html` templates. Styles (`.css` `.scss` `.sass` `.less`) only when they are the root cause of a verified issue in `FOLDER` — not for style preferences.
- **Exclusions**: build output (directories from the build config and `.gitignore`), `node_modules`, generated files.
- **Project files**: `angular.json` or `project.json`, `package.json`, `tsconfig.*`, lint and test config, to learn the workspace layout, bootstrap style (standalone vs NgModule), and the lint and type-check scripts.
- **Tools**: the project's lint and type-check scripts (and test script discovery for coverage judgment only).
- **Map**:
  - Bootstrap: standalone `bootstrapApplication` and/or `AppModule` / feature NgModules.
  - Routes and lazy-loaded features.
  - DI graph: services, `providedIn`, component/route providers.
  - HTTP and API calls: services, interceptors, endpoints.
  - Forms: reactive and template-driven forms, validators, submit paths.
  - Template↔class bindings for in-scope components.
- **Extra scope**: document assumed API contracts for backends called from in-scope services; do not audit the whole backend unless it is inside `FOLDER`.
- **Cross-slice**: duplication, architecture layers, circular DI, feature vs shared boundaries.
- **Lenses**:
  1. Template security and accessibility: binding sinks, bypass security APIs, roles/labels/focus/keyboard.
  2. Change detection and async correctness: subscription lifecycle, `async` pipe, switchMap vs mergeMap races, ignored HTTP errors.
  3. DI and architecture direction: wrong injector scope, circular providers, feature/shared dependency direction.
  4. Forms and validation round-trip: validators vs submit payload, disabled controls, valueChanges races.
- **Example**:

  > `user-table.component.ts:58` (`UserTableComponent.load`) resubscribes on every filter keystroke without cancelling the prior HTTP call, so an older response can overwrite a newer one. Use `switchMap` from a submit or debounced stream (or cancel the prior subscription) and keep the request parameters and empty-state behavior unchanged. Verify with tests for rapid filter changes and API failure.
```

- [ ] **Step 3: Write `review-criteria.md`**

```markdown
# Review criteria

Shared by the main agent and every slice sub-agent for Prime Review Angular.

Also read `../../shared/finding-contract.md` (What not to flag and Finding contract). Use the Category list below for every finding's Category field.

## Category list (section order)

Security, Correctness, Angular, Templates & forms, Performance, Accessibility, Simplicity, Tests

Cover both standalone and NgModules-era patterns under **Angular**; apply whichever the file uses.

## Review areas

Check what applies; skip what doesn't.

**Security**
- Template sinks: bindings that introduce HTML or URLs from untrusted data; `bypassSecurityTrust*` without a proven trust boundary; `innerHTML` equivalents.
- Open redirects from router or `window` navigation built with untrusted input.
- Auth tokens in `localStorage`/`sessionStorage` where the project already has a safer pattern, or tokens exposed to templates/logs.
- HTTP calls that mutate state without the auth/CSRF pattern the rest of the app uses (interceptors or explicit headers). Client-only checks are never authoritative.

**Correctness**
- Subscriptions without teardown (`unsubscribe`, `takeUntilDestroyed`, `AsyncPipe`, or an equivalent the project standardizes on) where leaks or duplicate handlers are reachable.
- HTTP errors ignored or shown as success; assumed response shapes without guards.
- Racey RxJS (`mergeMap` where `switchMap`/`exhaustMap` is required), stale route or input state.
- Router state assumptions that break on deep link, refresh or lazy load.

**Angular**
- NgModule and standalone mixed in ways that double-provide services or skip intended boundaries.
- DI mistakes: service scoped too wide or too narrow; circular DI; providers on the wrong component/route.
- Change detection pitfalls that cause missed updates or heavy re-entry with a concrete cost in this screen.
- Signals and RxJS mixed incorrectly where the project uses both (for example writing signals from uncleared subscriptions).

**Templates & forms**
- Reactive or template-driven validation gaps on submit paths.
- Disabled controls omitted from payloads when the server still requires the value (or the reverse).
- `valueChanges` / status races that submit stale values.
- Template logic that belongs in the class or a pipe the codebase already has.

**Performance**
- Unnecessary change detection work with a concrete cost on this screen (for example heavy getters in templates called every cycle).
- Unbounded lists rendered without virtualization or pagination where the data volume is real in this feature.
- Duplicate HTTP calls; work that should be shared through an existing service or resolver.

**Accessibility**
- Clickable non-interactive elements; keyboard access; visible focus.
- Inputs without labels; icon buttons without accessible names.
- Dialogs/overlays that don't manage focus; dynamic messages not announced.
- Incorrect or redundant ARIA.

**Simplicity**
- Delete: dead code, unused exports, unused NgModules/routes and dependencies. Search the whole repo before claiming anything is unused.
- Reuse: existing services, pipes, directives or utilities (name the path).
- Native: code or a dependency doing what Angular already provides.
- YAGNI: abstractions with one implementation, wrappers that only pass calls through.
- Merge: near-copies that must change together.
- Split: one component or service doing several unrelated jobs. Split by job, never by line count.
- Circular imports; feature modules/directories depending the wrong way on each other.

**Tests**
- Using the existing Angular test setup, find untested logic that carries real risk: the findings above, forms, HTTP error paths, route guards. Don't ask for tests of trivial presentational templates.
- Tests in the audited folder that are broken, skipped or assert nothing.
```

- [ ] **Step 4: Verify skill structure**

Run:

```bash
rg -n "^name:|Finding contract:|standalone|NgModule" plugins/prime-skills/skills/prime-review-angular/SKILL.md
rg -n "Category list|Templates & forms|WordPress" plugins/prime-skills/skills/prime-review-angular/review-criteria.md
```

Expected: profile mentions standalone and NgModule; Category list includes `Templates & forms`; no WordPress.

- [ ] **Step 5: Human git reminder**

Do not commit.

---

### Task 5: Docs and marketplace copy

**Files:**
- Modify: `README.md`
- Modify: `.claude-plugin/marketplace.json`
- Modify: `.cursor-plugin/marketplace.json`
- Modify: `docs/superpowers/specs/2026-11-13-multi-framework-prime-review-design.md` (status line only)

**Interfaces:**
- Consumes: new skill names and invocation patterns
- Produces: discoverable docs that match installed skills

- [ ] **Step 1: Update README skills table and intro**

Change the opening line from WordPress-only to cover multi-framework reviews, and add rows:

| Skill | Claude Code | Cursor | What it does |
|-------|-------------|--------|--------------|
| Prime Review React | `/prime-skills:prime-review-react <folder> <todo-file>` | `/prime-review-react <folder> <todo-file>` | Read-only audit of React / Next.js App Router UI in one folder. Records verified findings as Pending tasks in a TODO file. |
| Prime Review Angular | `/prime-skills:prime-review-angular <folder> <todo-file>` | `/prime-review-angular <folder> <todo-file>` | Read-only audit of Angular (standalone and NgModules) in one folder. Records verified findings as Pending tasks in a TODO file. |

Keep existing PHP, JS, and WP Core Reference rows. Adjust the sentence under the table to note React/Angular use `shared/finding-contract.md` in addition to the shared procedure; WP PHP/JS still embed their own Finding contract.

Suggested intro replacement:

```markdown
A Claude Code and Cursor plugin marketplace with read-only code-review skills (WordPress PHP/JS today, plus React/Next and Angular).
```

- [ ] **Step 2: Broaden marketplace descriptions**

In both `.claude-plugin/marketplace.json` and `.cursor-plugin/marketplace.json`, change the top-level description from WordPress-only to:

```text
Prime code-review skills for WordPress, React/Next, and Angular
```

Leave individual plugin entries' short descriptions accurate; the `prime-skills` plugin description can stay generic ("Read-only folder audits...") or match README tone — do not claim Laravel until Phase 2 ships it.

- [ ] **Step 3: Mark the design spec status**

In `docs/superpowers/specs/2026-11-13-multi-framework-prime-review-design.md`, set:

```markdown
Status: Approved; Phase 1 implementation plan at `docs/superpowers/plans/2026-11-13-multi-framework-prime-review.md`
```

- [ ] **Step 4: Verify docs consistency**

Run:

```bash
rg -n "prime-review-react|prime-review-angular|finding-contract" README.md
rg -n "React/Next|Angular" .claude-plugin/marketplace.json .cursor-plugin/marketplace.json
rg -n "prime-review-laravel" README.md || true
```

Expected: React/Angular documented; Laravel not listed as shipped in README.

- [ ] **Step 5: Human git reminder**

Do not commit or push.

---

### Task 6: Phase 1 verification gate

**Files:**
- Read-only checks across the plugin tree

**Interfaces:**
- Consumes: all Phase 1 outputs
- Produces: pass/fail checklist before human commit

- [ ] **Step 1: File inventory**

Run:

```bash
ls plugins/prime-skills/shared/finding-contract.md
ls plugins/prime-skills/skills/prime-review-react/SKILL.md plugins/prime-skills/skills/prime-review-react/review-criteria.md
ls plugins/prime-skills/skills/prime-review-angular/SKILL.md plugins/prime-skills/skills/prime-review-angular/review-criteria.md
test ! -d plugins/prime-skills/skills/prime-review-laravel && echo LARAVEL_ABSENT_OK
```

Expected: first three groups exist; `LARAVEL_ABSENT_OK`.

- [ ] **Step 2: WP skills untouched**

Run:

```bash
git diff -- plugins/prime-skills/skills/prime-review-php plugins/prime-skills/skills/prime-review-js
```

Expected: empty diff (no changes).

- [ ] **Step 3: Optional plugin validate**

If Claude Code CLI is available:

```bash
claude plugin validate .
```

Expected: success, or note "CLI unavailable" and rely on Steps 1–2 plus README grep.

- [ ] **Step 4: Stop for human**

Summarize created/modified paths. Do not commit. Do not push. Remind that Phase 2 is `prime-review-laravel` per the design spec.

---

## Phase 2 (out of scope for this plan)

Implement later from the same design spec:

1. Create `plugins/prime-skills/skills/prime-review-laravel/SKILL.md` + `review-criteria.md` using the Laravel profile and categories in the design doc.
2. Document Laravel in README / marketplace.
3. Optional: point WP PHP/JS criteria at `shared/finding-contract.md` without behavior change.

Do not start Phase 2 in the same execution pass as Phase 1 unless the human explicitly asks.
