# Schema Architecture

## Scope

The rules in this document apply only to files under:

`server/schema/**`

They define the architecture and organization rules for application schemas.

Schemas define validation contracts.

Schemas MUST NOT contain business logic, authorization decisions, access-control decisions, or application orchestration.

## Schema Types

There are three distinct schema types:

- Endpoint schemas.

- Persistence schemas.

- Socket.IO handshake/event schemas.

These schema types serve different purposes and MUST remain separate.

An endpoint schema MUST NOT be used as a persistence schema.

A persistence schema MUST NOT be used as an endpoint schema.

## Endpoint Schemas

Endpoint schemas live under:

`server/schema/endpoint/<entity>/`

There MUST be one schema per HTTP operation.

For example:

    server/schema/endpoint/organization/create.json

    server/schema/endpoint/organization/read.json

    server/schema/endpoint/organization/update.json

    server/schema/endpoint/organization/delete.json

Each operation schema contains only the schema sections applicable to that endpoint, such as:

- `body`

- `params`

- `query`

- `response`

Endpoint schemas are operation-specific.

Do NOT create one global endpoint schema per entity as a replacement for operation-specific schemas.

Do NOT reuse a schema between different operations merely to reduce the number of schema files.

Endpoint schema usage and loading rules are defined by:

`agents/endpoint.md`

## Persistence Schemas

Persistence schemas live directly under:

`server/schema/persistence/`

There MUST be exactly ONE persistence schema per persisted entity.

The filename MUST correspond to the persisted entity.

For example:

    server/persistence/organization.js

    → server/schema/persistence/organization.json

    server/persistence/organization_membership.js

    → server/schema/persistence/organization_membership.json

A persistence schema defines the complete valid stored document for that entity.

Persistence schemas are entity-specific, NOT operation-specific.

Do NOT create separate persistence schemas for `create`, `read`, `update`, `delete`, or any other persistence operation.

For example, this structure is forbidden:

    server/schema/persistence/organization/create.json

    server/schema/persistence/organization/read.json

    server/schema/persistence/organization/update.json

The correct structure is:

    server/schema/persistence/organization.json

Persistence schema usage and loading rules are defined by:

`agents/persistence.md`

## Schema Ownership

Socket.IO schemas live directly under `server/schema/socket/`, one external contract per handshake role or event. `server/lib/socket.js` loads them once and validates supplied arguments with `schemaHandler.validate(schema, data)` before delegation. Authentication identity and business authorization are not schema responsibilities.

Schemas MUST remain external to application implementation files.

Endpoint schemas MUST NOT be duplicated or hardcoded inside endpoint modules.

Persistence schemas MUST NOT be duplicated or hardcoded inside persistence modules.

Handlers MUST NOT define or own endpoint or persistence schemas.

Schema filenames and paths MUST correspond to the entity and operation they represent.

## Schema Infrastructure

All schema validation MUST use the shared generic validator contract:

`schemaHandler.validate(schema, data)`

The validator receives the complete supplied object as `data`.

The supplied schema determines which properties of that object are validated.

For endpoint validation, the complete request object is supplied:

`schemaHandler.validate(schema, req)`

For persistence validation, the complete entity is supplied:

`schemaHandler.validate(schema, entity)`

The shared validator MUST NOT contain endpoint-specific request extraction logic or assume that `data` is an Express request.

Do NOT introduce `$ref`, shared schema registries, schema composition, global schema registration, or additional schema infrastructure unless explicitly required by the architecture.

Do NOT introduce `server.addSchema()` or equivalent registration mechanisms merely to load external schema files.

External schema files do not imply that schemas must be globally registered.

Schema infrastructure MUST NOT be added solely for abstraction, reuse, or anticipated future requirements.

## Layer Ownership

This document defines how schemas are organized and what they represent.

How schemas are loaded and consumed by application code is governed by the corresponding layer-specific instructions:

`agents/endpoint.md`

`agents/persistence.md`

All global rules from the repository root `AGENTS.md` remain applicable.
