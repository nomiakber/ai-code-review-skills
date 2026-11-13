# Profile — laravel

Not for WordPress plugins or themes — use stack `php` for that.

- **Source**: `.php` and `.blade.php`
- **Exclusions**: `vendor/`, generated files, language dumps, build output, `node_modules`.
- **Project files**: `composer.json`, `artisan`, relevant `config/*`, PHPUnit or Pest config, Pint / PHPStan / Larastan configs, to learn the minimum PHP and Laravel versions, autoloading, test setup, and the lint and static-analysis commands.
- **Tools**: the project's Pint (check only, never `--fix`), PHPStan / Larastan, and related static-analysis commands when present.
- **Map**:
  - Routes → controllers, invokable actions, or closures (web, API, console).
  - Middleware on those routes (auth, guest, throttle, custom).
  - Form Requests and inline validation.
  - Eloquent models, scopes, relations, and `$fillable` / `$guarded` / casts.
  - Jobs, events, listeners, notifications, and scheduled tasks touched by in-scope code.
  - Policies, gates, and authorization calls.
  - Blade (or other view) entry points rendered from in-scope actions.
  - Invariants: rules enforced at one write route, with their sibling routes.
- **Extra scope**: frontend callers of in-scope API routes when they are contract consumers (document the assumed contract; follow them when the defect is in the Laravel handler).
- **Cross-slice**: duplication across classes, service boundaries, dependency direction, repeated authz gaps across sibling routes, dual write paths, data written in one place and read elsewhere with a different shape or meaning.
- **Lenses**:
  1. Authn/authz and mass assignment: middleware, policies/gates, `$fillable`/`$guarded`, forceFill / unguarded updates.
  2. Request → DB → response round trip: hostile values through validation, casts, storage, API/Blade output; early returns and partial writes.
  3. Queues/jobs and failure modes: retries, idempotency, lost updates, notifications only on some paths.
  4. N+1 and query cost: relation loading, unbounded `get()`, missing chunking on large jobs or exports.
- **Example**:

  > `app/Http/Controllers/OrderController.php:88` (`OrderController::store`) mass-assigns `$request->all()` into `Order::create`, so a client can set `status` or `user_id` and approve their own order. Validate with a Form Request (or explicit `$request->validated()` / only allowed keys) and keep `$fillable` tight. Leave the success response shape unchanged. Verify with a feature test that extra attributes are ignored and unauthorized status changes fail.
