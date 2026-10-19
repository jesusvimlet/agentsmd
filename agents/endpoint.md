# Endpoint Layer

## Scope

This document applies only to projects with a REST endpoint layer.

The rules in this document apply only to files under:

`server/endpoint/**`

They define the architecture and implementation rules for the REST endpoint layer only.

## Responsibility

Endpoints MUST remain thin.

Their responsibility is limited to the HTTP interface, request validation, authentication resolution, delegation to handlers, and HTTP response handling.

Each endpoint module MUST represent exactly ONE entity.

Each endpoint operation MUST:

1. Define the HTTP method and path.

2. Use its corresponding external operation schema.

3. Validate the request before calling the handler.

4. Resolve the authenticated user from the request when required.

5. Extract only the request values required by the handler.

6. Delegate the operation to the main handler corresponding to its entity.

7. Return the handler result through the appropriate HTTP response.

Endpoints MUST NOT:

- Contain business logic.

- Perform business authorization or resource access-control decisions.

- Access persistence directly.

- Import or use the Database SDK.

- Communicate directly with another entity's handler.

- Communicate directly with `handler/lib/**`.

- Pass Express `req` or `res` objects to handlers.

## Entity Structure

Each endpoint module MUST represent exactly ONE entity.

For example:

    server/endpoint/organization.js

    → entity: organization

    server/endpoint/organization_invitation.js

    → entity: organization_invitation

An endpoint module MUST communicate exclusively with the main handler corresponding to its own entity.

For example:

`endpoint/organization` → `handler/organization` — allowed

`endpoint/organization` → `handler/workspace` — forbidden

`endpoint/organization` → `handler/lib/organization/*` — forbidden

`endpoint/organization` → `persistence/organization` — forbidden

Any cross-entity operation MUST be coordinated by the endpoint's corresponding main handler.

## Operation Schemas

Each endpoint operation MUST have exactly ONE corresponding external schema.

Endpoint schemas are operation-specific.

There MUST NOT be one shared endpoint schema for the entire entity when the entity exposes multiple operations.

Schemas live under:

`server/schema/endpoint/<entity>/<operation>.json`

For example:

    server/endpoint/organization.js

    POST /organizations

    → server/schema/endpoint/organization/create.json

    GET /organizations/:id

    → server/schema/endpoint/organization/read.json

    PUT /organizations/:id

    → server/schema/endpoint/organization/update.json

    DELETE /organizations/:id

    → server/schema/endpoint/organization/delete.json

When creating or modifying an endpoint operation, the agent MUST verify that its corresponding schema exists. If it does not exist, the agent MUST create it.

Validation schemas MUST NOT be defined inline or hardcoded inside endpoint implementation files.

Each operation schema MUST be loaded exactly once at module initialization and used by its corresponding route.

For example:

```js
const schemas = {
  create: require("../schema/endpoint/entity/create.json"),
  update: require("../schema/endpoint/entity/update.json"),
  read: require("../schema/endpoint/entity/read.json"),
  list: require("../schema/endpoint/entity/list.json"),
  delete: require("../schema/endpoint/entity/delete.json")
};
```

Schema contents and conventions are governed by `agents/schema.md`.

## Request Validation

Every endpoint operation MUST validate the incoming request before calling its handler.

Request validation MUST use the shared schema validation helper:

`server/lib/schema`

For example:

```js
var schemaHandler = require("../lib/schema");

schemaHandler.validate(schemas.create, req);

const data = await entityHandler.create(
  req.user.userId,
  req.body
);
```

Rules:

- Endpoints MUST use `schemaHandler.validate(schema, req)` for structural request validation.

- `schemaHandler.validate(schema, req)` receives the complete request object. The supplied schema determines which request properties are relevant and validated.

- Validation MUST happen before the corresponding handler is called.

- Validation logic MUST NOT be implemented manually inside endpoint files.

- Validation schemas MUST remain external JSON files, one per operation.

- Handlers MAY assume that input received through an endpoint has already passed structural request validation.

- Business validation, authorization, state-dependent rules, and other business decisions belong to handlers, not endpoint schemas.

## Request Handling

Endpoints MUST extract only the values required by the handler.

These values may include:

- `req.body`

- `req.params`

- `req.query`

- Resolved authentication identity or other required authentication context.

For example:

```js
const data = await entityHandler.update(
  req.user.userId,
  req.params.id,
  req.body
);
```

Handlers MUST NOT receive Express `req` or `res` objects.

The endpoint MAY use `res` for HTTP-specific behavior such as setting the response status.

Errors SHOULD be forwarded through the application's HTTP error handling mechanism.

For example:

```js
try {
  schemaHandler.validate(schemas.read, req);

  const data = await entityHandler.read(
    req.user.userId,
    req.params.id
  );

  res.status(200).send(data);
} catch (error) {
  next(error);
}
```

## Authentication and Authorization

Protected endpoints MUST assume authentication has already been performed by the global authentication middleware configured in `server/index.js`.

The authenticated user context is provided through:

```
req.user = {
  userId: ...,
  email: ...
};
```

Endpoints MUST obtain authenticated identity exclusively from `req.user`.

Endpoints MUST NOT:

- Parse or inspect the `Authorization` header.

- Extract authentication tokens.

- Validate or verify authentication tokens.

- Call Supabase Auth or any other authentication provider directly.

- Reimplement authentication logic already handled by the global authentication middleware.

Endpoints MUST extract only the authentication context required by the operation and pass it to the corresponding handler.

For example:

```
const data = await entityHandler.update(
  req.user.userId,
  req.params.id,
  req.body
);
```

Endpoints MUST NOT make business authorization or resource access-control decisions.

Decisions such as whether an authenticated user may read, modify, delete, or otherwise operate on a specific application resource belong exclusively to the handler.

Public versus protected endpoint registration is governed by `agents/server.md` and the authentication boundary established in `server/index.js`.

## HTTP Responses and Errors

Endpoints are responsible for HTTP-specific behavior.

They MAY:

- Set the appropriate HTTP status code.

- Return the handler result.

- Forward errors to the HTTP error handling layer.

- Translate technical endpoint conditions into the appropriate HTTP error when required.

Endpoints MUST NOT interpret handler results to make business decisions.

## Layer Boundaries

The required application flow is:

`endpoint → handler → persistence`

Endpoints MUST NOT bypass the handler layer.

Persistence access and Database data access MUST NOT occur inside endpoint modules.

Cross-entity coordination MUST NOT occur inside endpoint modules.

## Testing

Endpoint behavior MUST be covered by functional tests under:

`test/functional/`

Tests SHOULD cover, where applicable:

- Request schema validation.

- Authentication requirements.

- Correct extraction of request values.

- Delegation to the corresponding entity handler.

- HTTP status and response behavior.

- Invalid request and error paths.

All global rules from the repository root `AGENTS.md` remain applicable.
