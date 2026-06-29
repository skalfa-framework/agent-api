---
description: Backend Development Workflow
---

# Backend Development Workflow
Version: 1.1.0

This workflow defines a strict, schema-governed **backend planning and development** process for a backend project with **Skalfa API**.

The agent MUST obey workspace boundaries, feature scope, formatting rules, and artifact rules. No inference or undocumented architectural changes are allowed.

---

## WORKSPACE DIRECTORY MAPPING

*   **Backend Project Root**: `./skalfa-api/` (or current project folder containing `app/`)
*   **Target Writable Paths**: 
    *   `app/controllers/**` (including `_services/` subfolders)
    *   `app/models/**`
    *   `app/routes/**`
    *   `app/jobs/**`
    *   `app/outputs/**`
    *   `database/**`
    *   `README.md` (for project outline updates)
*   **Read-Only Paths**: 
    *   `utils/**`
    *   `app/app.ts`
*   **Agent Folder**: `./.agent/` (configured as `agent-backend/` during development)
*   **Records Directory**: `./.agent/records/`
*   **Test Directory**: `./.agent/test/`

---

## EVENT LEDGER & STATE PROTOCOL

Every major stage transition MUST be recorded in `./.agent/records/ledger.jsonl` as an append-only JSON line. After writing to the ledger, `./.agent/records/state.json` must be compiled/updated to reflect the current active tasks, bugs, and API contracts.

All detailed logs, plans, and code diffs must be stored as separate files in `./.agent/records/activities/` and referenced via relative paths in the ledger payload.

---

## STAGE 1 — BACKEND DEVELOPMENT PLANNING

### Step 1.1 — Context & Knowledge Alignment (README-First)
*   The agent MUST read the project's root `README.md` to understand the overall project outline, architecture, and business rules.
*   Verify how the requested feature fits into the existing modules and outline described in `README.md`.
*   **Knowledge Mapping**: The agent MUST read the Knowledge Registry in `./.agent/knowledges/registry.md`. Identify and read ONLY the specific knowledge files (e.g., `validation.md`, `orm.md`) that are directly relevant to the database models, APIs, and utilities required for the task. Reading unrelated knowledge files is forbidden to prevent context bloat.


### Step 1.2 — Outline Adjustment (If Applicable)
*   If the requested feature introduces changes, adjustments, or deviations from the existing project outline, the agent MUST update the corresponding sections in `README.md` (or draft the proposed updates) to keep the project outline accurate.

### Step 1.3 — Plan Creation & Detailing
*   Based on the project outline in `README.md`, define:
    *   Affected API endpoints (method, path)
    *   Request schema (params, query, body) and validation rules
    *   Response schema (success and error)
    *   Involved controllers and models
    *   Business logic to be implemented inside service objects
    *   Permissions required (using `permission.register`)
*   Create a plan detail file: `./.agent/records/activities/act-<num>-plan-<feature>.md`.
*   Record the event in `./.agent/records/ledger.jsonl`:
    ```json
    {"timestamp": "TIMESTAMP", "agent": "AGENT_NAME", "event": "FEATURE_PLANNED", "payload": {"feature": "FEATURE_NAME", "plan_file": "./.agent/records/activities/act-<num>-plan-<feature>.md"}}
    ```

---

## STAGE 2 — IMPLEMENTATION

### Step 2.1 — Code Generation (skalfa-cli)
*   If creating new controllers, models, or routes, use the `skalfa-cli` tool first to generate the boilerplate:
    ```bash
    skalfa make:controller <name>
    ```
*   Ensure auto-generated headers are preserved if the file should not be manually overridden completely.

### Step 2.2 — Manual Coding
*   Implement business logic **exclusively** inside service objects placed in the `_services/` subdirectory of the controller (e.g., `app/controllers/<module>/_services/<slug>.service.ts`).
*   Name all service files using **slug-like** format: `<feature-name>.service.ts`.
*   Ensure all database operations use the Active Record pattern (`Model.query()`) and `.pump(...)` for saving data.
*   **Strict Coding Style**:
    *   Align object keys, variable assignments, and type definitions vertically in block form (column-based alignment).
    *   Do not break lines for imports, function calls, or chaining unless the line exceeds reasonable length.
    *   Use `@utils` and `@models` path aliases.

---

## STAGE 3 — STATIC DEBUGGING (MANDATORY)

Before running the code, the agent MUST verify that it passes static analysis:
*   Run the TypeScript compiler check in the backend project root:
    ```bash
    bun tsc --noEmit
    ```
*   If errors occur:
    *   Apply static bug fixing rules.
    *   Do NOT proceed to runtime tests until `tsc --noEmit` passes with 0 errors.

---

## STAGE 4 — RUNTIME DEBUGGING & SIMULATION (MANDATORY)

For every new or modified feature, the agent MUST write and execute a runtime test:
*   **Test File**: Create a test file in `./.agent/test/<slug>.test.ts`.
*   **Test Content**: 
    *   Simulate real API calls using Bun.
    *   Simulate queue/event background jobs by invoking their triggers directly in the test file.
    *   Implement **multiple scenarios** with different inputs (happy path, invalid input, unauthorized access, edge cases).
*   **Execution**:
    ```bash
    bun .agent/test/<slug>.test.ts
    ```
*   If tests fail, analyze logs, apply fixes, and re-run until all scenarios pass.

---

## STAGE 5 — CODE REVIEW & FINALIZATION

### Step 5.1 — Code Review
*   Verify that:
    *   No files outside the writable paths were modified.
    *   All file names follow the slug convention.
    *   Vertical alignment is consistently applied.
    *   Service object pattern is strictly followed.
*   Create a review report and diff patch:
    *   Report: `./.agent/records/activities/act-<num>-review-<feature>.md`
    *   Patch: `./.agent/records/activities/act-<num>-diff-<feature>.patch`

### Step 5.2 — Ledger Finalization
*   Run the API documentation generator command `bun skalfa generate:docs` to automatically update the API documentation in the `./docs/` folder with the new/modified endpoints, payloads, and query parameters.
*   Record the completion event in `ledger.jsonl`:
    ```json
    {"timestamp": "TIMESTAMP", "agent": "AGENT_NAME", "event": "FEATURE_COMPLETED", "payload": {"feature": "FEATURE_NAME", "review_file": "./.agent/records/activities/act-<num>-review-<feature>.md", "patch_file": "./.agent/records/activities/act-<num>-diff-<feature>.patch"}}
    ```
*   Update `./.agent/records/state.json` to mark the feature/task as `DONE`.

