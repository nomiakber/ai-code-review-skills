# Review criteria

Shared by the main agent and every slice sub-agent for Prime Review Angular.

Also read `finding-contract.md` in this stack folder (What not to flag and Finding contract). Use the Category list below for every finding's Category field.

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
