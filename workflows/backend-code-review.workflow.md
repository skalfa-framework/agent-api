---
description: Backend Code Review Workflow
---

# Backend Code Review Workflow
Version: 1.0.0

This workflow defines a strict, non-destructive **code review and structural alignment process** for backend projects using **Skalfa API**.

The workflow ensures that the codebase conforms to Skalfa folder structures, slug-like naming conventions, Active Record patterns, and vertical formatting rules.

---

## WORKSPACE DIRECTORY MAPPING

*   **Backend Project Root**: `./skalfa-api/` (or current project folder containing `app/`)
*   **Agent Folder**: `./.agents/`
*   **Records Directory**: `./.agent.tools/records/`

---

## REVIEW CHECKLIST

The reviewer agent MUST analyze all modified and new files against the following criteria:

### 1. Folder Structure & Placement
*   Controllers and models MUST be organized inside feature/domain folders under `app/controllers/` and `app/models/` (e.g., `app/controllers/iam/`, `app/models/iam/`).
*   Business logic MUST live inside **service objects** located in a nested `_services/` directory:
    *   *Controller-level*: `app/controllers/<module>/_services/` (for endpoint orchestration).
    *   *Model-level*: `app/models/<module>/_services/` (for model-specific logic/calculations, registered inside the model class).
*   No business logic is allowed directly inside controllers or routes.

### 2. File Naming (Slug-like)
*   All service files MUST be named using a **slug-like** format with the `.service.ts` suffix (e.g., `user-registration.service.ts`, `booking.service.ts`).
*   Controller files MUST be named using a **slug-like** format with the `.controller.ts` suffix (e.g., `auth.controller.ts`).
*   Model files MUST be named using a **slug-like** format with the `.model.ts` suffix (e.g., `user.model.ts`).
*   Route files MUST be named using a **slug-like** format with the `.routes.ts` suffix (e.g., `base.routes.ts`).

### 3. Coding Patterns
*   **Database Queries**: MUST use the Active Record pattern via `Model.query()` (e.g., `User.query().where(...)`).
*   **Data Saving/Updating**: MUST use the `.pump(...)` pattern (e.g., `await record.pump(payload)`).
*   **Model Relations**: MUST declare relations using decorators (e.g., `@BelongsTo(() => Product)`) and MUST NOT use the `static relations` attribute.
*   **Validation**: Validation MUST be executed inside the controller using `await c.validation({ ... })` before processing.
*   **Permissions**: Permissions MUST be registered using `permission.register({ ... })` and guarded in controller methods using `p.have("xxx.yy").guard(c)`.
*   **Route Setup**: CRUD routes should be mapped using the `api(route, "name", Controller)` helper.

### 4. Code Formatting (Vertical Alignment)
*   Object keys, variable assignments, and type definitions MUST be aligned vertically in block form (column-based alignment using spaces).
    *   *Example of correct alignment*:
        ```typescript
        await c.validation({
            username  :  ["required", "max:100"],
            password  :  ["required", "max:100"],
        })
        ```
*   **Single-Line Imports**: Verify all import statements are written on a single line. Do not allow multi-line imports (no wrapping of curly braces).
*   Do not break lines for function calls or chaining unless the line exceeds reasonable length or becomes hard to read.

---

## STAGE 1 — EXECUTION

### Step 1.1 — Run Code Review
*   Scan all modified files in the current commit or task scope.
*   Verify each file against the checklist above.
*   Identify any violations (e.g., camelCase filenames, unaligned colons, business logic in controllers).

### Step 1.2 — Apply Alignment / Refactoring
*   If violations are found:
    *   Rename files to match the slug-like convention.
    *   Move files to their correct directories.
    *   Refactor code formatting to apply vertical alignment.
    *   Extract business logic from controllers into service objects in `_services/`.
*   Verify that these changes do NOT alter the functional behavior of the feature.

### Step 1.3 — Finalize Review Report
*   Create a code review report: `./.agent.tools/records/activities/act-<num>-review-report.md`.
*   Record the `CODE_REVIEW_COMPLETED` event in `./.agent.tools/records/ledger.jsonl`:
    ```json
    {"timestamp": "TIMESTAMP", "agent": "AGENT_NAME", "event": "CODE_REVIEW_COMPLETED", "payload": {"report_file": "./.agent.tools/records/activities/act-<num>-review-report.md"}}
    ```