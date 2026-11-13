# Profile — react

Not for WordPress/`@wordpress/*` plugin UI — use stack `js` for that.

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
