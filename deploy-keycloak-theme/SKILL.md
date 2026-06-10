---
name: deploy-keycloak-theme
description: Hướng dẫn cách build và deploy Keycloak theme (React) vào server Keycloak nội bộ.
---

# Deploy Keycloak Theme

Khi làm việc với project `keycloak-theme` (`apps/keycloak-theme`), nếu bạn cần build và cập nhật (deploy) giao diện vào máy chủ Keycloak đang chạy cục bộ (local) của dự án, bạn **BẮT BUỘC** phải làm theo quy trình sau:

## 1. Không sử dụng lệnh build độc lập
Thay vì tự chạy `pnpm run build-keycloak-theme` và tự đóng gói file `.jar`, bạn hãy sử dụng lệnh deploy đã được cấu hình sẵn trong dự án.

## 2. Cách thức Deploy
Sử dụng script sau tại thư mục `apps/keycloak-theme`:

```bash
pnpm run deploy
```

Lệnh này sẽ tự động gọi file `deploy-theme.bat` để:
1. Build toàn bộ ứng dụng React và biên dịch sang giao diện FreeMarker của Keycloak.
2. Tự động `xcopy` toàn bộ thư mục theme đã build vào thư mục `themes/keycloakify-starter` của máy chủ gốc (`nestjs-framework/themes/...`).
3. Nhờ cơ chế Hot-Reload của thư mục `themes/`, bạn chỉ cần yêu cầu người dùng **F5 (Hard Refresh)** lại trình duyệt là sẽ thấy ngay cập nhật mà không cần phải khởi động lại server Keycloak.

## 3. Lưu ý khi Deploy
- Quá trình deploy có thể tốn vài giây đến một phút do cần chạy trình đóng gói Vite và sao chép file.
- Luôn sử dụng lệnh này thay cho các phương thức đóng gói `.jar` thủ công trước đây.
