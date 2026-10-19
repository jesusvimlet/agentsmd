# AGENTS.md

Version: 0.0.7

This file defines my default stack, workflow, and non-negotiable rules.

Read it fully before doing any work in this repository.

---

## Versioning (semver)

* Projects follow **semver** (`MAJOR.MINOR.PATCH`) in `package.json`, honored as far as it makes sense:

  * PATCH `1.0.0 → 1.0.1` — the DEFAULT increment: fixes, tweaks, internal changes

  * MINOR `1.0.x → 1.1.0` — new backward-compatible functionality

  * MAJOR `x.y.z → 2.0.0` — breaking changes

* When I say **"bump version"** → increment PATCH. **"bump minor"** / **"bump major"** → increment that position and reset the lower ones to 0.

* A bump order never implies touching git (§1).

* The bump target depends on context: inside a project → `version` in `package.json`; when editing this AGENTS.md → the `Version:` header at the top.

* This file keeps its own simple versioning to track its progress: any edit bumps it, PATCH by default, one bump per task — unpublished edits share one bump until pushed.

## 0. Read project-specific instructions FIRST

* If an `agents/` directory exists in the project root, you MUST read **all files inside it, recursively** before starting any task. It contains project-specific instructions that complement this file.

* Precedence (highest wins):

  1. Explicit instructions given in the current conversation

  2. Files under the project's `agents/` directory

  3. This AGENTS.md

## 1. GIT — never commit or push on your own (CRITICAL, NEVER ASSUME)

* You MUST NOT run `git add`, `git commit`, `git push`, or any operation that writes to the repository history unless I give the explicit order **"push"**.

* Finishing a task, passing tests, or me saying "ok", "great", "done", "looks good" is NEVER authorization to touch git. Do not assume it. Ever.

* The order **"push"** means the FULL cycle, in this exact order:

  ```
  git add .
  git commit -m "<short, descriptive message>"
  git pull
  git push
  ```

* One "push" order authorizes exactly one cycle. The next one requires a new explicit order.

* If any step fails (conflicts on `git pull`, rejected push, hooks), STOP and report. Never `--force`, never skip hooks, never resolve conflicts destructively without asking.

* Read-only git commands (`git status`, `git diff`, `git log`) are always allowed.

* `git init` on a brand-new project is allowed, but no commits.

## 2. Stack

These technologies and instructions are reusable standards: they apply, and remain mandatory, only when the corresponding technologies, components, or architectural capabilities are present in the consuming project. Listing them here does not require their adoption, installation, or introduction; instructions for absent technologies or components do not apply.

| Layer           | Choice                                                                                                             |
| --------------- | ------------------------------------------------------------------------------------------------------------------ |
| Runtime         | Node.js — latest LTS                                                                                               |
| Backend         | Express + helmet                                                                                                   |
| Frontend        | Vanilla JS + vanilla CSS ONLY (no frameworks, no TypeScript, no CSS preprocessors like Sass/Tailwind, no bundlers) |
| Database        | PostgreSQL-compatible database whenever persistence is needed; database access only through the persistence layer  |
| Package manager | pnpm                                                                                                               |
| Configuration   | dotenvx + `.env.local` / `.env.production`                                                                         |
| Tests           | Jest (unit) + Puppeteer (functional/E2E)                                                                           |

Server bootstrap reference:

```
const express = require('express');
const helmet = require('helmet');

const app = express();
const PORT = process.env.PORT || 3000;

app.use(helmet());

app.listen(PORT, () => {
  console.log(`Listening on port ${PORT}`);
});
```

When the project includes a webapp, it MAY be served statically by the same Express application:

```
app.use(express.static('src/webapp'));
```

If helmet's default CSP blocks a legitimate frontend asset, configure the CSP explicitly — never solve it by removing helmet.

## 3. Project structure (mandatory)

```
src/
  webapp/                 # Frontend when present: vanilla JS, HTML, CSS

server/
  endpoint/               # REST layer: Express routers, req/res + validation only
  handler/                # Business logic: no Express, no DB driver code here
    lib/                  # Private handler-specific implementation modules when needed
  persistence/            # ALL runtime database access lives here
  schema/                 # Backend endpoint and persistence schemas
  lib/                    # Shared server infrastructure/helpers

test/
  functional/             # E2E & API tests (Puppeteer + Jest)
  unit/                   # Unit tests (Jest)
```

* Layering rule: `endpoint → handler → persistence`.

* Endpoints never talk to the database directly.

* Handlers never receive or manipulate Express `req`/`res`.

* Only `persistence/` may perform application runtime database access or import the database client/SDK.

* Supabase authentication infrastructure is an explicit exception: it MAY use the Supabase client outside `persistence/` exclusively for authentication operations such as verifying the authenticated user. This exception MUST NOT be used for application/domain database access.

* `server/lib/` contains genuinely shared server infrastructure or utilities and MAY be used by server layers where appropriate.

* `server/handler/lib/<entity>/` contains private implementation details belonging to that entity's main handler. These modules MUST NOT be treated as shared server utilities. Their exact boundaries are governed by the handler-specific AGENTS.md when present.

* Layer-specific AGENTS.md files may impose stricter entity boundaries and dependency rules than this general structure.

* When `src/webapp` exists, it talks to the backend through public REST endpoints and the authenticated Socket.IO interface. Real-time communication is centralized in `js/api/socket.mjs`; transport infrastructure delegates plain data to `server/handler/socket.js`. Live presence is ephemeral.

## 4. Testing (mandatory, no exceptions)

* EVERY piece of code you produce MUST ship with a way to test it.

* Defaults: **Jest** for unit tests in `test/unit/`, **Puppeteer** (driven by Jest) for browser functional/E2E tests in `test/functional/`.

* Backend logic → unit tests for `handler/` and applicable `lib/` code (mock persistence/external dependencies as required).

* REST endpoints → functional tests through the public HTTP interface.

* Frontend flows → functional tests through the public application interface, using Puppeteer when browser behavior is involved.

* Required `package.json` scripts:

  ```
  {
    "scripts": {
      "test": "jest",
      "test:unit": "jest test/unit",
      "test:functional": "jest test/functional"
    }
  }
  ```

* Run the relevant tests after your changes and report the results before declaring a task done.

* If tests cannot run in the current environment, say so explicitly and provide the exact commands for me to run them.

* Detailed testing architecture and boundaries are governed by the applicable AGENTS.md under `test/` when present.

## 5. Secrets & configuration

* Environment configuration MUST use **dotenvx**.

* Projects use two runtime environment files:

  * `.env.local` — local development.

  * `.env.production` — production.

* `.env.local` and `.env.production` MUST be gitignored.

* They MUST NEVER be committed, logged, or printed.

* Never hardcode secrets, tokens, credentials, connection strings, ports, or environment-specific configuration in source code.

* `.env.example` MUST document required variables using placeholder values only.

* dotenvx encryption MUST NOT be used.

* `.env.keys` MUST NOT be created or used.

* Standard scripts:

  ```
  {
    "scripts": {
      "start": "dotenvx run -f .env.local -- node server/index.js",
      "start:prod": "dotenvx run -f .env.production -- node server/index.js"
    }
  }
  ```

## 6. Dependencies

* Package manager is **pnpm**.

* Versions in `package.json` are ALWAYS exact pins:

  ```
  "express": "4.19.2"
  ```

* Never use `^`, `~`, `*`, `latest`, or version ranges in `dependencies` or `devDependencies`.

* Enforce exact versions in `.npmrc`:

  ```
  save-exact=true
  ```

* `pnpm-lock.yaml` MUST be committed.

* Fresh or CI installs SHOULD use:

  ```
  pnpm install --frozen-lockfile
  ```

* Prefer Node's standard library and already-present dependencies.

* Adding a new dependency requires a good reason — state it when you do.

* Do not introduce dependency-management, supply-chain, sandboxing, or package-security tooling unless explicitly requested.

* The database implementation is intentionally not prescribed by this general AGENTS.md.

* When persistence is required, use the database client or abstraction selected by the project and keep application/domain database access behind `persistence/`.

* Supabase authentication infrastructure may use the Supabase client outside `persistence/` exclusively for authentication operations.

* Do NOT invent or install a package named `database` or treat "PostgreSQL-compatible database" as a dependency name.

## 7. .gitignore baseline (every project)

```
# dependencies
node_modules/

# environment / secrets
.env
.env.*
!.env.example

# NOTE: pnpm-lock.yaml is committed on purpose — do NOT ignore it
package-lock.json

# build & misc
dist/
build/
coverage/
*.log
.DS_Store
```

## 8. New project bootstrap

When I ask for a new project, set up in this order:

1. Directory structure from §3, creating `src/webapp/` only when the project includes a web frontend.

2. `.gitignore` from §7.

3. `package.json` using the latest Node.js LTS and pnpm.

4. `.npmrc` with `save-exact=true`.

5. Applicable scripts from §4 and the dotenvx scripts from §5.

6. Install `express`, `helmet`, and `@dotenvx/dotenvx`; install `jest` for tests and `puppeteer` when browser functional tests are required.

7. If the project needs persistence, install the project-selected database client/abstraction separately.

8. Express server in `server/`, with helmet enabled.

9. If the project includes a webapp, serve `src/webapp` as appropriate.

10. Create `.env.local`, `.env.production`, and `.env.example`. Runtime environment files MUST remain gitignored.

11. One passing sample unit test and the applicable functional test coverage for the initial application surface.

12. README with run/test instructions.

13. NO git commits — wait for my explicit "push" (§1).

## 9. Definition of done (check before reporting a task complete)

* [ ] Project `agents/` directory read recursively (if present)

* [ ] Structure and layering respected (§3)

* [ ] Tests created/updated and passing (§4)

* [ ] No secrets or credentials hardcoded or committed (§5)

* [ ] New dependencies exact-pinned (§6)

* [ ] NO `git add`/`commit`/`push` performed unless I explicitly said "push" (§1)

## 10. Temporary workspace for agent-created files

* Never create temporary diagnostic, debugging, experimental, reproduction, or ad-hoc test files in the repository root or anywhere in the production/test source tree.

* All temporary files created only to investigate or verify something must go under `.opencode/tmp/`.

* `.opencode/tmp/` is disposable workspace and must never contain permanent project tests or production code.

* Permanent tests belong under the existing `test/` structure.

* Delete temporary files when they are no longer useful.
