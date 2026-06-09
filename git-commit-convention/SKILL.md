---
name: git-commit-convention
description: Quy định bắt buộc khi viết commit message cho dự án
---

# Quy Định Viết Git Commit Message

Khi thực hiện tạo commit cho dự án, bạn **bắt buộc** phải tuân thủ các quy tắc sau:

1. **Ngôn ngữ**: Commit message phải được viết bằng **Tiếng Việt**.
2. **Menu bị ảnh hưởng**: Bắt buộc phải lưu/ghi chú rõ **tên menu bị ảnh hưởng** bởi thay đổi trong nội dung commit (ví dụ: `=> Menu bị ảnh hưởng: Quản lý danh sách hoạt động cộng đồng`).
3. **Cấu trúc**:
   - Tiêu đề commit (dòng 1): Theo chuẩn `<type>(<scope>): <short summary>`.
     - `<type>` bắt buộc phải là một trong các loại chuẩn: `feat`, `fix`, `chore`, `ci`, `build`, `docs`, `refactor`, `test`, `perf`, `style`. Tuyệt đối KHÔNG DÙNG từ không chuẩn như `update` hoặc "Cập nhật".
     - `<scope>` (trong ngoặc đơn) **BẮT BUỘC** là tên dự án/package bị ảnh hưởng (ví dụ: `sae-new`, `aq-core-framework`, `core-ui`). Nếu thay đổi từ 2 package trở lên, phân tách bằng dấu phẩy (ví dụ: `refactor(sae-new,core-ui): ...`). Tuyệt đối không dùng tên folder feature làm scope.
     - `<short summary>` **BẮT BUỘC** phải viết bằng **Tiếng Việt** (ví dụ: `sắp xếp lại menu và xoá file jwtUtils`).
   - Body commit: Giải thích chi tiết hơn về thay đổi (nếu cần thiết) và dòng ghi chú menu bị ảnh hưởng.
4. **Quy trình thực hiện**: Tuyệt đối **KHÔNG** tự ý thực thi các lệnh `git commit`. Bạn chỉ được phép sinh ra nội dung commit message và in ra trong phần chat.
5. **Định dạng hiển thị**: TUYỆT ĐỐI KHÔNG sinh ra tin nhắn commit dưới dạng lệnh Terminal (ví dụ `git commit -m "..."`). Hãy in TOÀN BỘ nội dung commit (bao gồm Tiêu đề, một dòng trống, và Body) gộp chung vào **MỘT khối code block duy nhất** dạng text thô. Không tách rời tiêu đề và body ra văn bản riêng, để người dùng có thể bấm Copy một lần và dán (paste) thẳng vào giao diện Git GUI (như Fork, SourceTree) cho nhanh.
