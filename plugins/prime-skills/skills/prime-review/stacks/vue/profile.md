# Profile — vue

Not for WordPress/`@wordpress/*` plugin UI — use stack `js` for that.

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
