---
description: Backend Static Debug Workflow
---

# Backend Static Debug Workflow
Version: 1.0.0

This workflow defines a strict, non-destructive process for identifying and fixing **static type errors and compilation issues** in a **Skalfa API** project.

The agent MUST rely on the TypeScript compiler as the source of truth for static correctness. No linting tools (like ESLint) are used.

---

## WORKSPACE DIRECTORY MAPPING

*   **Backend Project Root**: `./skalfa-api/` (or current project folder containing `app/`)
*   **Agent Folder**: `./.agents/`
*   **Records Directory**: `./.agent.tools/records/`

---

## STAGE 1 — STATIC ANALYSIS

### Step 1.1 — Run Compiler Check
*   Execute the TypeScript compiler in the backend project root:
    ```bash
    bun tsc --noEmit
    ```
*   Capture the entire terminal output.

### Step 1.2 — Parse Errors
*   Extract the file paths, line numbers, error codes, and messages.
*   Filter out any errors in read-only directories (e.g., `utils/`). If errors exist in read-only paths, flag them as `need_human`.
*   If static errors are found:
    *   Create a static error report: `./.agent.tools/records/activities/act-<num>-static-errors.md`.
    *   Record the `STATIC_ERRORS_FOUND` event in `./.agent.tools/records/ledger.jsonl`:
        ```json
        {"timestamp": "TIMESTAMP", "agent": "AGENT_NAME", "event": "STATIC_ERRORS_FOUND", "payload": {"report_file": "./.agent.tools/records/activities/act-<num>-static-errors.md"}}
        ```

---

## STAGE 2 — RESOLUTION & RE-VERIFICATION

### Step 2.1 — Fix Errors
*   Apply the skill `backend_fix_static_bug` to resolve the compiler errors.
*   Ensure all fixes:
    *   Are minimal and scoped.
    *   Stay within the writable paths.
    *   Adhere to Skalfa API patterns and vertical alignment rules.

### Step 2.2 — Re-run Compiler Check
*   Re-run the compiler check:
    ```bash
    bun tsc --noEmit
    ```
*   Repeat the fix-and-check cycle until `tsc --noEmit` returns 0 errors.

### Step 2.3 — Record Resolution
*   Once all compiler errors are resolved:
    *   Create a resolution report: `./.agent.tools/records/activities/act-<num>-static-resolved.md`.
    *   Record the `STATIC_ERRORS_RESOLVED` event in `./.agent.tools/records/ledger.jsonl`:
        ```json
        {"timestamp": "TIMESTAMP", "agent": "AGENT_NAME", "event": "STATIC_ERRORS_RESOLVED", "payload": {"resolution_file": "./.agent.tools/records/activities/act-<num>-static-resolved.md"}}
        ```