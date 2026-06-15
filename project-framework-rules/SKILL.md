---
name: project-framework-rules
description: Mandatory rules regarding the use of framework packages (aq-core-framework, aq-standard-framework, aq-legacy-framework) for each application in the monorepo.
---

# Framework Packages Usage Rules

This project is a Monorepo containing multiple applications (`apps/`) and multiple shared libraries (`packages/`). To ensure consistency and avoid bloating the application bundle size, when coding or installing modules, you **must strictly adhere** to the following framework allocation rules:

### 1. UI Components (@aq-fe/core-ui)
- **Scope**: Used by all projects.
- **Rule**: All projects inside `apps/` are allowed to import and use `@aq-fe/core-ui`.

### 2. Logic Processing Frameworks (@aq-fe/aq-*-framework)
Absolutely DO NOT import redundant frameworks that do not belong to the project. The mapping rules are as follows:

- **`sae-new`**: ONLY use `@aq-fe/aq-core-framework`.
- **`spm`**: ONLY use `@aq-fe/aq-standard-framework`.
- **All other projects** (like `eaq`, `school`, `college`, etc.): use `@aq-fe/aq-legacy-framework`.

---

### Important Note for Developers:
Looking at the actual `package.json` file, sometimes you might see a project being installed with all 3 frameworks (e.g., when adding this skill, the `spm` project was found to have `core`, `legacy`, and `standard` installed).
=> When working, you **are only allowed to import code from the exact framework assigned above**. The redundant installation might be due to cloning configuration files, but when importing code, absolutely do not mistakenly call the library of another project.
