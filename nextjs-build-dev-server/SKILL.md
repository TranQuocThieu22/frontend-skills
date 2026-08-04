---
name: nextjs-build-dev-server
description: Quy tắc xử lý xung đột giữa lệnh Next.js Build và Next.js Dev Server đang chạy.
---

# Lỗi xung đột Next.js Build và Dev Server

## Vấn đề
Khi dự án Next.js đang được chạy ở chế độ phát triển (Dev Server qua lệnh `pnpm dev` hoặc `next dev`), nếu bạn chạy đồng thời lệnh build (`pnpm build` hoặc `next build`), quá trình build sẽ ghi đè hoặc xóa các tệp tin trong thư mục `.next` (hoặc thư mục build tương ứng) mà Dev Server đang sử dụng. 

Điều này dẫn đến việc Dev Server bị mất file và gây ra hàng loạt lỗi trên trình duyệt (như lỗi HTTP 500 Internal Server Error liên tục, hoặc thông báo `missing required error components, refreshing...` trên giao diện, và các lỗi `ENOENT` ở console).

## Cách xử lý
- **Trước khi Build**: Nếu có thể, hãy tắt (kill) task chạy Dev Server trước khi thực hiện `pnpm build`.
- **Sau khi Build xong**: Nếu trước đó bạn vừa build dự án mà Dev Server vẫn đang chạy ngầm, bạn **BẮT BUỘC** phải:
  1. Kill task Dev Server cũ.
  2. Chạy lại lệnh Dev Server (VD: `pnpm --filter rrs dev`) để khởi tạo lại thư mục build cho môi trường dev.

## Dấu hiệu nhận biết
- Trình duyệt hiện dòng chữ: `missing required error components, refreshing...`
- Tab Network báo lỗi liên tục mã 500 cho các request lấy tệp tin chunk `.js`.
- Console báo lỗi: `Error: ENOENT: no such file or directory, open '...\app-build-manifest.json'`
