# Persistence Layer

## Scope

The rules in this document apply only to files under:

`server/persistence/**`

They define the architecture and implementation rules for the persistence layer only.

## Responsibility

Persistence has exactly one responsibility: perform database operations according to the entity's canonical storage schema.

Each persistence module MUST:

1. Receive the data required for the persistence operation.

2. Perform the corresponding database operation.

3. Return the complete persistence result produced by the database abstraction used by the application.

Persistence MUST NOT:

- Contain business logic.

- Perform authorization or access-control decisions.

- Make business decisions.

- Coordinate operations between entities.

- Transform, enrich, derive, or otherwise modify application data except for persistence metadata explicitly owned by this layer.

- Perform work that belongs to `endpoint`, `handler`, or other application layers.

- Expose database clients, schemas, adapters, queries, or internal implementation details.

All application runtime database access MUST remain inside the persistence layer.

## Module and Entity Structure

Each persistence module MUST represent exactly ONE persisted entity.

The persistence module filename identifies the entity it owns.

For example:

    server/persistence/profile.js

    → entity: profile

    server/persistence/provider_account.js

    → entity: provider_account

Each persistence module owns all database operations for its entity.

For example:

`persistence/profile` MAY create, read, update, delete, and query profiles.

It MUST NOT directly operate on `provider_account`, `organization`, or any other entity.

Database clients, query builders, adapters, or equivalent infrastructure used by persistence MUST remain private implementation details.

Persistence modules MUST NOT expose the underlying database technology to handlers.

## Storage Schema

Each persisted entity MUST have exactly ONE canonical storage schema defining the valid application-level shape of that entity when persisted.

Storage schemas live under:

`server/schema/persistence/<entity>.json`

The schema tree MUST mirror the persistence tree.

For example:

    server/persistence/profile.js

    → server/schema/persistence/profile.json

    server/persistence/provider_account.js

    → server/schema/persistence/provider_account.json

The storage schema represents the canonical persisted entity contract.

It MUST:

- Describe the complete valid persisted record.

- Define required and optional properties.

- Define property types.

- Preserve applicable constraints such as `enum`, `minLength`, `maxLength`, `format`, and nullability.

- Reject properties that are not part of the persisted entity contract.

- Include persistence-owned fields such as identifiers and timestamps when those fields are part of the stored record.

There MUST NOT be separate persistence schemas for `create`, `read`, `update`, `delete`, or list operations.

There is exactly ONE canonical storage schema per persisted entity.

Operation-specific HTTP/request schemas belong to the endpoint schema layer and MUST NOT replace the persistence storage schema.

The physical database schema and the canonical storage schema MUST remain compatible.

If the database representation intentionally differs from the canonical entity shape, that mapping belongs exclusively to persistence and MUST NOT leak into handlers or endpoints.

Storage schemas MUST NOT be exported from persistence modules.

## Storage Validation

Every persistence write operation MUST validate the complete entity before writing it to the database.

Validation MUST use the shared schema validation helper:

`server/lib/schema`

Each persistence module MUST load exactly ONE canonical schema:

`server/schema/persistence/<entity>.json`

For example:

```js
var schemaHandler = require("../lib/schema");

var schema = require("../schema/persistence/entity.json");
```

The same schema MUST be used for ALL write operations on that entity, including `create()` and `update()`.

Before writing:

```js
schemaHandler.validate(schema, data);
```

Rules:

- Persistence MUST use `schemaHandler.validate(schema, data)` for structural storage validation.

- `schemaHandler.validate(schema, data)` validates the complete object supplied as `data` against the canonical persistence schema.

- Validation MUST happen before the database write.

- The complete final entity being persisted MUST be validated.

- `create()` and `update()` MUST use the same canonical entity schema.

- There MUST NOT be operation-specific persistence schemas.

- Validation logic MUST NOT be implemented manually inside persistence modules.

- The schema MUST be loaded exactly once at module initialization and reused by all write operations.

- Structural validation belongs to the persistence schema.

- Business validation belongs to handlers.

## Database Schema

The database itself MUST enforce integrity constraints that naturally belong to storage.

Where supported and applicable, this includes:

- Primary keys.

- Foreign keys.

- Unique constraints.

- `NOT NULL` constraints.

- Appropriate database types.

- Referential integrity.

- Indexes required for persistence access patterns.

Application storage schemas MUST NOT be treated as a replacement for database integrity constraints.

The database schema and application storage schemas serve complementary purposes:

`database schema` → physical integrity and relationships

`persistence storage schema` → canonical application-level persisted shape

Changes to either MUST preserve compatibility with the other.

### Database Behavior

The database MUST remain a persistence and integrity mechanism.

The database MUST NOT use:

- `ON DELETE CASCADE` or `ON UPDATE CASCADE`.
- Triggers implementing application behavior.
- Stored procedures or stored functions implementing application behavior.
- Database-side RPC for application operations.
- Automatic creation, deletion, propagation, synchronization, or mutation of related entities.

Entity lifecycle and cross-entity behavior MUST be explicitly controlled by the application.

Database transactions are allowed when required for atomicity. All database-specific transaction mechanics MUST remain inside the persistence layer or its private database infrastructure.

The database MAY enforce structural integrity through primary keys, foreign keys, unique constraints, `NOT NULL`, appropriate types, indexes, and other non-behavioral integrity constraints.

## Persistence Metadata

Persistence MAY establish or update metadata that belongs specifically to storage lifecycle management.

For example:

- `id`

- `createdAt`

- `updatedAt`

Timestamp ownership follows these rules unless the database owns them automatically:

- `create()` MUST establish `createdAt` and `updatedAt`.

- `update()` MUST establish a new `updatedAt`.

If identifiers or timestamps are generated by the database, persistence MUST return the generated values as part of the persisted result.

Persistence MUST NOT modify application data beyond persistence-owned metadata or explicit database representation mapping.

## Persistence Results

Persistence functions MUST return the complete meaningful result produced by the database operation.

Persistence MUST NOT discard information merely to simplify the handler.

Where applicable, results may include:

- Persisted record data.

- Generated identifiers.

- Affected-row information.

- Database operation metadata required by the handler.

Interpretation of persistence results belongs to the handler.

Persistence MUST NOT interpret a database result to make a business decision.

Database-specific result structures SHOULD NOT leak beyond persistence when doing so would couple handlers to the selected database technology.

## Queries and Filtering

Persistence MAY implement entity-specific queries required by handlers.

For example:

- Read by ID.

- List by `profile_id`.

- Find by external identifier.

- Check existence.

- Paginated entity queries.

Query construction belongs to persistence.

The decision about WHY a query is performed belongs to the handler.

Persistence MUST NOT introduce business filtering or authorization filtering unless the filtering itself is explicitly part of the persistence contract supplied by the handler.

## Transactions

Persistence MAY participate in database transactions when required.

The persistence layer owns database-specific transaction mechanics.

Business decisions about which operations must succeed atomically belong to handlers.

Cross-entity transactions MUST NOT be implemented by bypassing entity boundaries.

If a business operation requires atomic changes across multiple entities, the implementation MUST preserve the handler architecture while keeping database-specific transaction details inside the persistence infrastructure.

## Entity Boundaries

A persistence module MUST only be called by the main handler corresponding to its own entity.

For example:

`handler/profile` → `persistence/profile` — allowed

`handler/provider_account` → `persistence/profile` — forbidden

`endpoint/profile` → `persistence/profile` — forbidden

`handler/lib/profile/*` → `persistence/profile` — forbidden

Persistence modules MUST NOT be used directly by endpoints, other entities' handlers, or handler auxiliary libraries.

Any cross-entity operation MUST be coordinated through the corresponding main handlers.

## Layer Boundaries

Persistence MUST remain a pure database-access layer.

The standard flow is:

`endpoint/entity → handler/entity → persistence/entity`

Cross-entity application coordination follows:

`handler/entityA → handler/entityB`

Never:

`handler/entityA → persistence/entityB`

Any operation beyond database access, persistence-owned metadata, structural storage validation, and database representation mapping belongs outside this layer.

## Testing

Persistence behavior MUST be covered by persistence/integration tests where applicable.

Tests SHOULD verify:

- Stored records conform to the canonical storage schema.

- Database constraints are respected.

- CRUD operations behave correctly.

- Entity-specific queries behave correctly.

- Generated identifiers and metadata are returned correctly.

- Persistence does not silently discard database results required by handlers.

Business behavior and authorization MUST NOT be tested as persistence responsibilities.

All global rules from the repository root `AGENTS.md` remain applicable.
