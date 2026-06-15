---
name: typescript-module-augmentation
description: Guide on TypeScript Module Augmentation technique to synchronize data types (Type-safe) from the Application (App) down to the shared library (Framework) without writing wrapper files.
---

# TypeScript Module Augmentation Technique (Dynamic Type Casting for Frameworks)

In the Monorepo architecture, shared libraries (packages) such as `@aq-fe/aq-standard-framework` are written as Generics so they can run on any project. We are not allowed to import specific App configurations (like `features.ts`) backwards into the Framework.
However, to provide developers in Apps with the best coding experience (VS Code automatically suggesting IntelliSense for Components/Hooks from the Framework), we **ABSOLUTELY MUST NOT** create cumbersome "wrapper" directories. Instead, use **Module Augmentation**.

### Standard Implementation Structure:

#### Step 1: At the shared library (Framework)
Declare an empty `interface` to act as a "socket", and a dynamic `Type` that automatically fallbacks to string if no one plugs into it:
```typescript
// Example at: packages/aq-standard-framework/src/shared/features/FeatureFlagContext.tsx

// 1. Empty interface waiting for apps to plug types into
// eslint-disable-next-line @typescript-eslint/no-empty-object-type
export interface FeatureFlagRegistry {}

// 2. Automatically get the list of keys if someone plugs in, otherwise return the default string type
export type FeatureName = keyof FeatureFlagRegistry extends never ? string : keyof FeatureFlagRegistry;

// 3. Use dynamic Type for functions/components
export function useFeatureFlag(featureName: FeatureName): boolean { ... }
export interface FeatureGuardProps { feature: FeatureName; }
```

#### Step 2: At the Application (App)
In the application's type definition file, use `declare module` to directly "inject" the App's Type into the Framework's empty "socket":
```typescript
// Example at: apps/spm/src/shared/types/features.ts
export interface AppFeatures {
    enableAdvancedExport?: boolean;
    enableFaceDetect?: boolean;
}

// Inject AppFeatures type into the base library
declare module '@aq-fe/aq-standard-framework/shared/features/FeatureFlagContext' {
    export interface FeatureFlagRegistry extends AppFeatures {}
}
```

### Output Standards:
- Developers at the App import Components directly from the Framework (`import { FeatureGuard } from '@aq-fe/aq-standard-framework...'`).
- VS Code immediately suggests the correct list of variables defined in `AppFeatures` without the need for redundant code.
