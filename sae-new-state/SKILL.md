---
name: sae-new-state
description: Directs state management patterns using Zustand in the SAE-new project. Use when designing, updating, or syncing feature-specific and global states across components, side-effects, or local storage.
---

# SAE-New State Management Skill

This skill outlines how to design, create, and interact with global and feature-specific stores using `zustand` in the `sae-new` project.

## Global Stores vs. Feature Stores

- **Global Stores** (`apps/sae-new/src/shared/stores/`): Manage global states like user session, authentication, and global UI preferences.
  - Example: `authenticate-store` houses the active JWT token, user info, and role permissions, persisted in `localStorage`.
- **Feature Stores** (`apps/sae-new/src/features/<feature-name>/shared/` where `<feature-name>` is in camelCase, e.g., `apps/sae-new/src/features/activityManagement/shared/`): Keep local user edits, active selections, sidebar panel visibility, or complex form states encapsulated within a specific feature.
  - Example: `useActivityManagementStore` manages active lists, selections, and unsaved edits inside the feature.

## Zustand Implementation Patterns

Zustand stores in this project follow a structured type-safe model using TypeScript.

### Creating a Feature Store

Create a store file (e.g., `activityManagementStore.ts` inside the `shared/` directory of the camelCased feature folder):
```typescript
import { create } from "zustand";

interface ActivityManagementState {
  // State variables
  selectedId: string | null;
  leftOpen: boolean;
  expandedIds: Set<string>;
  
  // Actions/Mutations
  setSelectedId: (id: string | null) => void;
  setLeftOpen: (open: boolean) => void;
  setExpandedIds: (fn: (prev: Set<string>) => Set<string>) => void;
  reset: () => void;
}

export const useActivityManagementStore = create<ActivityManagementState>((set) => ({
  selectedId: null,
  leftOpen: true,
  expandedIds: new Set<string>(),

  setSelectedId: (id) => set({ selectedId: id }),
  setLeftOpen: (open) => set({ leftOpen: open }),
  setExpandedIds: (fn) => set((state) => ({ expandedIds: fn(state.expandedIds) })),
  reset: () => set({ selectedId: null, leftOpen: true, expandedIds: new Set() }),
}));
```

### Accessing Store State (Selectors Pattern)

To avoid unnecessary component re-renders, always use selector-based accessors instead of reading the whole store object:

```tsx
import { useActivityManagementStore } from "@/features/operation/activityManagement/shared/activityManagementStore";

export function LeftPanel() {
  // CORRECT: Component re-renders ONLY when leftOpen changes
  const leftOpen = useActivityManagementStore((state) => state.leftOpen);
  const setLeftOpen = useActivityManagementStore((state) => state.setLeftOpen);

  // INCORRECT (Causes unnecessary re-renders when other states change):
  // const store = useActivityManagementStore();
}
```

### Syncing Store with API Queries

When fetching complex trees or list structures, synchronize the queried server-state with the Zustand store inside a `useEffect` inside a dedicated controller or custom query hook (e.g., `useActivityManagementQueries.ts`):

```typescript
useEffect(() => {
  const data = listQuery.data;
  if (!data) return;

  const s = useActivityManagementStore.getState(); // Non-reactive read inside effect
  const merged = [...s.versions, ...data];
  
  s.setVersions(merged);
}, [listQuery.data]);
```

## Persisted Stores (`localStorage`)

For stores whose states need to persist across page reloads (like authentication), utilize Zustand's `persist` middleware:

```typescript
import { create } from "zustand";
import { persist } from "zustand/middleware";

export const useAuthenticateStore = create(
  persist(
    (set) => ({
      token: null,
      user: null,
      setToken: (token) => set({ token }),
      setUser: (user) => set({ user }),
    }),
    {
      name: "authenticate-store", // localStorage key
    }
  )
);
```

### Accessing Persisted Store outside React (e.g., Axios Interceptors)

You can read or update Zustand store states outside the React lifecycle using `.getState()`:
```typescript
const storeStr = localStorage.getItem("authenticate-store");
if (storeStr) {
  const storeObj = JSON.parse(storeStr);
  const token = storeObj?.state?.token;
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
}
```

## Best Practices
1. **Always clean up on unmount**: If a feature-specific store persists state across sessions, trigger a `reset()` inside the layout's `useEffect` cleanup return function.
2. **Batch mutations**: Combine multiple updates into a single store action to minimize component renderings.
3. **Selector efficiency**: Avoid creating new arrays or object instances inside inline selectors to prevent infinite render loops.
