---
name: project-framework-rules
description: Mandatory rules regarding the use of framework packages (aq-core-framework, aq-standard-framework, aq-legacy-framework) for each application in the monorepo.
---

# Framework Packages Usage Rules

This project is a Monorepo containing multiple applications (`apps/`) and multiple shared libraries (`packages/`). To ensure consistency, avoid bloating the application bundle size, and prevent circular dependencies, when coding or installing modules, you **must strictly adhere** to the following framework allocation rules:

### 1. UI Components (@aq-fe/core-ui)
- **Scope**: Used by all projects.
- **Rule**: All projects inside `apps/` are allowed to import and use `@aq-fe/core-ui`.
- **Context-Based Permission Mechanism**: Shared UI components in `@aq-fe/core-ui` (such as `CustomButton`, `CustomActionIcon`, `CustomDataTable`, etc.) **MUST NOT** directly import or use backend-specific stores (e.g., `usePermissionStore`). Instead, they determine user permissions (create, update, read, delete, export, etc.) dynamically via the `PermissionContext` (using the `usePermissionContext()` hook).
- **Application Responsibility**: Each application must wrap its root layout or provider tree with the correct `PermissionProvider` corresponding to its backend:
  - Legacy applications (`eaq`, `sea`, `srm`, `lom`, etc.) must use `LegacyPermissionProvider` (imported from `@aq-fe/aq-legacy-framework`).
  - New applications (like `sae-new`) must use their own specific `PermissionProvider` matching the new API's permission structure.

### 2. Logic Processing Frameworks (@aq-fe/aq-*-framework)
Absolutely DO NOT import redundant frameworks that do not belong to the project. The mapping rules are as follows:

- **`sae-new`**: ONLY use `@aq-fe/aq-core-framework`.
- **`spm`**: ONLY use `@aq-fe/aq-standard-framework`.
- **All other projects** (like `eaq`, `school`, `college`, `sea`, `srm`, `lom`): use `@aq-fe/aq-legacy-framework`.

### 3. Strict Decoupling Rules (No Circular or Reverse Dependencies)
- **`@aq-fe/aq-core-framework`** is designed for the new backend system and is strictly isolated.
- **CRITICAL RULE**: Code inside `@aq-fe/aq-core-framework` **MUST NEVER** import anything from `@aq-fe/aq-legacy-framework`.
- If the new backend needs a component, utility, or business logic similar to one present in the legacy framework (e.g., `CustomButtonImport`, `CustomButtonDelete`, `CustomApiSelect`), you **MUST clone and refactor** that component directly into `@aq-fe/aq-core-framework` (or a shared library independent of the legacy framework). You cannot reuse it directly from legacy.

---

### Important Note for Developers:
Looking at the actual `package.json` file, sometimes you might see a project being installed with all 3 frameworks (e.g., when adding this skill, the `spm` project was found to have `core`, `legacy`, and `standard` installed).
=> When working, you **are only allowed to import code from the exact framework assigned above**. The redundant installation might be due to cloning configuration files, but when importing code, absolutely do not mistakenly call the library of another project.
