---
name: nextjs-build-dev-server
description: Quy tắc xử lý xung đột giữa lệnh Next.js Build và Next.js Dev Server đang chạy.
---

# Lỗi xung đột Next.js Build và Dev Server

## Vấn đề
Khi dự án Next.js đang được chạy ở chế độ phát triển (Dev Server qua lệnh `pnpm dev` hoặc `next dev`), nếu bạn chạy đồng thời lệnh build (`pnpm build` hoặc `next build`), quá trình build sẽ ghi đè hoặc xóa các tệp tin trong thư mục `.next` (hoặc thư mục build tương ứng) mà Dev Server đang sử dụng. 

Điều này dẫn đến việc Dev Server bị mất file và gây ra hàng loạt lỗi trên trình duyệt (như lỗi HTTP 500 Internal Server Error liên tục, hoặc thông báo `missing required error components, refreshing...` trên giao diện, và các lỗi `ENOENT` ở console).

## Cách xử lý
- **Trước khi Build (BẮT BUỘC)**: Bất cứ khi nào bạn định chạy lệnh build dự án (ví dụ `pnpm build` hoặc `next build`) để kiểm tra, bạn **PHẢI tắt (kill) các task chạy Dev Server đang chạy ngầm** bằng công cụ `manage_task`.
- **Sau khi Build xong (BẮT BUỘC)**: Sau khi tiến trình build kết thúc thành công, bạn **PHẢI nhớ chạy lại lệnh Dev Server** (ví dụ `pnpm exec turbo dev` hoặc `pnpm --filter ... dev` với tuỳ chọn `IsDaemon=true`) để hệ thống phục vụ lại trang web cho môi trường dev.

**Quy trình chuẩn:**
1. Chạy `manage_task` với action `list` để tìm các task Dev Server đang chạy.
2. Chạy `manage_task` với action `kill` để tắt hoàn toàn các task đó.
3. Chạy lệnh build (VD: `pnpm build`).
4. (Chờ build xong).
5. Chạy lệnh `run_command` để start lại Dev Server.

## Dấu hiệu nhận biết
- Trình duyệt hiện dòng chữ: `missing required error components, refreshing...`
- Tab Network báo lỗi liên tục mã 500 cho các request lấy tệp tin chunk `.js`.
- Console báo lỗi: `Error: ENOENT: no such file or directory, open '...\app-build-manifest.json'`
