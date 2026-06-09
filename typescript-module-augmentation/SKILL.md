---
name: typescript-module-augmentation
description: Hướng dẫn kỹ thuật Module Augmentation để đồng bộ kiểu dữ liệu (Type-safe) từ ứng dụng (App) xuống thư viện dùng chung (Framework) mà không cần viết file wrapper.
---

# Kỹ thuật TypeScript Module Augmentation (Ép kiểu động cho Framework)

Trong kiến trúc Monorepo, các thư viện (packages) dùng chung như `@aq-fe/aq-standard-framework` được viết dưới dạng Generic để có thể chạy trên mọi dự án. Chúng ta không được phép import các cấu hình cụ thể của App (như `features.ts`) ngược vào Framework.
Tuy nhiên, để developer ở các App có được trải nghiệm code tốt nhất (VS Code tự động nhắc lệnh IntelliSense đối với các Component/Hook từ Framework), chúng ta **TUYỆT ĐỐI KHÔNG** tạo các thư mục "wrapper" rườm rà. Thay vào đó, hãy sử dụng **Module Augmentation**.

### Cấu trúc thực hiện chuẩn:

#### Bước 1: Tại thư viện dùng chung (Framework)
Khai báo một `interface` rỗng để làm "đầu cắm", và một kiểu `Type` động tự fallback về string nếu không ai cắm vào:
```typescript
// Ví dụ tại: packages/aq-standard-framework/src/shared/features/FeatureFlagContext.tsx

// 1. Giao diện rỗng chờ các ứng dụng cắm type vào
// eslint-disable-next-line @typescript-eslint/no-empty-object-type
export interface FeatureFlagRegistry {}

// 2. Tự động lấy danh sách key nếu có người cắm vào, nếu không thì trả về kiểu string mặc định
export type FeatureName = keyof FeatureFlagRegistry extends never ? string : keyof FeatureFlagRegistry;

// 3. Sử dụng Type động cho các hàm/component
export function useFeatureFlag(featureName: FeatureName): boolean { ... }
export interface FeatureGuardProps { feature: FeatureName; }
```

#### Bước 2: Tại Ứng Dụng (App)
Trong file định nghĩa type của ứng dụng, sử dụng `declare module` để "bơm" (inject) trực tiếp Type của App vào cái "đầu cắm" rỗng của Framework:
```typescript
// Ví dụ tại: apps/spm/src/shared/types/features.ts
export interface AppFeatures {
    enableAdvancedExport?: boolean;
    enableFaceDetect?: boolean;
}

// Bơm kiểu AppFeatures vào thư viện gốc
declare module '@aq-fe/aq-standard-framework/shared/features/FeatureFlagContext' {
    export interface FeatureFlagRegistry extends AppFeatures {}
}
```

### Tiêu chuẩn đầu ra:
- Developer ở App import thẳng Component từ Framework (`import { FeatureGuard } from '@aq-fe/aq-standard-framework...'`).
- VS Code ngay lập tức gợi ý đúng danh sách các biến được định nghĩa trong `AppFeatures` mà không cần code dư thừa.
