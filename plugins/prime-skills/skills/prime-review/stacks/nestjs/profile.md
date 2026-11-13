# Profile — nestjs

- **Source**: `.ts .js`
- **Exclusions**: `dist/`, `node_modules`, generated clients and build output from the project config / `.gitignore`.
- **Project files**: `package.json`, `nest-cli.json`, `tsconfig.*`, the test config, to learn Nest major, optional microservices/GraphQL packages, and the lint, type-check and test scripts.
- **Extra prep**: detect optional layers independently.
  - Microservices: enable when `package.json` lists `@nestjs/microservices` and/or in-scope code uses microservice transports, `@MessagePattern`, or `@EventPattern`. Otherwise skip microservices-only Map items, the Microservices criteria category, and the microservices lens.
  - GraphQL: enable when `package.json` lists `@nestjs/graphql` (or Nest GraphQL driver packages such as Apollo/Mercurius integrations used by the project) and/or in-scope code defines resolvers / `@Query` / `@Mutation`. Otherwise skip GraphQL-only Map items, the GraphQL criteria category, and the GraphQL lens.
- **Tools**: the project's lint, type-check, and test scripts.
- **Map**:
  - Root / `AppModule` and feature module graph (imports, exports, providers).
  - Controllers and route handlers in scope.
  - Providers/services and their injected dependencies.
  - Global and route pipes, guards, interceptors, and filters that apply to in-scope handlers.
  - DTO / `ValidationPipe` usage and response serialization.
  - When microservices detected: message/event patterns and transport config used by in-scope handlers.
  - When GraphQL detected: resolvers, schema or code-first models, and GraphQL-specific guards/interceptors used in scope.
- **Extra scope**: shared modules, guards, pipes, and filters imported by in-scope code; document assumed external HTTP/RPC contracts without auditing unrelated services end-to-end.
- **Cross-slice**: duplication, circular module imports, dependency direction, repeated authz gaps across sibling controllers or resolvers.
- **Lenses**:
  1. Authn/authz and guard gaps: public vs protected routes, missing guards, trusting client-supplied identity.
  2. Request → handler → response validation: DTOs, pipes, exception filters, partial writes, wrong status or contract.
  3. DI and module boundaries: provider scope mistakes, circular deps, leaking internals across modules.
  4. Optional layers: when microservices detected, message payload validation, ack/retry/idempotency gaps with concrete loss, trusting network identity without auth; when GraphQL detected, resolver auth gaps, N+1 with concrete cost, field exposure of sensitive data, input validation gaps. Run both halves of this lens when both layers are detected.
- **Example**:

  > `orders.controller.ts:41` (`OrdersController.create`) is `@Public()` while `OrdersService.create` writes `userId` from the body, so any caller can create orders for another user. Require the authenticated user (or an admin guard) and ignore client-supplied `userId`. Leave the success response shape unchanged. Verify with e2e tests for anonymous, wrong-user, and admin paths.
