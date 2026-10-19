# Handler Layer

## Scope

This document applies only to projects with a business-logic handler layer.

The rules in this document apply only to files under:

`server/handler/**`

They define the architecture and implementation rules for the business-logic layer only.

## Responsibility

Handlers sit between `endpoint` and `persistence` and are responsible for orchestrating complete business operations.

Each main handler module MUST represent exactly ONE entity.

Main handlers are the only interface through which cross-entity application operations may be coordinated.

Each main handler MUST:

1. Receive the data and user context required for the operation from its corresponding endpoint or another main handler.

   When Socket.IO is used, the shared `handler/socket.js` also receives plain data from `server/lib/socket.js`, including trusted authenticated identity when authentication is required. It owns authorization/business rules for the project's supported clients; Socket.IO objects, room adapters, transport timers, and acknowledgement callbacks remain in infrastructure.

2. Perform and orchestrate all business logic required by the operation.

3. Perform business authorization and access-control decisions when required.

4. Apply required business transformations, calculations, relationships, and decisions.

5. Coordinate with other entities exclusively through their main handlers.

6. Call its own entity's persistence module when persistence operations are required.

7. Consume and interpret persistence results as required by the business operation.

8. Return the appropriate result to its caller.

## Entity Structure

Each main handler module MUST represent exactly ONE entity.

For example:

    server/handler/organization.js

    → entity: organization

    server/handler/organization_membership.js

    → entity: organization_membership

A main handler MAY access:

- Its own entity's persistence module.

- Its own private modules under `handler/lib/<entity>/`.

- Other entities exclusively through their main handlers.

## Entity Boundaries

Entity boundaries MUST be strictly preserved.

Cross-entity communication MUST occur exclusively between main handlers.

For example:

`handler/organization` → `handler/workspace` — allowed

`handler/organization` → `handler/user` — allowed

`handler/organization` → `persistence/organization` — allowed

`handler/organization` → `persistence/workspace` — forbidden

`handler/organization` → `handler/lib/workspace/*` — forbidden

`handler/organization` → `handler/lib/user/*` — forbidden

A main handler MUST NOT access another entity's persistence layer directly.

A main handler MUST NOT import, call, or otherwise depend on another entity's `handler/lib/<entity>/` modules.

Cross-entity operations MUST always enter through the target entity's main handler.

The required cross-entity flow is:

`handler/entityA → handler/entityB`

Never:

`handler/entityA → persistence/entityB`

## Persistence Results

Persistence returns the complete meaningful result through the established application-facing contract, including all application-relevant data and metadata. Database-specific driver structures MUST NOT leak into handlers.

The main handler is responsible for consuming and interpreting that result.

The handler MAY extract data, inspect persistence metadata, transform the result as required by the business operation, and decide what result is returned to its caller.

Business filtering or interpretation of persistence results MUST NOT be pushed down into persistence merely to simplify the handler.

## Clean Handlers

Main handler files MUST remain clean, concise, and readable.

A main handler MUST express the business operation at a high level so its execution flow can be understood without reading implementation details.

A business operation may involve, for example:

1. Performing required business authorization.

2. Reading or operating on related entities through their main handlers.

3. Applying business rules or transformations.

4. Persisting state through the entity's own persistence module.

5. Returning the appropriate result.

This sequence is illustrative, not mandatory. The actual order MUST follow the requirements of the business operation.

Complex implementation details MUST NOT obscure the orchestration flow.

## Handler Libraries

Supporting logic that would obscure the main handler's business flow MUST be extracted under:

`server/handler/lib/<entity>/`

For example:

`server/handler/lib/organization/`

These modules are private implementation details of their owning main handler.

They MAY contain auxiliary logic required by that main handler.

They MUST NOT:

- Be imported or called by endpoints.

- Be imported or called by other entities.

- Coordinate operations with other entities.

- Import or call other entities' main handlers.

- Access persistence directly.

Only the owning main handler may use its `handler/lib/<entity>/` modules.

The main handler SHOULD call clearly named auxiliary functions rather than embedding complex implementation details inline.

Trivial logic MUST NOT be extracted merely to reduce line count. Extraction exists to preserve clarity and readability of the business flow.

## Layer Boundaries

Handlers MUST NOT receive or manipulate Express `req` or `res` objects.

Endpoints MUST extract and pass only the data and user context required by the handler.

Handlers MUST NOT import or use the database SDK directly.

All database access MUST go through persistence.

Only a main handler may access its own entity's persistence module.

The standard application flow is:

`endpoint/entity → handler/entity → persistence/entity`

Cross-entity coordination follows:

`handler/entityA → handler/entityB`

Handlers contain business logic.

Endpoints and persistence modules MUST NOT contain business logic.

## Update Operations

`server/lib/merge.js`

Update operations MUST preserve existing entity data unless a property is explicitly modified or removed.

Handlers performing partial updates MUST use `merge.update(existing, body)` when the required behavior is deep merge plus top-level filtering.

`merge.update(existing, body)` MUST:

- Deep merge the existing entity with the incoming update data.

- Preserve nested existing properties not modified by the update.

- Replace explicitly provided values.

- Remove properties explicitly set to `null` according to the existing `deep()` merge semantics.

- Return only the top-level properties explicitly included in the incoming update body.

Its standard behavior is equivalent to:

```js
const merged = merge.deep(existing || {}, body);
const cleanBody = merge.clean(Object.keys(body), merged);
```

Handlers MUST NOT reimplement this `deep()` plus `clean()` pattern when `merge.update()` provides the required behavior.

Handlers MAY use `merge.deep()` directly when an operation requires deep-merge behavior without the top-level filtering provided by `merge.update()`.

## Testing

Handler business logic MUST be covered by unit tests under:

`test/unit/`

Persistence MUST be mocked when unit-testing handlers, as required by the repository root `AGENTS.md`.

Authorization rules, business decisions, cross-entity coordination, and relevant success and failure paths MUST be tested.

All global rules from the repository root `AGENTS.md` remain applicable.
