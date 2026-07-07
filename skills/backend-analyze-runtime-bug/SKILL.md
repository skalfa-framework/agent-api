---
name: backend_analyze_runtime_bug
description: Analyze backend runtime failures by executing feature-scoped tests and inspecting logs.
---

# Backend Analyze Runtime Bug Skill

## Goal
Determine the root cause of backend runtime errors by executing feature-scoped API tests in `.agents/test/` and inspecting runtime logs.

## Allowed Tools & Actions
*   Create and edit test files inside `.agents/test/`.
*   Run tests using Bun:
    ```bash
    bun .agent/test/<slug>.test.ts
    ```
*   Read backend source files (writable and read-only paths) to trace execution flow.
*   Read server/console logs.

## Step-by-Step Instructions

1.  **Read API Docs**: Read the generated API documentation in the `./docs/` folder (or check the route definitions in `app/routes/`) to identify the target API endpoint, query parameters, and payload structure.
2.  **Identify Feature**: Determine which feature or API endpoint is failing.
3.  **Locate/Create Test File**:
    *   Check `.agents/test/` for an existing `<slug>.test.ts` file.
    *   If none exists, create a new one.
3.  **Write Scenarios**:
    *   Implement a scenario that reproduces the reported failure.
    *   Ensure other scenarios (happy path, invalid input, auth, edge case) are also present.
    *   For queues/events, trigger them directly within the test file.
4.  **Execute Test**:
    *   Run `bun .agents/test/<slug>.test.ts` to reproduce the error.
    *   Capture the HTTP status, response payload, error message, stack trace, and console logs.
5.  **Correlate**: Match the stack trace and error message to the specific controller, service, model, or middleware.
6.  **Report & Record**:
    *   Create a bug analysis report: `.agents/records/activities/act-xxx-bug-analysis-xxx.md`.
    *   Append the `BUG_FOUND` event to `.agents/records/ledger.jsonl`.
    *   Update `.agents/records/state.json` to list the bug as active.

## Forbidden Actions
*   Modifying database schemas, running migrations, or mutating persistent data.
*   Editing production backend logic during the analysis phase.
*   Skipping test creation or running tests outside `.agents/test/`.
