---
trigger: always_on
---

# Backend Development Rules
Version: 1.0.0

## Purpose

This document defines **MANDATORY backend engineering rules** for projects using **Skalfa API**. These rules are derived directly from Skalfa API's architectural patterns and establish the standard patterns for directory structure, thin controllers, validation, permissions, Active Record query handling, and code formatting.

If an implementation violates these rules $\rightarrow$ **the task MUST NOT be considered DONE**.

---

## 1. Mandatory Research & Planning Phase (Registry-First)

Before creating any plan (such as `implementation_plan.md`) or performing any codebase analysis/research:
*   The agent **MUST** read the local Knowledge Registry at `./.agents/knowledges/registry.md` to understand the available technical services and utilities.
*   Identify which specific utilities (e.g. ORM, Controller, Validation, Permission, Auth) or technical services (e.g. Database, Storage, Email, Queue) are required for the task.
*   The agent **MUST** open and read those specific knowledge files in `./.agents/knowledges/` **BEFORE** exploring the codebase or proposing any changes. This prevents unnecessary codebase searches and ensures alignment with Skalfa API patterns.

---

## 2. Core Principles (Binding)

### 2.1 Feature-Driven Architecture
*   Code MUST be organized around features/domains under `app/controllers/` and `app/models/`.
*   Cross-feature coupling is forbidden unless explicitly designed.

### 2.2 Service Objects (Controller-level & Model-level)
*   All business logic MUST live inside **service objects** located in a nested `_services/` folder.
*   **Controller-level Services**:
    *   Located in `app/controllers/<module>/_services/`.
    *   Responsible for orchestrating endpoint business flows, handling input processing, and preparing response data.
    *   Controllers MUST remain thin: they only handle transport (Elysia context), request parsing, validation, permission guarding, and response shaping. Direct business logic or database mutation inside controllers is strictly forbidden.
*   **Model-level Services**:
    *   Located in `app/models/<module>/_services/`.
    *   Responsible for model-specific logic, calculations, or data generation (e.g., generating a transaction number, recalculating booking totals).
    *   These services MUST be registered/imported and called inside the Model class (e.g. `generateNumber = async () => await BookingService.generateNumber()`).


### 2.3 Explicit Backend Behavior
*   All backend behavior (validations, authorization checks, database transactions, side effects) MUST be explicit and traceable.
*   No magic branching, implicit behavior, or hidden state.

---

## 3. File & Naming Rules (Slug-like)

Readability and structural consistency are more important than brevity.

*   **Service Files**: MUST use slug-like naming with the `.service.ts` suffix.
    *   *Controller Service*: `app/controllers/iam/_services/user-registration.service.ts`
    *   *Model Service*: `app/models/transaction/_services/booking.service.ts`
    *   *Incorrect*: `app/controllers/iam/_services/UserRegistration.ts`, `app/controllers/iam/_services/service.ts`
*   **Controller Files**: MUST use slug-like naming with the `.controller.ts` suffix.
    *   *Correct*: `app/controllers/iam/auth.controller.ts`
*   **Model Files**: MUST use slug-like naming with the `.model.ts` suffix.
    *   *Correct*: `app/models/iam/user.model.ts`
*   **Route Files**: MUST use slug-like naming with the `.routes.ts` suffix.
    *   *Correct*: `app/routes/base.routes.ts`

---

## 4. Layering & Coding Patterns

### 4.1 Permissions & Guards
*   Permissions MUST be registered at the top of the controller file using `permission.register({ ... })`.
*   Controller methods MUST be guarded using the registered permission object:
    ```typescript
    const p = permission.register({
        "400": {
            name: "Presence",
            accesses: {
                "00": "Melihat",
                "01": "Membuat",
            }
        }
    })
    
    // Inside Controller class:
    static async index(c: ControllerContext) {
        p.have("400.00").guard(c)
        // ...
    }
    ```

### 4.2 Controller & Validation
*   All external inputs (body, query, params) MUST be validated inside the controller method using `await c.validation({ ... })` before delegating to services.
*   Responses MUST be returned using the helper methods on the controller context `c`:
    *   `c.responseSuccess(data, message)`
    *   `c.responseData(data, total)`
    *   `c.responseSaved(record)`
    *   `c.responseError(err, context)`

### 4.3 Data Access & Active Record
*   Database operations MUST use the Active Record pattern via `Model.query()`.
*   Writing raw queries scattered across controllers is forbidden.
*   Data saving and updating MUST use the `.pump(...)` method:
    ```typescript
    // Creating
    record = await (new Presence).pump({ ...c.payload, user_id: c.user.id })
    // Updating
    await record.pump(c.payload)
    ```
*   Transactions MUST be explicit using `db.transaction()` and handled with try-catch rollback/commit blocks.

### 4.4 Reuse of Existing Utilities (Utility Reuse Discipline)
*   Skalfa API provides a rich set of built-in utilities (e.g., `api`, `middleware`, `auth`, `db`, `permission`, `queue`, `redis`, `socket`, `storage`, `validation`, `cron`, `mail`, `da`) via the `@utils` path alias.
*   The agent MUST reuse these existing utilities. Writing custom helper functions or reinventing logic (like custom db wrappers, manual file upload handlers, or custom validation engines) is strictly forbidden.
*   Do not casually create custom utility files.

---

## 5. Code Formatting (Vertical Alignment)

*   Object keys, variable assignments, and type definitions MUST be aligned vertically in block form (column-based alignment). Use spaces to align `:` and `=` consistently within the same block.
    *   *Example*:
        ```typescript
        await c.validation({
            username  :  ["required", "max:100"],
            password  :  ["required", "max:100"],
        })
        ```
*   **Single-Line Imports**: All import statements (including multiple destructured modules/bindings) MUST be written on a single line. Breaking imports into multiple lines (e.g., placing bindings on new lines with enters) is strictly forbidden.
    *   *Correct*: `import { one, two, three } from '...'`
    *   *Incorrect*:
        ```typescript
        import {
          one,
          two,
          three
        } from '...'
        ```
*   Do NOT break lines for function calls or chaining unless the line becomes hard to read or exceeds reasonable length.

---

## 6. Skalfa CLI Integration

*   When creating new modules, controllers, models, or routes, the agent MUST use `skalfa-cli` first:
    ```bash
    skalfa make:controller <name>
    ```
*   The agent may modify the generated files but must preserve the blueprint comments if the file is intended to be managed by the blueprint engine:
    ```typescript
    // ============================================
    // ## file THIS FILE IS AUTO-GENERATED BY BLUEPRINT
    // ?? Blueprint : name.blueprint.json
    // !! If this comment is removed, blueprint engine WILL NOT override this file.
    // ============================================
    ```

---

## 7. Code Modification Boundaries

### 7.1 Writable Paths (agent MAY modify)
*   `app/controllers/**`
*   `app/models/**`
*   `app/routes/**`
*   `app/jobs/**`
*   `app/outputs/**`
*   `database/**`
*   `.agent/records/**`
*   `.agent/test/**`

### 7.2 Read-Only Paths (agent MUST NOT modify)
*   `utils/**`
*   `app/app.ts`

If a fix requires changes inside read-only paths, the agent MUST NOT modify the file, but must report it to the human as `fix_but_need_human`.

---

## 8. Definition of DONE

A backend task is considered **DONE** only when:
1.  Skalfa API patterns and slug-like naming are strictly followed.
2.  Service object pattern is respected (logic in `_services/`).
3.  Controllers remain thin with explicit validation and permission guards.
4.  All database operations use `Model.query()` and `.pump()`.
5.  Code formatting is vertically aligned.
6.  The TypeScript compiler check (`bun tsc --noEmit`) passes with 0 errors.
7.  A test file in `.agent/test/<slug>.test.ts` is created, executed via Bun, and all scenarios pass.
8.  The API documentation is updated by running the `bun skalfa generate:docs --path=<route_path>` command for the newly created or modified API endpoints.
9.  The ledger (`.agent/records/ledger.jsonl`) and state (`.agent/records/state.json`) are updated.

