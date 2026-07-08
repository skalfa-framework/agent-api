---
description: Backend Runtime Debug Workflow
---

# Backend Runtime Debug Workflow
Version: 1.0.0

This workflow defines a strict, feature-driven, and ledger-governed **runtime debugging process for a backend project using Skalfa API**.

The agent MUST use runtime tests as the primary observation and verification tool.

---

## WORKSPACE DIRECTORY MAPPING

*   **Backend Project Root**: `./skalfa-api/` (or current project folder containing `app/`)
*   **Records Directory**: `./.agent.tools/records/`
*   **Test Directory**: `./.agent.tools/test/`

---

## STAGE 1 — REPRODUCTION & ANALYSIS

### Step 1.1 — Identify the Issue
*   The agent MUST read the generated API documentation in `./docs/` (or check the route definitions in `app/routes/`) to identify the target API endpoint, its expected query parameters, and payload structure.
*   Determine the failing feature, API endpoint, or queue handler.
*   Check `./.agent.tools/test/` for an existing test file for this feature (e.g., `./.agent.tools/test/<slug>.test.ts`).
*   If no relevant test file exists, create a new one:
    *   File Path: `./.agent.tools/test/<slug>.test.ts`
    *   Test Content: Write multiple scenarios (happy path, invalid input, auth, edge case, and a scenario reproducing the reported runtime failure).
    *   Queue/Event Simulation: Trigger the queue jobs or event handlers directly inside the test code.

### Step 1.2 — Execution & Capture
*   Execute the test file to reproduce the failure:
    ```bash
    bun .agent.tools/test/<slug>.test.ts
    ```
*   Capture:
    *   HTTP status codes
    *   Response payloads
    *   Error messages and stack traces
    *   Server/queue logs

### Step 1.3 — Record Bug
*   Create a bug analysis report: `./.agent.tools/records/activities/act-<num>-bug-analysis-<slug>.md`.
*   Record the `BUG_FOUND` event in `./.agent.tools/records/ledger.jsonl`:
    ```json
    {"timestamp": "TIMESTAMP", "agent": "AGENT_NAME", "event": "BUG_FOUND", "payload": {"bug_id": "BUG-<num>", "feature": "FEATURE_NAME", "analysis_file": "./.agent.tools/records/activities/act-<num>-bug-analysis-<slug>.md", "test_file": "./.agent.tools/test/<slug>.test.ts"}}
    ```
*   Update `./.agent.tools/records/state.json` to add the bug to the active bugs list.

---

## STAGE 2 — FIXING & VERIFICATION

### Step 2.1 — Apply Minimal Fix
*   Modify the backend source code in writable paths (`app/controllers/**`, `app/models/**`, etc.).
*   Do NOT modify read-only paths (like `utils/**`). If a utility change is needed, mark as `need_human`.
*   Maintain vertical alignment and coding style.

### Step 2.2 — Static Verification
*   Run the compiler check to ensure no new type errors are introduced:
    ```bash
    bun tsc --noEmit
    ```

### Step 2.3 — Runtime Verification
*   Run the test file again:
    ```bash
    bun .agent.tools/test/<slug>.test.ts
    ```
*   Ensure all scenarios (including the reproduction scenario) now pass successfully.

---

## STAGE 3 — FINALIZATION

### Step 3.1 — Record Resolution
*   If the bug is resolved:
    *   Create a resolution report and patch file in `activities/`.
    *   Record the `BUG_RESOLVED` event in `./.agent.tools/records/ledger.jsonl`:
        ```json
        {"timestamp": "TIMESTAMP", "agent": "AGENT_NAME", "event": "BUG_RESOLVED", "payload": {"bug_id": "BUG-<num>", "patch_file": "./.agent.tools/records/activities/act-<num>-fix-<slug>.patch", "resolution_file": "./.agent.tools/records/activities/act-<num>-resolution-<slug>.md"}}
        ```
    *   Update `./.agent.tools/records/state.json` to mark the bug status as `RESOLVED`.