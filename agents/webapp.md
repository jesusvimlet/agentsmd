# Web Application

## Scope

The rules in this document apply only to files under:

`src/webapp/**`

This directory contains the complete browser application.

The webapp MUST follow the frontend stack requirements defined by the repository root `AGENTS.md`.

## Architecture

The web application MUST be implemented as a Single Page Application (SPA).

The application uses:

- Vanilla HTML.

- Vanilla JavaScript with native ES Modules.

- Vanilla CSS.

Frameworks, TypeScript, CSS preprocessors, bundlers, and other compilation or bundling layers MUST NOT be introduced.

The application MUST run directly in the browser from the source files served by the backend.

## Structure

Frontend code is organized under:

`src/webapp/html/`

`src/webapp/js/`

`src/webapp/css/`

Each area MUST preserve a clear responsibility:

- `html/` contains HTML structure belonging to the SPA.

- `js/` contains application behavior, views, components, API communication, and client-side orchestration.

- `css/` contains all application styling.

Specific JavaScript and CSS conventions are governed by `agents/webapp-js.md` and `agents/webapp-css.md`, respectively.

## SPA

The application MUST behave as a SPA.

Navigation and view changes MUST occur without loading separate application documents.

JavaScript is responsible for rendering and coordinating application views within the SPA.

Views and visual components SHOULD remain modular rather than accumulating application behavior into a single large file.

## Visual module consistency

Visual modules MUST maintain consistent naming and organization across JavaScript and CSS.

When a JavaScript view or component requires specific styling, its JavaScript and CSS modules MUST represent the same visual concept and SHOULD mirror the same relative structure.

For example:

`js/views/organization.js`

→ `css/views/organization.css`

`js/components/left-sidebar.js`

→ `css/components/left-sidebar.css`

Detailed CSS naming and prefix conventions are defined by `agents/webapp-css.md`.

## Backend communication

The webapp MUST communicate with the backend through REST endpoints and the authenticated Socket.IO interface.

REST communication MUST be centralized through the API modules defined under `js/api/`.

Socket.IO communication MUST be centralized in `js/api/socket.mjs`, using the existing Supabase session. Views consume application state and MUST NOT open sockets.

Views, components, and other frontend modules MUST NOT communicate directly with backend persistence or reproduce backend business logic.

## Unified design

The entire application MUST present a coherent visual system.

Views and components MUST consume the shared design tokens and visual conventions rather than independently defining competing design systems.

Typography, spacing, and other shared visual foundations MUST remain consistent throughout the application.

## Internationalization

All user-visible text MUST be provided through the shared internationalization system.

Locale files MUST live under:

`src/webapp/locales/`

Each locale MUST be represented by a JSON file.

Locale JSON MUST use hierarchical objects grouped by domain or feature.

For example:

{
  "auth": {
    "title": "...",
    "github": "..."
  }
}

Flat translation keys such as `auth.title` MUST NOT be used as the JSON structure.

User-visible text MUST NOT be hardcoded anywhere under `src/webapp/**`, including HTML, JavaScript, views, components, loading states, error states, fallback content, and accessibility text.

All user-visible text MUST come from the shared internationalization system.

Views, components, and other frontend modules MUST NOT load locale files directly.

Locale loading, translation resolution, fallback behavior, and active-locale management MUST be centralized in the shared internationalization module.

Adding a new supported language MUST only require adding its locale data and registering/selecting the locale where necessary; application views MUST NOT require language-specific changes.

## Responsive design

The entire web application MUST be responsive.

Every view and visual component MUST work across the relevant viewport sizes and MUST NOT assume a fixed desktop viewport.

Responsive behavior SHOULD be implemented with CSS whenever possible.

## Testing

Frontend behavior MUST be covered by functional tests under `test/functional/` using Jest and Puppeteer as defined by the repository root `AGENTS.md`.

Functional tests SHOULD exercise real user-visible flows through the SPA and its REST integration rather than bypassing application layers.

All global rules from the repository root `AGENTS.md` remain applicable.
