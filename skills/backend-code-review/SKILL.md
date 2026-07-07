---
name: backend_code_review
description: Review code structure, naming conventions, patterns, and vertical alignment for Skalfa API.
---

# Backend Code Review Skill

## Goal
Detect and resolve files, folders, and code patterns that violate Skalfa API conventions, ensuring the codebase remains clean, consistent, and vertically aligned.

## Allowed Tools & Actions
*   Read all backend files.
*   Move or rename files and folders to match conventions.
*   Refactor code formatting to align colons (`:`) and equals signs (`=`).
*   Extract business logic from controllers into service objects.

## Review Checklist

1.  **Folder Structure**:
    *   Controllers and models must be under `app/controllers/<module>/` and `app/models/<module>/`.
    *   Services must be in `_services/` inside either the controller or model module folder.
        *   Controller-level: `app/controllers/<module>/_services/`
        *   Model-level: `app/models/<module>/_services/` (must be registered inside the model file).
2.  **File Naming (Slug-like)**:
    *   Verify all files use slug-like names (e.g., `user-registration.service.ts`, `auth.controller.ts`).
    *   No camelCase or PascalCase file names are allowed in the modified scope.
3.  **Service Responsibility**:
    *   Ensure all business logic is delegated to service objects.
    *   Controllers must only handle input validation, permissions, and response formatting.
4.  **Pattern Compliance**:
    *   Verify queries use `Model.query()`.
    *   Verify updates/saves use `.pump(...)`.
    *   Verify permissions use `permission.register` and `p.have().guard(c)`.
5.  **Code Formatting (Vertical Alignment & Imports)**:
    *   Ensure colons (`:`) in object declarations and equals signs (`=`) in variable assignments are vertically aligned.
    *   Verify all import statements are written on a single line (no multi-line wrapping).
    *   Check that function calls or chaining do not have unnecessary line breaks.

## Step-by-Step Instructions

1.  **Scan Changes**: Review all modified and new files in the current git diff or task scope.
2.  **Identify Violations**: Check each file against the review checklist.
3.  **Apply Refactoring**:
    *   Rename files if they violate the slug-like convention.
    *   Move services to the correct `_services/` directory.
    *   Adjust spacing to enforce vertical alignment.
    *   Refactor controllers to extract business logic into services.
4.  **Report**:
    *   Create a review report: `.agents/records/activities/act-xxx-review-report.md`.
    *   Record the `CODE_REVIEW_COMPLETED` event in `.agents/records/ledger.jsonl`.

## Forbidden Actions
*   Changing functional feature behavior during refactoring.
*   Modifying files in read-only paths (`utils/**`, `app/app.ts`).
