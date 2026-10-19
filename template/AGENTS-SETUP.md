# One-time instruction setup

## Purpose

`AGENTS.md` and the documents under `agents/` contain reusable development instructions. `SETUP.md` lists the configuration tasks to review when adopting those instructions in a new project.

`TODO` markers identify pending configuration decisions. The user is responsible for reviewing and resolving every TODO before using the agent instructions in a project. Once all TODOs are resolved, this document is no longer required for normal development.

## TODO conventions

Each TODO must explicitly reference its target instruction file and section and include all three fields below. Use this template for future entries:

```markdown
- [ ] TODO
  - **Target:** `<exact instruction file path>`
  - **Section:** `<exact section heading>`
  - **Action:** <what the user must review, confirm, or configure>
```

Use the exact repository-relative file path, such as `AGENTS.md` or `agents/handler.md`. Copy the section heading verbatim, including any numbering. Keep the action concise.

## Resolving TODOs

1. Review each TODO when configuring the instructions for a new project.
2. Determine whether the referenced section applies to the project.
3. If applicable, review the existing instructions and add any necessary project-specific rules or configuration to the relevant instruction document.
4. If no additions are necessary, the TODO can be removed after review.
5. If the referenced component does not exist in the project, no configuration is required for that component.
6. Remove each TODO only after its review is complete.
7. Do not treat TODOs as instructions to automatically install dependencies, create components, or modify the project's architecture.
8. Once every TODO has been reviewed and all applicable configuration has been resolved, setup is complete.

## Separation of responsibilities

- `SETUP.md` is a configuration checklist, not a permanent source of development instructions.
- TODOs do not automatically modify or override the rules in `AGENTS.md` or the documents under `agents/`.
- `AGENTS.md` must not require AI agents to inspect `SETUP.md` during ordinary development.
- Do not introduce runtime TODO validation or blocking mechanisms into `AGENTS.md`.
- Project-specific additions belong in the relevant instruction documents, not permanently in `SETUP.md`.

## Pending configuration

- [ ] TODO
  - **Target:** `AGENTS.md`
  - **Section:** `## 2. Stack`
  - **Action:** Review the existing conditional stack configuration and decide whether any additional project-specific technologies or rules need to be added. If additions are needed, add them to the appropriate instruction section. Otherwise, remove this TODO after review.

- [ ] TODO
  - **Target:** `agents/server.md`
  - **Section:** `## Authentication Context`
  - **Action:** If the project requires authentication, select and configure its authentication provider and add any provider-specific instructions needed by the existing authentication architecture. Remove this TODO after review. If the project does not require authentication, remove the TODO without introducing an authentication provider.

- [ ] TODO
  - **Target:** `AGENTS.md`
  - **Section:** `## 6. Dependencies`
  - **Action:** If the project requires persistence, select the database engine and client or abstraction, and document any project-specific database configuration or rules in the appropriate instruction section. Remove this TODO after review. If persistence is not required, remove the TODO without introducing a database.
