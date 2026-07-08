---
name: backend_development
description: Implement backend features according to an approved backend plan in Skalfa API.
---

# Backend Development Skill

## Goal
Implement backend code strictly according to the approved implementation plan, Skalfa API patterns, and vertical alignment formatting rules.

## Allowed Tools & Actions
*   Edit backend source files in writable paths (`app/controllers/**`, `app/models/**`, `app/routes/**`, etc.).
*   Run the Skalfa CLI to generate boilerplates:
    ```bash
    skalfa make:controller <name>
    ```
*   Run TypeScript compiler checks:
    ```bash
    bun tsc --noEmit
    ```
*   Write and run test files under `.agent.tools/test/` using Bun:
    ```bash
    bun .agent.tools/test/<slug>.test.ts
    ```

## Step-by-Step Instructions

1.  **Read the Plan**: Open the latest planning file in `.agent.tools/records/activities/act-xxx-plan-xxx.md`.
2.  **Generate Boilerplate**: If new controllers or models are required, run `skalfa-cli` to generate them. Do not write boilerplate from scratch.
3.  **Implement Service Logic**:
    *   For complex or long operations, put business logic inside service objects under `app/controllers/<module>/_services/<slug>.service.ts`.
    *   Simple CRUD operations (validating, uploading, saving/updating a single model via `.pump()`, or deleting) MUST be written inline directly in the controller method.
    *   Name the service file using slug-like format.
4.  **Format Code**: Apply vertical alignment to object keys (`: `) and variable assignments (`= `) in the block. Keep imports and function calls on a single line unless excessively long.
5.  **Verify Statics**: Run `bun tsc --noEmit` in the project root. Resolve any compiler errors immediately.
6.  **Verify Runtimes**: Create and run a test file in `.agent.tools/test/<slug>.test.ts` using Bun. Ensure it passes multiple scenarios.
7.  **Record Activities**: Save the code diff patch and explanation in `.agent.tools/records/activities/`, and append the `FEATURE_COMPLETED` event to `.agent.tools/records/ledger.jsonl`.

## Forbidden Actions
*   Modifying files in read-only paths (such as `utils/**` or `app/app.ts`).
*   Putting business or database logic directly inside controllers or routes.
*   Skipping static verification or runtime test generation.
