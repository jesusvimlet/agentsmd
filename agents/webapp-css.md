# CSS Layer

## Scope

The rules in this document apply only to files under:

`src/webapp/css/**`

This directory contains all application styles.

CSS MUST remain modular, predictable, responsive, and consistent across the entire application.

## Visual modules

Every visual view or component SHOULD have its own CSS module when it requires specific styles.

The CSS structure MUST mirror the corresponding visual JavaScript structure whenever applicable.

For example:

`js/views/organization.js`

→ `css/views/organization.css`

`js/components/left-sidebar.js`

→ `css/components/left-sidebar.css`

A CSS file MUST NOT be created merely to mirror a JavaScript module that has no specific visual styles.

## Class naming

Every view or visual component MUST have a canonical name in `kebab-case`.

All CSS classes owned by that module MUST use the module name as their prefix.

For example, `left-sidebar.css`:

```css
.left-sidebar {}

.left-sidebar-header {}

.left-sidebar-title {}

.left-sidebar-content {}
```

And `organization.css`:

```css
.organization {}

.organization-header {}

.organization-content {}
```

Generic class names such as `.header`, `.title`, `.content`, or `.container` MUST NOT be used for module-specific elements.

This convention MUST remain consistent between filenames, JavaScript modules, and CSS class prefixes.

## Global styles

Styles intended to be shared across the entire application MUST be explicitly defined as global styles rather than duplicated inside individual modules.

Global styles are responsible for application-wide foundations such as:

- Design tokens.

- Typography.

- Base document styles.

- Shared visual conventions.

Views and components MUST consume the global design system rather than independently redefining common colors, spacing conventions, typography, or other application-wide visual decisions.

## Design tokens

Components MUST consume CSS custom properties defined by the application's design system when an appropriate design token exists.

## Responsive design

The entire application MUST be responsive.

Views and components MUST work across relevant viewport sizes and MUST NOT assume a fixed desktop viewport.

Responsive behavior SHOULD remain local to the view or component that owns it whenever possible.

Modules MUST NOT introduce responsive rules that unexpectedly alter unrelated modules.

## Isolation

A module's CSS MUST style only elements belonging to that module.

A module MUST NOT depend on another module's private CSS classes or override them as a shortcut.

Shared visual behavior MUST be promoted to the global styling system rather than created through cross-module CSS dependencies.

## Tooling

Only vanilla CSS is allowed.

CSS preprocessors, CSS frameworks, generated CSS systems, and bundlers MUST NOT be introduced.

All CSS MUST be directly usable by the browser.

All global rules from the repository root `AGENTS.md` remain applicable.
