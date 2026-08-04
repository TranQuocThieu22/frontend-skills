---
name: rrs-backend-integration
description: Lưu ý và quy trình kiểm tra lỗi khi làm dự án RRS liên quan đến tích hợp API và vấn đề Backend
---

# Hướng dẫn kiểm tra vấn đề Backend khi tích hợp API dự án RRS

Khi thực hiện tích hợp API trong ứng dụng `rrs` (Room Reservation System) và gặp hiện tượng API trả về thành công nhưng dữ liệu không thay đổi, hãy thực hiện các bước kiểm tra sau để xác định lỗi Backend:

1. **Kiểm tra Payload gửi đi:** 
   - Sử dụng tab Network trên trình duyệt (hoặc log) để đảm bảo cấu trúc Payload (tên biến dạng `snake_case` hoặc `camelCase`) và giá trị gửi lên API là chính xác tuyệt đối so với tài liệu mô tả hoặc cấu trúc DB.

2. **Dấu hiệu nhận biết lỗi Backend:**
   - Nếu Frontend gửi đúng Payload, API trả về Code 200 (Thành công), nhưng gọi API `GET` ngay sau đó lại vẫn trả về dữ liệu cũ => Lỗi 100% thuộc về Backend.

3. **Các lỗi Backend thường gặp trong dự án (Đặc tả lỗi):**
   - **Quên Commit Transaction:** Backend sử dụng `db.Database.BeginTransaction()` để cập nhật dữ liệu (`db.BulkEntityUpdateOne`) nhưng lập trình viên quên gọi `trans.Commit()`. Hậu quả là dữ liệu bị rollback tự động khi kết thúc block `using`.
   - **Lỗi Cache:** Dữ liệu GET bị lưu Cache (Redis/Memory) nhưng không được Invalidate/Clear sau khi thực hiện thao tác PUT/POST/DELETE.
   - **Gắn sai Map Field:** Cấu trúc biến của Frontend không được ánh xạ đúng vào Model ở Controller (đôi khi do khác biệt `snake_case` và `PascalCase` trong C#).

## Cấu trúc mã nguồn Backend RRS
- Toàn bộ API nghiệp vụ của hệ thống RRS (Room Reservation System / Đăng ký phòng chức năng) được đặt tại: `Source.NET/trunk2/Edusoft.NET/Edusoft.Api/Controllers/Customer/`
- Controller chính xử lý đăng ký phòng, quản lý phòng, bộ lọc tìm kiếm: `PhongChucNangController.cs`

**Quy tắc:**
- Không tự ý sửa mã nguồn Backend của dự án C#. Chỉ cần phân tích, khoanh vùng chính xác vị trí lỗi (ví dụ file Controller tương ứng) và đưa ra đặc tả (specification) để báo cho team Backend tự khắc phục.
