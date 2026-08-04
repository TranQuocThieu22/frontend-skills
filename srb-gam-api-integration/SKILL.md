---
name: srb-gam-api-integration
description: Guides the implementation of API services, axios requests, React Query data fetching hooks, and unified API-UI data models in the SRB and GAM projects.
---

# SRB & GAM API Integration Skill

This skill explains how to build and maintain API services, manage server-state using `@tanstack/react-query` and `useCustomReactQuery`, and directly bind backend entities to user interface form models inside the `srb` and `gam` projects.

> [!NOTE]
> **Backend Repository Path**: The backend source code for the SRB and GAM projects is located locally at `D:\AQ-Project\Source.NET`.
>
> - **CRITICAL RULE**: Before reading or analyzing any backend code in `D:\AQ-Project\Source.NET`, you **MUST** run `git pull` or `git fetch` (using the `run_command` tool) in that directory to ensure you are working with the latest codebase.
> - When integrating new APIs or if there is any ambiguity about the request/response schema, actively check the backend controllers, DTOs, and services in that directory to ensure exact alignment.

## API Services Directory (`apps/srb/src/shared/APIs/` and `apps/gam/src/shared/APIs/`)

All API services must be declared as distinct files inside the `src/shared/APIs/` folder of the respective app (`apps/srb/src/shared/APIs/` or `apps/gam/src/shared/APIs/`). They use `axiosInstance` imported from `@/shared/configs/axiosInstance` and optional base API helpers like `createBaseAPI` from `@aq-fe/aq-core-framework/shared/libs/createBaseAPI`.

### Writing an API Service

> [!WARNING]
> **CRITICAL RULE: DO NOT PREFIX WITH `/api/`**: The `axiosInstance` already configures the base URL with `/api/` automatically. When defining `CONTROLLER` in your API service, you **MUST NOT** include `/api/` in the path (e.g. use `const CONTROLLER = "/RoomBooking";` instead of `const CONTROLLER = "/api/RoomBooking";`). Failure to do so will result in duplicated `/api/api/...` URL errors.

```typescript
import axiosInstance from "../configs/axiosInstance";
import { CustomAPIResponse } from "@aq-fe/aq-core-framework/shared/interfaces/CustomAPIResponse";
import { createBaseAPI } from "@aq-fe/aq-core-framework/shared/libs/createBaseAPI";

const CONTROLLER = "/myController";

export const myService = {
  // Inherit standard CRUD endpoints (getById, create, update, delete)
  ...createBaseAPI<MyModel>(CONTROLLER, axiosInstance),

  // Implement custom endpoints
  getCustomData: (tenantId: string, pageNumber: number) => {
    return axiosInstance.get<CustomAPIResponse<MyModel[]>>(
      `${CONTROLLER}/${tenantId}/custom`,
      { params: { PageNumber: pageNumber } },
    );
  },
};
```

## Data Fetching with `useCustomReactQuery`

The project utilizes `useCustomReactQuery` from `@aq-fe/aq-core-framework/shared/hooks/useCustomReactQuery` to wrap standard `@tanstack/react-query`'s `useQuery`.

> [!IMPORTANT]
> **Mandatory Usage for Error Handling**: You MUST use `useCustomReactQuery` and `useCustomReactMutation` instead of raw `@tanstack/react-query` hooks for all backend API interactions to automatically handle standard backend errors and exceptions.

### Custom Queries Implementation Pattern

**CRITICAL RULE**: Do NOT create separate `hooks/` folders or separate `use*Queries.ts` files for features. Instead, whichever component needs to query the API should declare the `useCustomReactQuery` directly inside that component's file.

### Data Mapping Caution (Axios + React Query)

> [!WARNING]
> **BE CAREFUL WITH `.data` NESTING**: If your API service returns `axiosInstance.post(...).then(res => res.data)`, the payload returned to React Query is ALREADY the body of the response (e.g. `CustomAPIResponse`).
> When you destructure from `useQuery`:
> `const { data: myDataRes } = useQuery(...)`
> The variable `myDataRes` IS the API response body. DO NOT accidentally chain `.data.data` (e.g., `myDataRes.data.data`) unless the backend response actually wraps the data twice. This is a common bug that results in `undefined` values.

## Unified API and UI Model Pattern

To maximize maintainability and reduce code complexity, **do not build translation mappings back and forth** between the frontend UI structures (form states, table columns) and the backend API models.

### Core Guidelines:

1. **Direct Interface Binding**: Ensure that the Mantine `useForm` values, TypeScript interfaces, and component properties utilize the exact backend API entity and request models directly.
   - **CRITICAL**: Khi định nghĩa cột bảng (Table Columns) bằng `accessorKey` hoặc các trường dữ liệu trên UI, **PHẢI sử dụng chính xác tên field của backend**. Tuyệt đối **không tạo ra các props riêng trên UI** rồi map lại thủ công từ field của backend.
2. **Handle Backend Defaults Dynamically**: Compute necessary derived fields in the form's `onSubmit` payload without altering the form's underlying model structure.
3. **Preserve UI Prototypes (CRITICAL & MANDATORY)**: If the backend API is missing some fields or features that are present in the pre-existing UI prototype, **DO NOT delete the UI fields or disable them**.
   - Keep the UI fields fully operational on the client side (e.g., by extending the backend TypeScript interface locally).
   - Omit the missing fields from the API request payloads during submission, or let the backend ignore them.
   - **Mandatory Deliverable (API Status Tracker)**: Every time an API integration is developed, you **MUST** create an `API_STATUS_PENDING.md` or `API_STATUS_DONE.md` report directly inside the target feature folder tracking progress and UI/API gaps.
   - **Visual Indicator for Missing Fields (Icon + Tooltip)**: Khi tích hợp API, nếu giao diện (UI) có trường dữ liệu (field) nhưng backend không có field đó, bạn **bắt buộc** phải hiển thị rõ icon cảnh báo (ví dụ icon warning dấu chấm than) kèm theo tooltip giải thích trên giao diện, đồng thời **phải note rõ** vấn đề này vào file báo cáo tích hợp (API_STATUS).

## General Integration Notes for SRB & GAM

- Similar to other apps, implement server-side pagination, search, and filtering wherever possible.
- All date filtering payload parameters should be formatted correctly to UTC.
- Mock prototypes must have the `isPrototype` badge removed from the menu configuration in `layout.tsx` or `routes.config.tsx` once fully integrated.
