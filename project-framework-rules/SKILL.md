---
name: project-framework-rules
description: Quy định bắt buộc về việc sử dụng các package framework (aq-core-framework, aq-standard-framework, aq-legacy-framework) cho từng ứng dụng trong monorepo.
---

# Quy Định Sử Dụng Framework Packages

Dự án này là một Monorepo chứa nhiều ứng dụng (`apps/`) và nhiều thư viện dùng chung (`packages/`). Để đảm bảo tính nhất quán và tránh phình to kích thước ứng dụng (bundle size), khi code hoặc cài đặt module, bạn **bắt buộc phải tuân thủ nghiêm ngặt** quy tắc phân bổ framework sau đây:

### 1. Thành phần UI (@aq-fe/core-ui)
- **Phạm vi**: Tất cả các dự án đều xài.
- **Quy tắc**: Mọi dự án trong `apps/` đều được phép import và sử dụng `@aq-fe/core-ui`.

### 2. Các Framework xử lý Logic (@aq-fe/aq-*-framework)
Tuyệt đối KHÔNG import thừa các framework không thuộc dự án. Quy tắc ánh xạ như sau:

- **`sae-new`**: CHỈ xài `@aq-fe/aq-core-framework`.
- **`spm`**: CHỈ xài `@aq-fe/aq-standard-framework`.
- **Tất cả các dự án còn lại** (như `eaq`, `school`, `college`, v.v.): xài `@aq-fe/aq-legacy-framework`.

---

### Lưu ý quan trọng cho Developer:
Nhìn vào file `package.json` thực tế đôi khi bạn sẽ thấy một dự án bị cài "kẹp" cả 3 framework (như hình ảnh lúc thêm skill này, dự án `spm` bị cài cả `core`, `legacy`, và `standard`). 
=> Khi làm việc, bạn **chỉ được phép import code từ đúng framework được phân công ở trên**. Việc cài thừa có thể do clone file cấu hình, nhưng lúc import code tuyệt đối không được gọi nhầm thư viện của dự án khác.
