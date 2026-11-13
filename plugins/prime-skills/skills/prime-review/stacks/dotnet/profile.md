# Profile — dotnet

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
