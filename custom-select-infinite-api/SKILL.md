---
name: custom-select-infinite-api
description: Instructions and guidelines for using the CustomSelectInfiniteAPI and CustomMultiSelectInfiniteAPI components for scroll paging and server-side search.
---

# CustomSelectInfiniteAPI & CustomMultiSelectInfiniteAPI Component Guidelines

These two infinite-scroll dropdown select components in `@aq-fe/aq-core-framework` are designed to support infinite scroll paging and server-side search (search-on-type) under Mantine v7/v8.

## Location
- Component files: 
  - Single select: `packages/aq-core-framework/src/shared/components/withAPI/CustomSelectInfiniteAPI.tsx`
  - Multi select: `packages/aq-core-framework/src/shared/components/withAPI/CustomMultiSelectInfiniteAPI.tsx`
- Component import paths:
  - `import { CustomSelectInfiniteAPI } from "@aq-fe/aq-core-framework/shared/components/withAPI/CustomSelectInfiniteAPI";`
  - `import { CustomMultiSelectInfiniteAPI } from "@aq-fe/aq-core-framework/shared/components/withAPI/CustomMultiSelectInfiniteAPI";`

## 1. Single Select: CustomSelectInfiniteAPI

### Component Props
- `value`: `string | null` - The currently selected option's value.
- `onChange`: `(value: string | null, selectedItem: T | null) => void` - Callback fired when option is selected or cleared.
- `placeholder`: `string` (Optional) - Input placeholder. Default: `"-- Chọn giá trị --"`.
- `label`: `string` (Optional) - Form input label.
- `queryKeyPrefix`: `string` - Prefix for the react-query cache key.
- `fetchDataFn`: `(params: { PageNumber: number; PageSize: number; Code?: string }) => Promise<any>` - Service function returning paginated results.
- `getLabel`: `(item: T) => string` - Function returning the visual text label for an option item.
- `getValue`: `(item: T) => string` - Function returning the unique identifier value for an option item.
- `pageSize`: `number` (Optional) - Number of items to fetch per page. Default: `20`.

### Single Select Usage Example
```tsx
<CustomSelectInfiniteAPI
  label="Chọn tài khoản người dùng"
  placeholder="-- Chọn người dùng --"
  queryKeyPrefix="user-authorization-users-list"
  fetchDataFn={(params) => userService.getAll({ pagingParams: params })}
  value={selectedUser?.id || selectedUser?.code || null}
  onChange={(val, user) => onSelectUser(user)}
  getValue={(u) => u.id || u.code || u.username || ""}
  getLabel={(u) => `${u.code || u.username} - ${u.name || u.firstName + ' ' + u.lastName}`}
/>
```

---

## 2. Multi Select: CustomMultiSelectInfiniteAPI

### Component Props
- `value`: `string[]` - Array of currently selected values.
- `onChange`: `(value: string[], selectedItems: T[]) => void` - Callback fired when options are toggled or removed.
- `placeholder`: `string` (Optional) - Input placeholder. Default: `"-- Chọn giá trị --"`.
- `label`: `string` (Optional) - Form input label.
- `queryKeyPrefix`: `string` - Prefix for the react-query cache key.
- `fetchDataFn`: `(params: { PageNumber: number; PageSize: number; Code?: string }) => Promise<any>` - Service function returning paginated results.
- `getLabel`: `(item: T) => string` - Function returning the visual text label for an option item.
- `getValue`: `(item: T) => string` - Function returning the unique identifier value for an option item.
- `pageSize`: `number` (Optional) - Number of items to fetch per page. Default: `20`.

### Multi Select Usage Example
```tsx
import { CustomMultiSelectInfiniteAPI } from "@aq-fe/aq-core-framework/shared/components/withAPI/CustomMultiSelectInfiniteAPI";
import { userService } from "@aq-fe/aq-core-framework/shared/APIs/userService";
import { useState } from "react";

// Inside component:
const [selectedUserIds, setSelectedUserIds] = useState<string[]>([]);
const [selectedUsers, setSelectedUsers] = useState<any[]>([]);

<CustomMultiSelectInfiniteAPI
  label="Chọn nhiều tài khoản người dùng"
  placeholder="-- Chọn người dùng --"
  queryKeyPrefix="user-multi-auth-list"
  fetchDataFn={(params) => userService.getAll({ pagingParams: params })}
  value={selectedUserIds}
  onChange={(ids, users) => {
    setSelectedUserIds(ids);
    setSelectedUsers(users);
  }}
  getValue={(u) => u.id || u.code || u.username || ""}
  getLabel={(u) => `${u.code || u.username} - ${u.name || u.firstName + ' ' + u.lastName}`}
/>
```


## Design and UX Behavior
1. **Server-side Search**: Gõ từ khóa tìm kiếm (search input) sẽ kích hoạt gọi lại API với tham số `Code` (hoặc từ khóa tương ứng). Dữ liệu lọc ở phía Backend nên không có độ trễ client-side khi dữ liệu lớn.
2. **Cuộn phân trang (Scroll Paging)**: Dropdown tự động lắng nghe sự kiện `onScroll` của ScrollArea. Khi cuộn gần tới đáy (bottom), nó sẽ tăng `PageNumber` để load thêm 20 bản ghi tiếp theo và cộng dồn lại.
3. **Nút xóa nhanh**: Khi có giá trị được chọn, nút `Chevron` chuyển thành `CloseButton` cho phép xóa trắng giá trị hiện tại ngay lập tức.
4. **Trạng thái tải (Loader)**: Hiển thị icon Loading nhỏ khi đang trong quá trình Fetch dữ liệu trang tiếp theo hoặc tìm kiếm.

## Usage Example
```tsx
import { CustomSelectInfiniteAPI } from "@aq-fe/aq-core-framework/shared/components/withAPI/CustomSelectInfiniteAPI";
import { userService } from "@aq-fe/aq-core-framework/shared/APIs/userService";

// Inside component:
<CustomSelectInfiniteAPI
  label="Chọn tài khoản người dùng"
  placeholder="-- Chọn người dùng --"
  queryKeyPrefix="user-authorization-users-list"
  fetchDataFn={(params) => userService.getAll({ pagingParams: params })}
  value={selectedUser?.id || selectedUser?.code || null}
  onChange={(val, user) => onSelectUser(user)}
  getValue={(u) => u.id || u.code || u.username || ""}
  getLabel={(u) => `${u.code || u.username} - ${u.name || u.firstName + ' ' + u.lastName}`}
/>
```
