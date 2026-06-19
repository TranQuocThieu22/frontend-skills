---
name: sae-new-api-integration
description: Guides the implementation of API services, axios requests, React Query data fetching hooks, and unified API-UI data models in the SAE-new project.
---

# SAE-New API Integration Skill

This skill explains how to build and maintain API services, manage server-state using `@tanstack/react-query` and `useCustomReactQuery`, and directly bind backend entities to user interface form models inside the `sae-new` project.

> [!NOTE]
> **Swagger API Documentation & System Architecture**: 
> - **SAE-New API** (`https://hst.aqtech.io.vn/sae-dev/swagger/index.html`): Đây là dự án nghiệp vụ quản lý riêng biệt các hoạt động cộng đồng, ngoại khóa.
> - **IAM API** (`https://hst.aqtech.io.vn/iam-dev/swagger/index.html`): Hệ thống trung gian quản lý quyền và các thông tin dùng chung như User, Role, Đơn vị (Phân quyền).
> 
> Khi cần gọi API về User hay cấu hình hệ thống chung thì gọi IAM, còn dữ liệu nghiệp vụ hoạt động thì gọi SAE-New. Always refer to these URLs when you need to read or integrate API endpoints.
> 
> **Backend Repository Path**: The backend source code is located locally at `D:\AQ-Project\SAE-Backend`. 
> - **CRITICAL RULE**: Before reading or analyzing any backend code in `D:\AQ-Project\SAE-Backend`, you **MUST** run `git pull` or `git fetch` (using the `run_command` tool) in that directory to ensure you are working with the latest codebase.
> - When integrating new APIs or if there is any ambiguity about the request/response schema, actively check the backend controllers, DTOs, and services in that directory to ensure exact alignment.

## API Services Directory (`apps/sae-new/src/shared/APIs/`)

All API services must be declared as distinct files inside the `src/shared/APIs/` folder. They use `axiosInstance` imported from `@/shared/configs/axiosInstance` and optional base API helpers like `createBaseAPI` from `@aq-fe/aq-core-framework/shared/libs/createBaseAPI`.

### Writing an API Service

Here is the standard pattern for an API service:

> [!WARNING]
> **CRITICAL RULE: DO NOT PREFIX WITH `/api/`**: The `axiosInstance` already configures the base URL with `/api/` automatically. When defining `CONTROLLER` in your API service, you **MUST NOT** include `/api/` in the path (e.g. use `const CONTROLLER = "/LearningOutcomeProgress";` instead of `const CONTROLLER = "/api/LearningOutcomeProgress";`). Failure to do so will result in duplicated `/api/api/...` URL errors.

```typescript
import axiosInstance from "../configs/axiosInstance";
import { CustomAPIResponse } from "@aq-fe/aq-core-framework/shared/interfaces/CustomAPIResponse";
import { createBaseAPI } from "@aq-fe/aq-core-framework/shared/libs/createBaseAPI";
import { MyModel } from "@/shared/interfaces/MyModel";

const CONTROLLER = "/myController";

export const myService = {
  // Inherit standard CRUD endpoints (getById, create, update, delete)
  ...createBaseAPI<MyModel>(CONTROLLER, axiosInstance),

  // Implement custom endpoints
  getCustomData: (tenantId: string, pageNumber: number) => {
    return axiosInstance.get<CustomAPIResponse<MyModel[]>>(
      `${CONTROLLER}/${tenantId}/custom`,
      { params: { PageNumber: pageNumber } }
    );
  },
};
```

## Data Fetching with `useCustomReactQuery`

The project utilizes `useCustomReactQuery` from `@aq-fe/aq-core-framework/shared/hooks/useCustomReactQuery` to wrap standard `@tanstack/react-query`'s `useQuery`.

> [!IMPORTANT]
> **Mandatory Usage for Error Handling**: You MUST use `useCustomReactQuery` and `useCustomReactMutation` instead of raw `@tanstack/react-query` hooks for all backend API interactions (unless explicitly stated otherwise, e.g. for shared dropdowns). These custom hooks contain centralized logic to automatically handle standard backend errors and exceptions, ensuring consistent error reporting across the entire application without needing to manually write `try/catch` or `onError` blocks everywhere. Even if you are mocking data locally, still use `useCustomReactQuery` and pass a mocked Promise into the `serviceFn` (returning an object mimicking `AxiosResponse<CustomAPIResponse<T>>`) so that when the real backend API is ready, you only need to swap out the `serviceFn` seamlessly.

### Custom Queries Implementation Pattern

**CRITICAL RULE**: Do NOT create separate `hooks/` folders or separate `use*Queries.ts` files for features. Instead, whichever component needs to query the API should declare the `useCustomReactQuery` directly inside that component's file.

For example, inside `MyFeatureTable.tsx`:
```typescript
"use client";

import { useCustomReactQuery } from "@aq-fe/aq-core-framework/shared/hooks/useCustomReactQuery";
import { myService } from "@/shared/APIs/myService";
import { MAIN_TENANT_ID } from "@/shared/consts/data/mainTenantId";

export function MyFeatureTable() {
  const [selectedId, setSelectedId] = useState<string | null>(null);

  const listQuery = useCustomReactQuery({
    queryKey: ["my-feature-list", MAIN_TENANT_ID],
    serviceFn: () =>
      myService.getCustomData(MAIN_TENANT_ID, 1),
    refetchOnMount: false,
    refetchOnWindowFocus: false,
    refetchOnReconnect: false,
  });

  const detailQuery = useCustomReactQuery({
    queryKey: ["my-feature-detail", selectedId],
    serviceFn: () =>
      selectedId ? myService.getById(MAIN_TENANT_ID, selectedId) : Promise.resolve(null),
    enabled: !!selectedId,
    refetchOnMount: false,
  });

  return (
    // ...render table using listQuery.data
  );
}
```

## Data Mutations with `useCustomReactMutation`

Always use `useCustomReactMutation` from `@aq-fe/aq-core-framework/shared/hooks/useCustomReactMutation` instead of calling raw `@tanstack/react-query`'s `useMutation` directly.

> [!NOTE]
> **Automatic Notifications**: When using `useCustomReactMutation`, **you do not need to write `onError` or call manual `notifications.show` toasts**. Based on the `mutationType` passed (e.g., `"create"`, `"update"`, `"delete"`), the framework's hook automatically triggers consistent success or error notifications with beautiful styling under the hood.

### Custom Mutations Implementation Pattern

**CRITICAL RULE**: Do NOT create separate `hooks/` folders or separate `use*Mutations.ts` files. Instead, declare the `useCustomReactMutation` directly inside the component that triggers the mutation (e.g. inside the Create/Update form component or the Table component for deletes).

For example, inside `MyFeatureForm.tsx`:

```typescript
import { useCustomReactMutation } from "@aq-fe/aq-core-framework/shared/hooks/useCustomReactMutation";
import { useQueryClient } from "@tanstack/react-query";
import { myService } from "@/shared/APIs/myService";

export function MyFeatureForm() {
  const queryClient = useQueryClient();

  const createMutation = useCustomReactMutation({
    mutationType: "create", // Required: "create" | "update" | "delete" etc. to trigger auto-notifications
    serviceFn: (newData: any) => myService.create(newData),
    onSuccess: () => {
      // Invalidate queries to fetch updated lists automatically
      queryClient.invalidateQueries({ queryKey: ["my-feature-list"] });
    },
  });

  return (
    // ...render form and use createMutation.mutateAsync() on submit
  );
}
```

> [!TIP]
> **Special Cases for Framework Buttons (`<CustomButtonCreateUpdate>`, `<CustomButtonDelete>`, etc.)**: 
> If you are using the `<CustomButtonCreateUpdate>` or `<CustomButtonDelete>` component from the core framework to build your forms or delete actions, you do **NOT** even need to declare a `useCustomReactMutation` hook inside your component. The button components automatically manage the mutation lifecycle, loading states, and notifications internally. You simply pass an `onSubmit` or `deleteFn` function that directly returns your API service Promise, and use `customReactMutationProps={{ onSuccess: () => ... }}` (for CreateUpdate) or handle cache invalidation inside the `deleteFn` (or via its internal hook mechanism) to invalidate queries.

> [!IMPORTANT]
> **Extract Action Components**: Always extract individual action components (like Create/Update modals, Delete buttons, Approve buttons, etc.) into their own separate files (e.g. `MyFeatureDeleteButton.tsx`, `MyFeatureCreateUpdate.tsx`). Do **NOT** inline `<CustomButtonDelete>` or large form components directly inside the Table component's `renderRowActions`. Each component should have exactly one responsibility. This keeps the Table component clean and prevents unnecessary re-renders.

## Unified API and UI Model Pattern

To maximize maintainability and reduce code complexity, **do not build translation mappings back and forth** between the frontend UI structures (form states, table columns) and the backend API models.

### Core Guidelines:
1. **Direct Interface Binding**: Ensure that the Mantine `useForm` values, TypeScript interfaces, and component properties utilize the exact backend API entity and request models directly.
    * **CRITICAL**: Khi định nghĩa cột bảng (Table Columns) bằng `accessorKey` hoặc các trường dữ liệu trên UI, **PHẢI sử dụng chính xác tên field của backend** (ví dụ backend trả về `reason` thì dùng `accessorKey: "reason"`). Tuyệt đối **không tạo ra các props riêng trên UI** rồi map lại thủ công từ field của backend.
    * Nếu backend chưa có field đó mà UI thiết kế yêu cầu phải có, hãy tham khảo quy tắc số 4 bên dưới để note lại.
2. **Eliminate Mappers**: Do not write `mapApiToUi` or `mapUiToRequest` helpers, and **DO NOT remap fields during form loading or saving** (e.g., inside `useEffect` calling `form.setValues()`). Bind input props directly using backend schema fields (e.g. use `code` instead of `clause`, `isExternal` instead of `isOutside`, and nested API arrays like `subCriterias` instead of mock UI collections like `pointSteps`). For example, if the backend returns `communityCriteriaId`, the form state must use `communityCriteriaId`, NOT `criteriaId`.
3. **Handle Backend Defaults Dynamically**: If the backend requires fields that aren't manually entered by the user (like calculated maximum values), compute them directly in the form's `onSubmit` payload without altering the form's underlying model structure.
4. **Preserve UI Prototypes (CRITICAL & MANDATORY)**: If the backend API is missing some fields or features that are present in the pre-existing UI prototype, **DO NOT delete the UI fields or disable them**. Instead:
    * Keep the UI fields fully operational on the client side (e.g., by extending the backend TypeScript interface locally).
    * Omit the missing fields from the API request payloads during submission, or let the backend ignore them.
    * **Mandatory Deliverable (API Status Tracker)**: Every time an API integration is developed, you **MUST** create an integration progress and status report directly inside the target feature folder (e.g., `apps/sae-new/src/features/<feature-name>/`). The filename must be dynamically named based on the integration completeness status:
      - **`API_STATUS_PENDING.md`**: Use this when there are still unresolved gap analysis items, missing backend APIs/fields, or **if any part of the UI/feature is still using mock data** (e.g. dropdown lists, units, semesters, criteria that are still static constants). If mock data is used, the integration is considered NOT completed (chưa hoàn thành) and MUST use `API_STATUS_PENDING.md`.
      - **`API_STATUS_DONE.md`**: Use this ONLY once all frontend integration is finished, all backend APIs are fully compliant, **absolutely no mock data is used**, and the feature is 100% complete and verified with real database connections.
    * **Visual Indicator for Missing Fields (Alerts Placement)**: When keeping UI fields or filters that the backend does not yet support, you MUST add a visual indicator. 
      - If it's a missing column in a `CustomTanstackTable`, use the **`unsupportedTooltip`** property directly in the column definition (e.g., `unsupportedTooltip: "Backend hiện chưa hỗ trợ trường này"`). This automatically renders the correct warning icon on the column header. Do NOT manually construct a React Node with icons for the `header` property to avoid React child function rendering errors.
      - If it's a missing filter for a list/table, place a full-width `<Alert>` block **ABOVE** the primary content.
      - If it's a specific missing field inside a form, add a `<Tooltip>` with an orange `<IconExclamationCircle>` next to the input's label.
    * **Handling Missing Backend Fields (Khi thiếu field từ backend)**: You must simultaneously do 3 things: (1) Add a warning Alert/Tooltip on the UI form for that field, (2) Add a `// FIXME: Missing from Backend` comment in the TypeScript model/interface, and (3) Log the missing field into the `API_STATUS_PENDING.md` file.
      ```tsx
      <CustomTextInput
          label={
              <Group gap="xs">
                  Hình thức tổ chức
                  <Tooltip label="Backend hiện chưa hỗ trợ trường này" position="top">
                      <IconExclamationCircle size={16} color="orange" />
                  </Tooltip>
              </Group>
          }
          // ...other props
      />
      ```
      
      **Purpose and Structure of the API_STATUS Report**:
      - **Integration Status Report**: It acts as a report of the integration status when connecting to APIs, tracking progress and ensuring alignment between Frontend and Backend.
      - **Gap Analysis (Crucial)**: It **MUST** highlight the Gap Analysis, listing **what fields or features are missing in the backend API compared to the pre-existing/designed UI** (các field thiếu so với UI đã có sẵn), and how those gaps are resolved (e.g., frontend extends local interfaces/states, backend updates).
      - **Mock Data Constraint**: Clearly note any fields still relying on mock data, which defaults the overall feature status to **PENDING**.
      - **Standard Formatting and Sections**: You **MUST** strictly follow this exact markdown structure (in Vietnamese):
        ```markdown
        # 📊 BÁO CÁO TÍCH HỢP & TIẾN ĐỘ API: [TÊN TÍNH NĂNG] - [TRẠNG THÁI (DONE/PENDING)]

        Tài liệu này lưu trữ trực tiếp trong thư mục feature để quản lý, theo dõi tiến độ tích hợp API và các điểm khác biệt (Gap) giữa Giao diện (UI) và Backend API.

        ---

        ## 📌 1. BẢNG TIẾN ĐỘ TỔNG QUAN (INTEGRATION DASHBOARD)

        | Hạng mục | Tiến độ | Trạng thái | Ghi chú |
        | :--- | :---: | :---: | :--- |
        | **Giao diện mẫu (UI Prototype)** | **100%** | 🟢 HOÀN THÀNH | Mô tả... |
        | **Tích hợp Service & Axios** | **100%** | 🟢 HOÀN THÀNH | Mô tả... |
        | **Tích hợp React Query & UI Binding** | **100%** | 🟢 HOÀN THÀNH / 🟡 ĐANG XỬ LÝ | Mô tả... |
        | **Khả năng tương thích Backend** | **100%** | 🟢 HOÀN THÀNH / 🟡 ĐANG XỬ LÝ | Mô tả... |

        ### 📊 Tiến độ tích hợp tổng thể (Overall Completion): [X]%
        ```text
        [████████████████████████████████████████] 100%
        ```

        ---

        ## 🔌 2. MA TRẬN TÍCH HỢP ENDPOINT (API ENDPOINTS MATRIX)

        - [x] **GET** `/api/...` - Lấy danh sách...
          - **Trạng thái:** 🟢 Hoạt động tốt.
        - [x] **POST** `/api/...` - Tạo mới...
          - **Trạng thái:** 🟢 Hoạt động tốt.

        ---

        ## 🔎 3. DANH SÁCH CÁC KHOẢNG TRỐNG (GAP ANALYSIS CHECKLIST)

        ### 🗹 A. Phía Frontend (Các giải pháp đồng bộ)
        - [x] **Tên issue/khoảng trống:**
          - *Mô tả:* Chi tiết vấn đề
          - *Đã xử lý:* Cách frontend xử lý

        ### 🗹 B. Phía Backend (Đã đồng bộ)
        - [x] **Tên issue/khoảng trống:**
          - *Nghiệp vụ:* Yêu cầu backend
          - *Đã xử lý:* Trạng thái tích hợp
        ```
5. **Single Unified Interface (NO Duplicate UI Props)**: **NEVER** create two different variations of props or interfaces (e.g. do not define separate `*UI` interfaces duplicating/extending backend properties for local fallback). Always synchronize and use **exactly one unified, shared interface** when integrating the API. Both layout components, create/update forms, and services must strictly bind to the exact same backend model definitions to maintain high consistency.
6. **Remove `isPrototype` Flag (MANDATORY)**: Once an API is integrated into a page or feature, you **MUST** find the sidebar menu navigation configuration (e.g. inside `layout.tsx` files or sidebar routing configs) and remove the `isPrototype: true` flag from the respective menu item so that the "P" (Prototype) badge is no longer shown on the sidebar UI.
7. **Relocate Interfaces to Shared Folder (MANDATORY)**: Khi đã tích hợp API rồi thì di chuyển interface ra thư mục `shared/interfaces` của ứng dụng để dùng chung và có thể sử dụng ở các menu khác. Đồng thời, cập nhật tất cả các đường dẫn import cũ sang `@/shared/interfaces/<InterfaceName>` và xóa file interface cục bộ ban đầu.
8. **Strict Payload Typing (No `any`)**: When constructing payloads to send to the backend (e.g., inside a `buildPayload` function before an API call), **DO NOT use `any`**. Instead, explicitly define an `[EntityName]Payload` interface in the `shared/interfaces/[EntityName].ts` file (e.g., `CommunityActivityPayload`) and use it to strictly type your payload object. This ensures TypeScript enforces API schema compliance during form submission and prevents missing fields or mismatched data types.
9. **Explicit Mock Data Commenting**: Any mock arrays, static lists, dummy data constants, or unimplemented feature placeholders MUST be explicitly commented with `// MOCK:` (e.g., `// MOCK: Temporary mock data for academic terms, waiting for API integration`). This ensures they can be easily tracked and replaced during final API integration.
10. **Server-Side Search, Filtering & Pagination (MANDATORY)**: Always implement search bars, data filters, and **pagination (PageNumber, PageSize)** by passing them as query parameters to the backend API (`useCustomReactQuery` payload). **Luôn luôn sử dụng server-side paging và server-side search cho các bảng dữ liệu.** Do NOT filter, search, or paginate data locally on the frontend UI unless explicitly requested. If the backend does not yet support search or pagination, include `PageNumber`, `PageSize`, and the search query in the API payload interface anyway, explicitly report it in the `API_STATUS_PENDING.md` file, **AND ALWAYS NOTIFY THE USER IN THE CHAT IMMEDIATELY** so the backend team can update it. Use `useDebouncedValue` for search inputs to prevent spamming API requests.
11. **No Mock Data for Integrated APIs (MANDATORY)**: Once an API is integrated and functioning, you **MUST NOT** use fallback mock data for missing fields (e.g., `const val = row.original.type || MOCK_ENUM_VALUE`). If a field is missing from the backend response, show an empty state (like `--`) or a warning icon with a tooltip (e.g., `IconAlertTriangle` with "Đang chờ API") to indicate that the data is pending from the backend.
12. **No Inline Object Types (MANDATORY)**: Do not define nested inline object types inside a parent interface (e.g., `communityType: { id: string; name: string; }`). Always extract these nested objects into their own exported interface models (e.g., `export interface CommunityType {...}`) and import/reuse them across the project. This ensures proper reusability, consistency, and prevents duplication of type definitions.
13. **Date Payload Formatting (MANDATORY)**: Khi xử lý payload của các form có chứa ngày bắt đầu và ngày kết thúc (ví dụ: `startDate`, `endDate`, `openRegisterDate`, `closeRegisterDate`), bạn BẮT BUỘC phải format thêm giờ phút giây cụ thể để gửi xuống backend. Ngày bắt đầu phải cộng thêm `00:00:00` và ngày kết thúc phải cộng thêm `23:59:59`. Ví dụ: `dayjs(body.startDate).format("YYYY-MM-DDT00:00:00")` và `dayjs(body.endDate).format("YYYY-MM-DDT23:59:59")`.
14. **No Backend Suffixes on Frontend Models (MANDATORY)**: Do not append `Dto`, `Request`, or `Response` to your frontend interface names (e.g. `SuspensionSummaryDto`, `CreateViolationRequest`). Use clean entity names (e.g. `SuspensionSummary`) and explicitly define payload interfaces with the `Payload` suffix (e.g. `ViolationReportPayload`) to maintain frontend domain clarity.

> [!WARNING]
> **Strict Interface Uniformity**: Do not allow duplicate fields or properties with different names representing the same semantic value (e.g. using `needProof` and `requireEvidence` at the same time). When the backend updates their model schema, immediately rename and align the frontend properties to use the exact backend field name globally across the layout, form state, and endpoints.
> 
> **Prototype Badge Removal**: Do not forget to remove `isPrototype: true` when a page is no longer a mock/static prototype. Having mock pages marked as completed or completed pages marked with the "P" badge confuses users and QA teams.
>
### Example Implementation:

#### 1. Form Initialization using the Backend Interface
```typescript
import { CommunityCriteria, CommunityCriteriaPayload } from "@/shared/interfaces/CommunityCriteria";
import { ScoreConstraintEnum } from "@/shared/consts/enum/ScoreConstraintEnum";

const DEFAULT_VALUES: CommunityCriteria = {
    code: "",
    name: "",
    isExternal: false,
    scoreConstraint: ScoreConstraintEnum.None,
    subCriterias: [{ code: "", name: "", maxScore: 0 }]
};

const form = useForm<CommunityCriteria>({
    initialValues: DEFAULT_VALUES,
    validate: {
        code: (val) => (val ? null : "Mã điều bắt buộc nhập"),
        name: (val) => (val ? null : "Nội dung tiêu chí bắt buộc nhập"),
        subCriterias: {
            name: (val) => (val ? null : "Tên bậc bắt buộc nhập"),
        }
    }
});
```

#### 2. Direct Binding in JSX
```tsx
{/* Text Inputs */}
<CustomTextInput
    label="Mã Điều"
    withAsterisk
    {...form.getInputProps("code")}
/>

<CustomCheckbox
    label="Ngoài trường"
    {...form.getInputProps("isExternal", { type: "checkbox" })}
/>

{/* Nested Arrays */}
{(form.values.subCriterias || []).map((item, index) => (
    <Table.Tr key={index}>
        <Table.Td>
            <CustomTextInput
                placeholder="Tên bậc điểm..."
                {...form.getInputProps(`subCriterias.${index}.name`)}
            />
        </Table.Td>
        <Table.Td>
            <CustomNumberInput
                {...form.getInputProps(`subCriterias.${index}.maxScore`)}
            />
        </Table.Td>
    </Table.Tr>
))}
```

#### 3. Handling API Submissions
```typescript
const handleSubmit = async (values: CommunityCriteria) => {
    // Sync auxiliary fields or assign codes dynamically right before transmitting
    const subCriterias = (values.subCriterias || [])
        .filter((step) => step.name?.trim() !== "")
        .map((step) => ({
            ...step,
            code: step.name, // Auto-assign sub-criteria code
        }));

    const maxScore = Math.max(...subCriterias.map((s) => s.maxScore || 0), 0);

    const payload: CommunityCriteriaPayload = {
        code: values.code,
        name: values.name,
        isExternal: values.isExternal,
        maxScore,
        scoreConstraint: values.scoreConstraint,
        subCriterias,
    };

    return communityCriteriaService.create({ tenantId, body: payload });
};
```

## Best Practices
1. **Always handle loading & empty states**: Ensure your React component handles loading states smoothly with Mantine's `<Loader />` or custom skeleton loaders.
2. **Prevent redundant renders**: Use `refetchOnWindowFocus: false` inside `useCustomReactQuery` options unless real-time synchronization is requested by the user.
3. **Keep query keys descriptive**: Use unique query key arrays matching the resource and its query dependencies (e.g., `["rl-versions", tenantId, filterStatus]`).
4. **Use `aq-core-framework` for API UI Components**: When using UI components that trigger API calls directly (such as Delete buttons in tables, API Selects, Import/Export buttons), always import them from `@aq-fe/aq-core-framework` (e.g., `CustomButtonDelete`, `CustomButtonDeleteList`, `CustomApiSelect`). **DO NOT** use the legacy versions from `@aq-fe/aq-legacy-framework` under any circumstances. Likewise, do not use the raw standalone buttons from `@aq-fe/core-ui` for these actions, as they lack the newer integrated API logic.
5. **Pass ID instead of Objects to Modals**: When opening a modal component (like a Create/Update form) to view or edit an item from a table, pass the item's `id` as a prop (e.g., `planId`, `userId`) instead of passing the whole object (e.g., `plan`). Inside the modal, use `useCustomReactQuery` to fetch the detailed data using that `id`. This ensures the modal always has the latest and most complete data, including any nested properties that might not be present in the list view.
6. **Do Not Manually Map or Render STT Columns**: When replacing mock prototypes with real API integrations in `CustomTanstackTable`, always remember to completely remove any manually handled `STT` or `order` columns from your column definitions and data models. `CustomTanstackTable` automatically injects a sequence number column by default, so continuing to map it manually from API responses will result in visually duplicate STT columns on the UI.
7. **Self-Fetching Dropdowns & Shared Components**: When building shared dropdowns (e.g., `CustomSemesterSelect`, `CustomUnitSelect`) that rely on backend data, **DO NOT** fetch data in the parent and pass it down as props. Instead, encapsulate the data fetching logic directly inside the shared component. Retrieve necessary context variables (like `tenantId` or `academicYearId`) internally using global stores (e.g., `useAuthenticateStore`, `useAcademicYearStore`). This encapsulates business logic, reduces prop drilling, and prevents redundant API calls. **Furthermore, these self-fetching business components MUST be placed in `src/shared/features/<DomainName>/` (e.g., `src/shared/features/Semester/CustomSemesterSelect`) instead of the generic `shared/components/` directory to cleanly separate domain logic from pure UI components.**
8. **Avoid `useCustomReactQuery` for Shared Dropdowns**: When fetching simple, non-paginated lists (like Semesters, Faculties, Roles) inside shared UI components, **strictly use standard `@tanstack/react-query`'s `useQuery`** instead of `useCustomReactQuery`. The `useCustomReactQuery` hook manages pagination state internally and calls `setState`; using it in shared background components or modals can trigger severe cross-component render conflicts ("Cannot update a component while rendering a different component") and crash the application when multiple components mount simultaneously with the same queryKey.
9. **API-Side Filtering vs Frontend Filtering**: Always implement state filtering (e.g., filtering `operationState`, `state`, etc.) via the backend API query parameters (like sending `operationState: 0` in the payload) rather than fetching all data and filtering it on the frontend arrays. This ensures server-side pagination works correctly and reduces frontend payload size.
