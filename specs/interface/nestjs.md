---
id: nestjs
layer: interface
extends: []
---

# NestJS

## Purpose

NestJS encodes an opinionated module / controller / provider architecture through decorators and dependency injection. Its payoff — testable handlers, uniform validation, consistent error envelopes, generated OpenAPI, swappable implementations — depends entirely on using those primitives as intended. When controllers embed business logic, services are instantiated with `new`, auth checks hide inside route handlers, DTOs become plain interfaces, or configuration is read directly from `process.env`, the framework's guarantees silently evaporate: validation bypasses `ValidationPipe`, OpenAPI drifts from the real surface, tests spin up the whole HTTP stack instead of a DI container, and "swap the repository" becomes a refactor rather than a token change. This spec pins the idiomatic Nest patterns so the framework's invocation-layer affordances remain load-bearing.

## References

- **spec** `typescript` — NestJS projects conform to the general TypeScript language rules
- **external** `https://nestjs.com/` — NestJS home
- **external** `https://docs.nestjs.com/` — NestJS documentation
- **external** `https://github.com/nestjs/nest` — NestJS source repository
- **external** `https://github.com/typestack/class-validator` — class-validator rules used in DTOs
- **external** `https://docs.nestjs.com/openapi/introduction` — `@nestjs/swagger` documentation

## Rules

1. Organize code into feature modules with `@Module()`; each feature module declares its own controllers and providers and exports only what other modules need.
2. Keep business logic in `@Injectable()` services; controllers translate transport (HTTP, gRPC, Kafka, etc.) to service calls and back, without inlining domain logic.
3. Inject dependencies via constructor parameters on `@Injectable()`, `@Controller()`, and equivalent classes; do not instantiate providers with `new` in application code.
4. Use class-based providers as DI tokens by default; reach for string or symbol tokens only when an interface boundary (testing, multiple implementations) requires them.
5. Validate every request body, query parameter, and path parameter with DTO classes decorated by class-validator.
6. Register a global `ValidationPipe` configured with `whitelist: true`, `forbidNonWhitelisted: true`, and `transform: true`.
7. Declare DTOs as classes, not interfaces, so runtime validation decorators and `@nestjs/swagger` metadata are preserved.
8. Signal errors by throwing `HttpException` subclasses (`BadRequestException`, `NotFoundException`, `UnauthorizedException`, etc.) or a domain exception handled by a registered filter; do not construct error response bodies by hand inside controllers.
9. Register a global `ExceptionFilter` to shape the outgoing error envelope; do not duplicate error-shape logic across controllers.
10. Implement authorization decisions in `Guards`; do not inline user, role, or scope checks inside controller method bodies.
11. Implement cross-cutting response concerns (timing, logging, envelope wrapping, caching) with `Interceptors`; do not thread them through individual controller methods.
12. Parse and coerce primitive path / query parameters with built-in pipes (`ParseIntPipe`, `ParseUUIDPipe`, `ParseBoolPipe`, etc.); do not cast strings inside controller methods.
13. Read configuration through a typed `ConfigService` registered via `ConfigModule`; do not read `process.env` directly in controllers or services.
14. Perform provider setup in `OnModuleInit` / `OnApplicationBootstrap` and teardown in `OnModuleDestroy` / `OnApplicationShutdown`; do not perform I/O inside constructors.
15. Write unit tests by building a testing module with `Test.createTestingModule()` and providing mocks via DI; do not instantiate `@Injectable()` classes with `new` in test code.
16. Emit operational output through the Nest `Logger` (or a globally-registered logger implementation); do not use `console.log` for operational messages in application code.
17. Generate OpenAPI schema from `@nestjs/swagger` decorators on DTOs and controllers; do not maintain a parallel hand-written OpenAPI document for the same surface.
18. Name source files by their Nest role with the canonical suffix: `*.module.ts`, `*.controller.ts`, `*.service.ts`, `*.dto.ts`, `*.guard.ts`, `*.interceptor.ts`, `*.pipe.ts`, `*.filter.ts`.
19. Keep one HTTP resource per controller; anchor the controller at the resource root (e.g. `@Controller('users')`) and use sub-paths for actions.
20. Enable `"experimentalDecorators": true` and `"emitDecoratorMetadata": true` in `tsconfig.json` for every package that uses Nest decorators.
21. Select a single primary transport per application entry point; if a process must expose multiple transports (HTTP + microservice), wire each on its own adapter with explicit module boundaries.
