# JavaScript Layer

## Scope

This document applies only to projects with browser-side JavaScript.

The rules in this document apply only to files under:

`src/webapp/js/**`

This directory contains all client-side application logic.

The web application is a Single Page Application (SPA) implemented exclusively with browser-native JavaScript and ES Modules.

## SPA architecture

The application MUST operate as a SPA.

Navigation between application views MUST NOT require loading separate HTML documents.

JavaScript is responsible for rendering and coordinating views within the application's single HTML entry point.

## Modules

JavaScript MUST be organized into small, clearly scoped ES Modules using native `import` and `export`.

No bundler, transpiler, framework, or TypeScript compilation step may be introduced.

Modules MUST be directly executable by supported browsers.

## Visual modules

Views and visual components MUST use canonical names in `kebab-case`.

Whenever a visual JavaScript module requires specific styles, its CSS counterpart MUST follow the same relative structure and filename.

For example:

`js/views/organization.js`

→ `css/views/organization.css`

`js/components/left-sidebar.js`

→ `css/components/left-sidebar.css`

The JavaScript module MUST use the CSS class prefix established by its corresponding CSS module.

For example, `left-sidebar.js` uses:

`left-sidebar`

`left-sidebar-header`

`left-sidebar-title`

`left-sidebar-content`

Module-specific elements MUST NOT introduce unrelated or generic CSS class names.

## Views and components

Views represent application-level UI states or screens.

Reusable visual elements SHOULD be implemented as components rather than duplicated between views.

Views MAY compose multiple components.

Components SHOULD remain focused on their own UI responsibility and expose a clear interface to their callers.

## DOM rendering

Views and components MUST construct dynamic UI using browser-native DOM APIs.

Use APIs such as:

- `document.createElement()`
- `textContent`
- `append()`
- `appendChild()`
- `setAttribute()`
- `classList`

Application HTML MUST NOT be constructed through string concatenation, template literals containing markup, arrays of HTML fragments, `innerHTML`, or `insertAdjacentHTML`.

User-provided, API-provided, or otherwise dynamic values MUST be assigned through appropriate DOM properties such as `textContent` rather than manually escaped and interpolated into HTML strings.

Views and components SHOULD return or compose DOM nodes rather than serialized HTML strings.

## Backend communication

REST, Socket.IO, and authentication are independent, optional capabilities; none is required solely because a frontend exists.

All REST calls MUST be centralized under:

`src/webapp/js/api/`

API modules MUST be organized by entity.

For example:

`js/api/organization.js`

`js/api/workspace.js`

`js/api/user.js`

Each entity API module MUST expose the operations required to communicate with that entity's REST endpoints.

For example:

`organizationApi.get(id)`

`organizationApi.create(data)`

`organizationApi.update(id, data)`

Views, components, and other application modules MUST NOT perform REST requests directly.

Direct use of `fetch` outside `js/api/` is forbidden.

When multiple parts of the application require the same backend operation, they MUST reuse the corresponding entity API function rather than implementing duplicate requests.

API modules are responsible for backend communication. When Socket.IO is used, `js/api/socket.mjs` owns the shared Socket.IO client and session/connection lifecycle. When authentication is required, Socket.IO MUST reuse the project's existing authenticated session. API modules MUST NOT contain UI logic or reproduce backend business logic.

## Styling

JavaScript MUST NOT contain application styling that belongs in CSS.

JavaScript MAY add, remove, or toggle classes and attributes to represent UI state.

Visual modules MUST consume their corresponding CSS modules and the shared application design system rather than defining independent visual conventions.

## Responsive behavior

Views and components MUST be designed to work across relevant viewport sizes.

JavaScript MUST NOT assume a fixed desktop viewport.

Responsive presentation SHOULD be handled through CSS whenever possible. JavaScript SHOULD only participate when responsive behavior requires application logic that CSS cannot reasonably provide.

## Module boundaries

Modules MUST have clear responsibilities.

Shared behavior SHOULD be extracted into appropriately scoped modules rather than duplicated.

A module MUST NOT depend on another module's private implementation details.

Views and components MUST interact through explicit module interfaces rather than manipulating each other's internal state or DOM structure directly.

## Testing

Client-side flows MUST be covered by functional tests under `test/functional/` as required by the repository root `AGENTS.md`.

Relevant user interactions, navigation, rendering, REST integration, and observable UI behavior MUST be tested through the real application flow.

All global rules from the repository root `AGENTS.md` remain applicable.
