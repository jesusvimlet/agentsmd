# Server

## Scope

This document applies only to projects with an Express-based HTTP server.

The rules in this document apply to:

`server/**`

They define the global server architecture, initialization, middleware pipeline, route protection, and server-level boundaries.

More specific rules for `endpoint`, `handler`, `persistence`, and `schema` are defined by `agents/endpoint.md`, `agents/handler.md`, `agents/persistence.md`, and `agents/schema.md` and take precedence within their scope.

## Server Entry Point

`server/index.js` is the main application entry point.

It is responsible for:

* Creating and configuring the Express application.

* Installing global middleware.

* Establishing the public/protected REST route boundary when the project has protected REST endpoints.

* Registering endpoint routers when the project exposes REST endpoints.

* Installing the global error middleware.

* Creating and starting the HTTP server.

* Initializing server-level infrastructure required by the current boilerplate.

Business logic MUST NOT be added to `index.js`.

## Middleware Pipeline

Middleware and routes in `index.js` MUST preserve their execution order.

Static serving, REST endpoint registration, and authentication middleware apply only when the corresponding project capabilities exist. The request-protection, sanitization, and rate-limiting requirements below apply to REST endpoints. Applicable stages MUST preserve this relative order:

`express.json → static files → rate limit → sanitize → public endpoints → authentication → protected endpoints → error middleware`

Middleware ordering is part of the server's security architecture and MUST NOT be changed without understanding its effect on every route registered after it.

## Global Request Protection

The following middleware MUST execute before application REST endpoints:

`rateLimitMiddleware()`

`sanitizeMiddleware()`

They provide global request protection and MUST NOT be bypassed by individual endpoints unless explicitly required by the application architecture.

Rate limiting MUST remain before application routes.

Sanitization MUST remain before application routes so handlers and endpoints receive sanitized `req.query`, `req.params`, and `req.body` values.

Endpoints MUST NOT implement replacements for these global protections.

## Authentication Boundary

This section applies to projects with protected REST endpoints.

The position of `authMiddleware` in `index.js` defines the authentication boundary.

Routes registered BEFORE:

`app.use(authMiddleware);`

are outside the global authentication protection.

Routes registered AFTER:

`app.use(authMiddleware);`

are protected by the global authentication middleware when applicable to `/rest/**`.

This ordering is security-critical.

A REST endpoint MUST NOT be registered before `authMiddleware` unless it is intentionally designed to be publicly accessible.

When adding, moving, or modifying route registration, the agent MUST explicitly determine whether the route is public or protected before choosing its position in `index.js`.

Protected REST endpoints MUST remain after `authMiddleware`.

Public endpoints MUST be explicitly placed before `authMiddleware`.

The agent MUST NOT move `authMiddleware`, move routes across the authentication boundary, or change this ordering merely for organization or code cleanup.

## Authentication Context

For protected REST requests, `authMiddleware` MUST validate the supplied authentication credentials using the project's configured authentication provider.

After successful authentication it establishes:

```
req.user = {
  userId: user.id,
  email: ...
};
```

Endpoints MAY consume this authentication context and pass the required identity information to their corresponding handlers.

Endpoints MUST NOT duplicate authentication verification already performed by the global authentication middleware.

Failure to authenticate MUST prevent execution of the protected endpoint.

## Sanitization

The global sanitization middleware processes request:

* `query`

* `params`

* `body`

Sanitized values replace:

* `req.query`

* `req.params`

* `req.body`

The original values are preserved under:

* `req.unsafe.query`

* `req.unsafe.params`

* `req.unsafe.body`

The sanitization middleware also exposes its sanitization functionality through:

* `req.sanitize`

Application code SHOULD operate on the sanitized request values by default.

Code MUST NOT use `req.unsafe` as a way to bypass sanitization unless explicitly required and justified by the operation.

## Rate Limiting

Global rate limiting is applied before application routes.

Production requests MUST remain protected by the configured rate limit.

Development-specific limits MAY differ from production limits.

Individual endpoints MAY introduce additional or stricter rate limiting when explicitly required, but MUST NOT disable or bypass the global rate limiter.

## Error Handling

The global error middleware MUST remain registered after all application routes.

Its position is required so errors propagated by endpoints can reach the centralized error handling layer.

Application routes SHOULD propagate errors through the established Express error flow rather than implementing unrelated HTTP error handling mechanisms.

Known application errors MAY be translated into their corresponding HTTP responses by the global error middleware.

Unexpected errors MUST result in a generic server error response without exposing internal implementation details.

## Route Registration

When the project exposes REST endpoints, endpoint routers are registered from `index.js`.

When authentication is required for REST endpoints, route registration determines whether an endpoint is public or protected based on its position relative to `authMiddleware`.

When adding a new endpoint router, the agent MUST:

1. Determine whether the endpoint is public or protected.

2. Register it on the correct side of the authentication boundary when authentication is required.

3. Preserve global rate limiting and sanitization.

4. Preserve the final error middleware position.

Endpoint implementation rules themselves belong to `agents/endpoint.md`.

## Layer Boundaries

`index.js` is infrastructure and composition code.

It MUST NOT:

* Contain business logic.

* Perform business authorization.

* Access persistence directly.

* Perform database operations.

* Coordinate application entities.

* Reimplement endpoint, handler, persistence, or schema responsibilities.

Application operations MUST continue through the architecture defined by the corresponding layer instruction files under `agents/`.

## Security-Critical Changes

The following changes MUST be treated as security-sensitive:

* Moving `authMiddleware`.

* Registering a protected route before `authMiddleware`.

* Changing authentication middleware behavior.

* Removing or bypassing sanitization.

* Removing or bypassing rate limiting.

* Changing middleware execution order.

* Using `req.unsafe` in application code.

* Moving the global error middleware before application routes.

Agents MUST preserve these protections unless the requested task explicitly requires changing them.

Refactoring, cleanup, formatting, or route reorganization MUST NOT alter the effective protection applied to existing endpoints.

All global rules from the repository root `AGENTS.md` remain applicable.
