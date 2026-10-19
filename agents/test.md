# Testing Architecture

## Scope

This document applies only to projects with automated application tests.

The rules in this document apply only to files under:

`test/**`

They define the architecture and implementation rules for application tests.

Tests are divided into:

- Functional tests under `test/functional/`.

- Unit tests under `test/unit/`.

## Functional Tests

The following running-application requirements apply to functional tests of the project's HTTP or Socket.IO interfaces; they do not require either interface to exist.

Functional tests MUST test the real running application through the corresponding public interface.

Socket.IO functional tests exercise the same running application's public Socket.IO interface, using the real client protocol. When authentication is required, tests MUST use real authenticated sessions. Real fixture users/sessions may be provisioned through the configured authentication provider's public authentication interface and must be tracked/removed; application authentication is never mocked. Ephemeral presence requires no domain database fixtures.

HTTP functional tests MUST exercise the real application flow when those layers participate in the tested operation:

`HTTP request → endpoint → handler → persistence → HTTP response`

Functional tests MUST NOT:

- Create their own server.

- Start or stop the application server.

- Import or configure the application server.

- Import `server/index.js` to construct a test server.

- Import or use `createServer`.

- Invoke endpoints, handlers, or persistence modules directly.

- Build a parallel application specifically for testing.

- Mock endpoints, handlers, persistence, authentication, or database.

The application MUST already be running before HTTP or Socket.IO functional tests execute.

## Application URL

The following URL requirements apply to functional tests connecting to an HTTP or Socket.IO application interface.

Functional tests MUST connect to the running application using the existing application environment configuration.

The application URL MUST be derived from the same runtime configuration used by the application.

Tests MUST NOT hardcode:

- `localhost`

- `0.0.0.0`

- A port.

- An alternative application URL.

Tests MUST NOT introduce testing-specific URL configuration such as `TEST_BASE_URL` or equivalent unless explicitly required by an architectural change.

Tests MUST NOT introduce test-specific environment files or environment variables.

## Test Ownership

Functional tests MUST be organized according to the functionality or entity they test.

For example:

`test/functional/auth.test.js` → authentication behavior

`test/functional/organization.test.js` → organization API behavior

`test/functional/test.test.js` → test entity API behavior

Tests for one entity or application area MUST NOT be placed in another entity's functional test merely because the operations interact.

Cross-entity behavior SHOULD be tested from the API operation whose behavior is actually being verified.

## Functional Test Behavior

Functional tests MUST verify externally observable application behavior.

Depending on the operation, this may include:

- Request input.

- HTTP status.

- Response data.

- Authentication state.

- Cookies or sessions.

- Relevant error responses.

Tests MUST NOT depend on internal implementation details when the same behavior can be verified through the public API.

Success paths and relevant failure paths SHOULD be covered according to the actual contract of each operation.

Tests MUST NOT invent generic failure cases merely to enforce identical test structures across endpoints.

## Test Data

Functional tests MAY create real persistent data when required to exercise the application.

Test data MUST be isolated from existing application data.

When a test controls an ID or another suitable unique identifier, it MUST use the `test:` prefix when the application's existing format permits it.

For example:

`test:<unique-value>`

Other test-controlled identifiers, such as usernames, SHOULD use the same `test:` prefix when their existing format permits it.

Identifiers MUST be unique enough to prevent collisions between separate test runs.

The production application, persistence schemas, or business model MUST NOT be modified solely to support test-specific identifiers or metadata.

Test-only properties such as `_test` MUST NOT be added to persisted application entities solely for testing purposes.

## Cleanup

Functional tests MUST NOT leave persistent test data behind.

Every persistent resource created by a test execution MUST be tracked when cleanup is possible.

Cleanup MUST remove only resources created by that specific test execution.

Tests MUST NOT delete, reset, modify, or otherwise clean pre-existing application data.

When the application exposes a public operation capable of removing the created resource, cleanup MUST use that public application interface rather than bypassing the application through handlers, persistence, or direct database access.

Cleanup failures MUST NOT be silently ignored.

Empty error handlers such as:

`catch (err) {}`

MUST NOT be used to hide cleanup failures.

A cleanup failure MUST remain observable so persistent test-data contamination can be detected and corrected.

Test-data prefixes and unique identifiers are safeguards only. They do NOT replace proper cleanup.

## Browser and Authentication Testing

Puppeteer MAY be used when testing behavior that requires a real browser context.

This includes behavior involving:

- Cookies.

- Sessions.

- Authentication state.

- CSRF protection.

- Other browser-managed state.

Browser-based tests MUST interact with the application through its public HTTP or Socket.IO interfaces, as used by the tested flow, and MUST NOT bypass the public application interface.

## Test Helpers

Shared or local helpers MAY be used to reduce mechanical duplication.

For example, helpers may wrap HTTP requests or JSON response parsing.

Test helpers MUST NOT:

- Contain business logic.

- Reimplement application behavior.

- Alter application responses.

- Bypass application layers.

- Hide failures that are relevant to the tested behavior.

Helpers exist only to simplify test mechanics.

## Timeouts

Test suites MAY define reasonable timeouts appropriate for real application and infrastructure operations.

Timeouts MUST NOT be increased merely to hide application, database, networking, or infrastructure failures.

When an operation unexpectedly exceeds the established timeout, the underlying cause SHOULD be investigated before increasing it.

## Unit Tests

Unit tests live under:

`test/unit/`

Handler unit tests MUST test business logic in isolation.

Persistence and other external dependencies MUST be mocked when required to isolate the handler behavior.

Unit tests MUST NOT:

- Start the application server.

- Use Puppeteer.

- Depend on the running application.

- Depend on real database infrastructure.

- Become functional tests by exercising the complete application stack.

HTTP and Socket.IO functional tests verify application behavior through their corresponding public interfaces.

Unit tests verify isolated business logic.

## Testing Infrastructure

Tests MUST use the application's existing architecture and environment.

Agents MUST NOT introduce testing-specific infrastructure by default, including:

- Alternative application servers.

- Test-only server instances.

- Alternative databases.

- Parallel application initialization.

- Test-specific base URLs.

- New environment variables.

- Test-specific application architecture.

If the existing environment prevents a test from running correctly, the cause MUST be investigated rather than bypassed by constructing an alternative testing architecture.

All global rules from the repository root `AGENTS.md` and applicable server-layer instruction files under `agents/` remain applicable.
