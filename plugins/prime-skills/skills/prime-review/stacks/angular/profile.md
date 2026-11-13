# Profile — angular

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
