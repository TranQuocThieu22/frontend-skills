---
name: sae-new-feature
description: Guide on how to create, structure, and integrate new features and pages in the SAE-new Next.js application. Use when adding new functionalities, layouts, or nested pages inside apps/sae-new.
---

# SAE-New Feature Development Skill

This skill provides step-by-step guidance, standards, and patterns for creating and developing features in the `sae-new` application.

## 3. Component Architecture & Reusability

1. **Feature-Specific Components**: If a component (e.g., a modal, a tab, a special button) is only used within a single menu/page (e.g., `operation/planningCommunity`), keep it inside `src/features/<module>/<feature>/components/`.
2. **Shared Business Components**: If a business component or modal needs to be reused across **multiple menus**, **DO NOT** keep it in a specific feature folder.
   - **Where to put them**: Move them to `src/shared/features/<DomainName>/`. For example:
     - `src/shared/features/CommunityActivity/` should hold all Modals, Tabs, and Utility functions related to Community Activities that are shared across different operation menus.
     - `src/shared/features/Semester/CustomSemesterSelect` (Instead of `shared/components/input/...`).
   - **Why?**: Purely generic UI components (like generic text inputs, buttons, etc.) belong in `shared/components/` or `@aq-fe/core-ui`. However, components that contain **business logic, API calls, or domain models** (like fetching the list of semesters, or handling a specific plan creation) belong in `shared/features/`. This prevents cross-feature dependencies and keeps generic folders clean.

## Folder Architecture

The `sae-new` application uses a clean separation between routing (the Next.js `app` router) and feature logic. Always follow this separation of concerns:

1. **Routing Layer** (`apps/sae-new/src/app/...`)
   - Folders in the routing layer must be named using **kebab-case** (e.g., `activity-management`, `score-framework-version`).
   - Pages must only serve as thin wrappers that import and render feature layout components from the `features/` directory.
   - Example: `apps/sae-new/src/app/admin/score-framework-version/page.tsx`
     ```tsx
     import ScoreFrameworkVersionLayout from "@/features/admin/scoreFrameworkVersion/scoreFrameworkVersionLayout";
     export default function Page() {
       return <ScoreFrameworkVersionLayout />;
     }
     ```

2. **Feature Layer** (`apps/sae-new/src/features/<feature-name>/`)
   - Feature directories must be named using **camelCase** (e.g., `activityManagement`, `scoreFrameworkVersion`).
   - Each feature should contain:
     - `components/`: Specific UI components utilized only by this feature, named in **PascalCase** (e.g., `ActivityManagementTable.tsx`).
     - `modal/`: Popups/modals used in this feature, named in **PascalCase** (e.g., `ImportActivityManagementModal.tsx`).
     - `shared/`: Custom hooks, mutations, queries, constants, and types.
     - `<feature-name>Layout.tsx`: The orchestrating entry point of the feature, named in **camelCase** (e.g., `activityManagementLayout.tsx`, `scoreFrameworkVersionLayout.tsx`).
     - **Documentation & Metadata**: Any documentation or API status files (e.g., `API_STATUS_PENDING.md`) related to the feature MUST be placed here in the feature directory, NOT in the routing layer (`src/app/...`).

3. **Consistent Route & Feature Folder Names (Mandatory)**
   - The routing folder name (kebab-case) and the corresponding feature folder name (camelCase) **MUST represent the exact same words**.
   - Example 1: If the feature folder is `planningCommunity`, the routing folder MUST be named `planning-community` (never mismatch names like routing: `planning-register-community` with feature: `planningCommunity`). This guarantees seamless navigation, predictability, and avoids import casing conflicts.
   - Example 2: For "self declaration", the routing page folder MUST be `self-declaration` (kebab-case) and the feature folder MUST be `selfDeclaration` (camelCase).

## Step-by-Step Feature Creation Process

### 1. Define the Feature Directory
Create a subfolder in `apps/sae-new/src/features/` using **camelCase** (e.g., `features/operation/activityManagement`).
```
features/operation/activityManagement/
├── components/
│   ├── ActivityManagementHeader.tsx
│   └── ActivityManagementTable.tsx
├── modal/
│   └── ImportActivityManagementModal.tsx
├── shared/
│   ├── constants.ts
│   ├── types.ts
│   ├── useActivityManagementQueries.ts
│   └── useActivityManagementMutations.ts
└── activityManagementLayout.tsx
```

### 2. Configure State and Queries
Inside `shared/`:
- Put types in `types.ts`.
- Place constants in `constants.ts`.
- Implement API query hooks using `useCustomReactQuery` inside `useActivityManagementQueries.ts`.
- Implement API mutation hooks using `useMutation` inside `useActivityManagementMutations.ts`.

### 3. Build UI Components
- Use `@mantine/core` for layout components (`Box`, `Flex`, `Group`, `Stack`).
- Implement animations inside styled components or style sheets (e.g., `slideInLeft` or `slideInRight`).
- Use shared components from `@aq-fe/core-ui` like `CustomButton` and `CustomTanstackTable` to integrate action permissions and responsive tables natively.

### 4. Create the Layout Entry Point
In `<feature-name>Layout.tsx` (using camelCase, e.g., `activityManagementLayout.tsx`), compose your sub-components and inject feature-specific state/controllers:
```tsx
"use client";

import { Box } from "@mantine/core";
import { useActivityManagementQueries } from "./shared/useActivityManagementQueries";
import { ActivityManagementHeader } from "./components/ActivityManagementHeader";
import { ActivityManagementTable } from "./components/ActivityManagementTable";

export default function ActivityManagementLayout() {
  const { data, isLoading } = useActivityManagementQueries();

  return (
    <Stack gap="md">
      <ActivityManagementHeader />
      <ActivityManagementTable data={data} isLoading={isLoading} />
    </Stack>
  );
}
```

### 5. Hook up the Route
Create `apps/sae-new/src/app/<route-path>/page.tsx` (using **kebab-case** for page routes inside the routing layer, e.g., `apps/sae-new/src/app/operation/activity-management/page.tsx`) and render the feature layout:
```tsx
import ActivityManagementLayout from "@/features/operation/activityManagement/activityManagementLayout";

export default function Page() {
  return <ActivityManagementLayout />;
}
```

## Best Practices and Conventions

- **"use client"**: Ensure `"use client";` is added to the top of all feature layouts and interactive sub-components.
- **Responsive design**: Use `hookController.isMobile` or Mantine's responsive hooks to conditionally scale side panels (e.g., switching panel width from `340px` to `100%`).
- **Permissions**: When adding action buttons, use `<CustomButton actionType="create" />` from `@aq-fe/core-ui` to handle automatic role-based permission visibility natively.
- **Form Abstractions**: When building creation or editing components, do not construct manual modals or toggle states. Instead, utilize the centralized `<CustomButtonCreateUpdate>` wrapper.
  - **IMPORTANT**: You MUST ALWAYS import `<CustomButtonCreateUpdate>` from `@aq-fe/aq-core-framework` (never from `@aq-fe/core-ui`). Using the framework version is mandatory because it natively integrates with the Backend/Axios API ecosystem.
  - **Automated Validation Binding**: The framework wrapper natively intercepts HTTP `400 Bad Request` problem details from Axios, automatically parses `.response.data.errors`, maps them to individual fields via `form.setFieldError(...)`, and shows a unified notification toast. 
  - **Pure Submits**: Do not write manual `try/catch` boilerplate inside form submission handlers (`onSubmit`). Simply map parameters and directly return the Axios Promise (e.g., `return service.create(payload);`). All success/error parsing is handled automatically.
- **No Modal-Button Separation (Unified Trigger Components)**: Do NOT separate a trigger Button/Icon and its corresponding Modal/Drawer into separate files or separate component exports.
  - **Unified Integration**: Always use unified components (like `CustomButtonModal`, `<CustomButtonCreateUpdate>`, or your own unified feature components such as `CommunityCriteriaCreateUpdate`) that house both the button trigger and the modal internally within a single file.
  - **Table Render Support**: Use these unified components even when they are rendered inside a table (e.g., as row actions inside a custom table cell or toolbar).
  - **Benefits**: This keeps the code extremely readable, clean, easy to maintain, and strictly follows the "1 component, 1 file" architectural principle where each module has its own self-contained responsibility.
- **Extract Action Buttons (Mandatory)**: Action buttons (especially those containing complex mutation logic, API calls like Delete, Start, or Approve, or those using `CustomButtonModal`) inside tables or layout components MUST be extracted into their own separate component files (e.g., `ActivityListDeleteButton.tsx`, `AttendanceDetailCheckInButton.tsx`) and imported. Do NOT write API mutation logic, `CustomButtonModal` wrappers, or complex action components inline inside the main table component (`*Table.tsx` or `*Modal.tsx`) to keep the parent file clean, maintainable, and avoid React Hook errors in render functions.
- **Core Framework Reusable Wrappers (Single Responsibility Design)**: When encapsulating API behaviors (like deletion or data mutators) into reusable framework components, strictly adhere to Single Responsibility Principle (SRP) to maximize readability and avoid conditional bloat.
  - **Granular Specialization (Single vs List)**: Do NOT build multi-purpose components that branch based on heavy conditionals (e.g., avoiding `isBulk` flags). Instead, split them into atomic, distinct files:
    - Use `CustomButtonDelete` for single-row deletions (expecting a singular string `id` and rendering as a specialized ActionIcon).
    - Use `CustomButtonDeleteList` for multi-selection table toolbars (expecting an array `ids` and rendering as a primary Button).
  - **Pure API Injection (Service Functions)**: Design shared framework wrappers to accept clean backend API services as props (e.g., `deleteFn` or `deleteListFn`). Features consuming these components must NOT declare manual `useMutation` hooks or notification alerts; the framework wrapper MUST internally bind `useCustomReactMutation` and handle the full lifecycle transparently.
- **Component Naming (Mandatory Module Prefix)**: Component file names, React component exports, and local layout files MUST use **PascalCase** for UI components/modals and ALWAYS start with the full PascalCase feature/module prefix (e.g., all component files inside the `activityManagement` feature MUST be prefixed with `ActivityManagement`, resulting in `ActivityManagementTable.tsx`, `ActivityManagementCreateUpdate.tsx`). Generic names (like `GeneralInformationTab.tsx` or `RoleModal.tsx`) are STRICTLY FORBIDDEN. The feature layout entry point MUST be in **camelCase** matching the feature directory (e.g., `activityManagementLayout.tsx`). All component names must match the module prefix exactly to prevent namespace collision and ensure clean code boundaries.
- **Scrollbars**: Follow custom scrollbar standards using standard webkit styling for lists and panels:
  ```css
  ::-webkit-scrollbar { width: 5px; }
  ::-webkit-scrollbar-track { background: #F3F0EA; }
  ::-webkit-scrollbar-thumb { background: #C5BEB4; border-radius: 99px; }
  ```
- **Readonly Identity Fields**: During an Update operation (`isUpdate={true}`), the `Mã` (Code) or primary identifier input field MUST ALWAYS be disabled or set to `readOnly`. Never allow the user to modify unique codes after initial record creation to prevent accidental primary key synchronization drifts.
- **Routing BasePath Precedence**: Projects utilize specific application `basePath` prefixes configured in `next.config.ts` (e.g., `/sae-develop` sourced from `APP_CONFIG.alias`). When testing, navigating, or performing browser automation via subagents, you MUST ALWAYS prepend this `basePath` to your target URL (e.g., `http://localhost:3000/sae-develop/activity-management`).
- **Prototype Menu Flags (`isPrototype`)**: Whenever the user instructs to "implement only the UI" or "develop the UI first" ("làm trước UI"), you MUST ALWAYS locate the corresponding feature menu configuration (usually declared in `src/app/.../layout.tsx`) and append the property `isPrototype: true` to the menu item. This ensures the system properly flags the view as an unlinked/prototype state visually.
- **Prototype Mock Models (Local Scope)**: When building the UI first (isPrototype mode), you MUST place any mock enums and interfaces inside the feature's local `shared/` directory (e.g., `src/features/<featureName>/shared/`). Do NOT place them in the global `src/shared/` directory. Only after integrating the API and receiving the official backend models should you move them to the global `shared/consts/enum/` and `shared/interfaces/` folders.
- **Flatten Sidebar URLs (1-Level Routes)**: Avoid creating multi-level nested URL structures for sidebar links (e.g., do not use `/operation/planning/approve`). Instead, flatten the routes to exactly one level under the main module path (e.g., `/operation/planning-approve`). This structure eliminates prefix overlapping conflicts inside the `CustomAppShell` navigation sidebar component and keeps the layout predictable.
- **Layout Architecture & Atomic Queries (No Global Queries)**: Keep Layout components thin and move state and query logic into independent, encapsulated sub-components.
  - **Container Layouts**: Main layout files (e.g., `activityManagementLayout.tsx` inside features directory `activityManagement`) MUST only act as structural shell containers, simply importing and rendering the core feature table component (e.g., `ActivityManagementTable.tsx`). Do not write API hooks or complex local states here.
  - **No Hardcoded Wrappers**: Layout components MUST NOT wrap their contents in unnecessary `<Box>` containers with hardcoded styling (e.g., `height: "100vh"`, `p="md"`, or `overflowY: "auto"`). The global `CustomAppShell` already provides default structural padding and scroll behavior. Use `<Stack>` (with appropriate gap as needed) to stack the layout children.
  - **Self-Contained Sub-components**: Break down functionality into independent files (e.g., `ActivityManagementTable.tsx`, `ActivityManagementDeleteModal.tsx` inside components/modal subdirectories). Each sub-component must independently manage its own reactive state and handle its own API integrations.
  - **Inline React Query**: DO NOT create shared global queries files (e.g., remove files like `useActivityTypeQueries.ts`). Write `useQuery` and `useMutation` hooks directly inline within the components that consume them to ensure clear scope and boundaries.
  - **Implicit Global Cache Syncing**: Leverage React Query's global caching rather than passing manual mutation callback chains (like `onSuccessAction`). Let independent mutating components (like Delete/Update modals) simply call `queryClient.invalidateQueries(...)` on success to trigger automated, zero-boilerplate re-fetches across all sibling data consumers.
- **Data Structures Separation (Interfaces vs. Enums)**: Always strictly separate data interfaces from runtime enums to keep models pure and highly modular.
  - **Pure Interfaces**: Keep files inside `src/shared/interfaces/` dedicated strictly to TypeScript contracts (e.g., `CommunityType.ts`). Do NOT declare any logic, companion maps, or Enums inside these files.
  - **Enum Labels (No Switch Cases)**: Do NOT use inline `switch-case` statements or helper functions like `getStatusString(state)` to map enum numeric values to strings inside React components. Always define an explicit `Record<number, string>` label map alongside the Enum definition inside `src/shared/consts/enum/` (e.g., `ATTENDANCE_STATE_LABEL`) and use it directly (e.g., `ATTENDANCE_STATE_LABEL[state]`).
  - **Enum Consts**: All enums and their presentation metadata objects MUST be declared inside dedicated files under `src/shared/consts/enum/`.
  - **Numeric Enum Values (Mandatory)**: Enum values must always be defined as numbers (e.g., `1, 2, 3`), NEVER strings (e.g., `"ALL", "DEPARTMENT"`), even when creating temporary mock enums. This ensures seamless integration with the backend which strictly uses integer enums.
  - **Suffix Naming Conventions**: Always follow the project's suffix standard for naming enums and their companion map files and exports:
    - **File Name**: `[Name]Enum.ts` (e.g., `ScoreMechanismEnum.ts`)
    - **Enum declaration**: `[Name]Enum` (e.g., `ScoreMechanismEnum`)
    - **Label map**: `[Name]Labels` (e.g., `ScoreMechanismLabels`)
    - **Color map**: `[Name]Colors` (e.g., `ScoreMechanismColors`)
    - **Icon map**: `[Name]Icons` (e.g., `ScoreMechanismIcons`)
  - **Example and Pattern**:
    ```typescript
    // Path: src/shared/consts/enum/ScoreMechanismEnum.ts
    import { IconCpu, IconEdit, IconQrcode, type TablerIcon } from "@tabler/icons-react";
    export enum ScoreMechanismEnum { Qr = 1, Auto = 2, Manual = 3 }
    export const ScoreMechanismLabels: Record<ScoreMechanismEnum, string> = {
        [ScoreMechanismEnum.Qr]: "Tự động (QR)",
        [ScoreMechanismEnum.Auto]: "Tự động theo Role",
        [ScoreMechanismEnum.Manual]: "Phụ trách nhập (Thủ công)",
    };
    export const ScoreMechanismColors: Record<ScoreMechanismEnum, string> = {
        [ScoreMechanismEnum.Qr]: "blue",
        [ScoreMechanismEnum.Auto]: "teal",
        [ScoreMechanismEnum.Manual]: "gray",
    };
    export const ScoreMechanismIcons: Record<ScoreMechanismEnum, TablerIcon> = {
        [ScoreMechanismEnum.Qr]: IconQrcode,
        [ScoreMechanismEnum.Auto]: IconCpu,
        [ScoreMechanismEnum.Manual]: IconEdit,
    };
    ```
- **JSX Syntax Standard for Children**: Do NOT pass textual children explicitly via properties (e.g., `<CustomButton children="Label" />`). You MUST always use standard nested JSX tags (e.g., `<CustomButton>Label</CustomButton>`) to keep React code readable, clean, and uniformly formatted.
- **Entity Interface Separation (No Inline Objects)**: NEVER define complex, nested object shapes (like `semester?: { id?: string, name?: string }`) inline directly inside a primary domain interface. Instead, always extract the nested structure into its own dedicated, exported `interface` file inside `src/shared/interfaces/` and reference it via explicit `import` statements to promote reuse, consistency, and dry code structures across the application.
- **Avoid Redundant Page Headers**: When implementing a page UI that consists primarily of a `CustomTanstackTable`, do NOT wrap the table in a container that manually adds page titles (e.g., `<Title>`) or `<Breadcrumbs>`. The global `CustomAppShell` and the Table's own `title` prop already provide sufficient context.
- **Action Buttons Location**: Always place primary action buttons (like "+ Khai báo mới", "Thêm mới", "Tạo mới") inside the table's `renderTopToolbarCustomActions` prop rather than floating them in a separate page header.


## Static Build & Monorepo Deployment Configuration

When deploying applications to static hosts (e.g., IIS) in a monorepo setting, strictly observe the following structural requirements to avoid broken assets or bundle crashes:

1. **Register `transpilePackages`**: Next.js `output: 'export'` ignores shared local workspace CSS modules by default. You MUST register internal libraries in `next.config.ts` using the `transpilePackages: ["@aq-fe/core-ui", "@aq-fe/aq-core-framework"]` array to ensure shared components transpile and output CSS successfully.
2. **Relative Config Pathing**: Next.js processes `next.config.ts` BEFORE TypeScript alias mappings hydrate fully. NEVER use `@/` aliases inside the config file. Always utilize absolute relative paths (e.g., `import { APP_CONFIG } from "./src/shared/configs/appConfig"`).
3. **Synchronize `web.config` rewrite rules**: Before building, always audit the template inside `/public/web.config`. Ensure hardcoded alias identifiers (like `srm`) are fully swapped with the active application target alias (like `sae-develop`) inside `<rule name="...">` tags, guaranteeing IIS precisely maps rewritten requests on refresh.
4. **Strict Dependency Locking**: Whenever you install or add a new third-party library to `package.json`, you MUST ALWAYS remove the caret (`^`) or tilde (`~`) prefix from the version number (e.g. use `"react-photo-view": "1.2.7"` instead of `"^1.2.7"`). This enforces strict version locking to prevent unexpected breaking changes across environments in the monorepo.
