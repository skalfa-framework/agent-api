---
trigger: always_on
---

# Backend Debug Rules
Version: 1.0.0

## Purpose

This document defines **MANDATORY backend debugging rules** for projects using **Skalfa API**. These rules ensure that all bugs are reproduced, fixed minimally, and verified using automated tests without causing data corruption or regression.

If a bug fix violates these rules $\rightarrow$ **the fix MUST NOT be considered valid**.

---

## 1. Core Debugging Principles

### 1.1 Reproduce First
*   Do NOT attempt to fix a bug before reproducing it.
*   The primary tool for reproduction is a **runtime test** located in `.agent.tools/test/`.

### 1.2 Minimal Scoped Fixes
*   Fixes MUST be as small as possible.
*   Do NOT perform refactoring, change unrelated API contracts, or introduce new endpoints while fixing a bug.
*   Fixes must stay strictly within the affected feature/domain.

### 1.3 Safe Data Handling
*   Tests and fixes MUST NOT perform destructive database mutations on persistent data unless explicitly allowed.
*   Use transactions and rollbacks where possible, or use scoped test accounts/records.

---

## 2. Test-Driven Debugging Protocol

For every reported backend runtime bug:

### 2.1 Test File Placement & Naming
*   The test file MUST be placed in `.agent.tools/test/<slug>.test.ts`.
*   The file name must match the slug of the feature being debugged.
*   The test file MUST consistently import and use testing functions/assertions from `bun:test` (e.g., `import { describe, test, expect, beforeAll, afterAll } from "bun:test"`). Running tests as raw scripts without standard `bun:test` structure or proper assertions is strictly forbidden.

### 2.2 Mandatory Test Scenarios
Each test file MUST cover:
1.  **Reproduction Scenario**: A test case that mimics the exact input and conditions that caused the reported bug.
2.  **Happy Path**: A test case ensuring the core feature still works.
3.  **Invalid Input**: A test case ensuring validation still rejects malformed inputs.
4.  **Authorization Check** (if applicable): A test case verifying permission guards.
5.  **Edge Case**: A test case for boundary values.

### 2.3 Queue and Event Simulation
*   If the bug involves a queue job or event handler, write the trigger code directly in the test file (e.g., calling the job handler function with mock payloads). Do not rely on external cron/queue workers during the test.

---

## 3. Verification & Finalization

### 3.1 Static Verification
*   After applying a fix, the agent MUST run:
    ```bash
    bun tsc --noEmit
    ```
*   Any compiler error MUST be fixed before proceeding.

### 3.2 Runtime Verification
*   Run the test file:
    ```bash
    bun .agent.tools/test/<slug>.test.ts
    ```
*   The fix is only valid when **all scenarios** in the test file pass.

### 3.3 Event Recording
*   Every bug lifecycle transition must be recorded in `.agent.tools/records/ledger.jsonl`:
    *   On reproduction: Record `BUG_FOUND`.
    *   On resolution: Record `BUG_RESOLVED`.
*   Update `.agent.tools/records/state.json` to reflect the active/resolved bugs.