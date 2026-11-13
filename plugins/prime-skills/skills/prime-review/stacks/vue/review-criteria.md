# Review criteria

Shared by the main agent and every slice sub-agent for Prime Review Vue.

Also read `finding-contract.md` in this stack folder (What not to flag and Finding contract). Use the Category list below for every finding's Category field.

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
