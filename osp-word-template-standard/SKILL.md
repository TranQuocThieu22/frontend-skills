---
name: osp-word-template-standard
description: Hướng dẫn cấu trúc Tag và chuẩn hóa file Word Template để sinh UI tự động trong hệ thống OSP (Đơn từ).
---

# Chuẩn Hóa Cấu Trúc File Word Template (OSP)

Hệ thống OSP sử dụng tính năng **Rich Text Content Control** (hoặc Plain Text Content Control) của Microsoft Word để nhúng các biến dữ liệu. Dựa vào các biến này, hệ thống sẽ tự động sinh ra Form nhập liệu (Dynamic UI) tương ứng trên giao diện Web.

Tài liệu này quy định cú pháp chuẩn để thiết lập **Title (Nhãn)** và **Tag (Dữ liệu & Cấu hình)**.

## 1. Nguyên lý Thiết kế (Separation of Concerns)

- **Title (Tiêu đề)**: Chuyên dùng để hiển thị nhãn (Label) và Placeholder trên giao diện Web. Ví dụ: `Họ và tên sinh viên`.
- **Tag (Thẻ)**: Chuyên dùng để định nghĩa kiểu dữ liệu (Schema), cơ chế kiểm tra (Validation), và biến dữ liệu gửi về API. Bạn **bắt buộc** phải tuân thủ cú pháp Tag.

## 2. Cú pháp Tag Tổng Quát

Khi cấu hình Content Control (Properties) trong Word, ô **Tag** phải được nhập theo cú pháp sau:
```text
[group_name.][field_name]:[type]([options])?[modifiers]
```

**Trong đó:**
- `field_name` (Bắt buộc): Tên biến JSON sẽ gửi về Backend (Ví dụ: `ho_ten`, `ly_do`). Khuyến nghị dùng chữ thường và gạch dưới (snake_case).
- `type` (Tùy chọn): Kiểu dữ liệu để sinh UI (`text`, `number`, `date`, `phone`, `email`, `bool`, `select`, `api`). Mặc định là `text`.
- `options` (Tùy chọn): Tham số cấu hình cho `type`. Ví dụ cấu hình danh sách dropdown: `(1:Nam,2:Nữ)`.
- `modifiers` (Tùy chọn): Cờ bổ trợ điều khiển hành vi:
  - `*`: Bắt buộc nhập (Báo đỏ nếu để trống).
  - `!`: Chỉ đọc (Tự động điền, không cho người dùng tự sửa).
  - `~`: Khung nhập nhiều dòng (Chỉ dùng với type `text` để sinh ra Textarea).
- `group_name.` (Tùy chọn - Dành cho Tương lai): Dùng để nhóm dữ liệu thành mảng (ví dụ: bảng danh sách).

---

## 3. Bảng Tham Chiếu Kiểu Dữ Liệu & Modifiers

### 3.1. Các kiểu dữ liệu (Types)

| Type | Cú pháp trong Tag | Ý nghĩa & Giao diện sinh ra (UI) | Ví dụ |
| :--- | :--- | :--- | :--- |
| **Văn bản** | `field_name:text` (hoặc `field_name`) | Khung nhập chuỗi (TextInput). | `noi_sinh:text` |
| **Số** | `field_name:number` | Khung nhập chỉ cho phép số. | `so_tin_chi:number` |
| **Ngày tháng** | `field_name:date` | Hiển thị bảng chọn lịch (DatePicker) chuẩn DD/MM/YYYY. | `ngay_sinh:date` |
| **Đúng/Sai** | `field_name:bool` | Ô tích chọn (Checkbox). | `xac_nhan:bool` |
| **SĐT** | `field_name:phone` | Khung nhập văn bản, sẽ mở rộng validate SĐT. | `so_dien_thoai:phone` |
| **Email** | `field_name:email` | Khung nhập văn bản, sẽ mở rộng validate Email. | `email_lien_he:email` |
| **Dropdown tĩnh** | `field_name:select(key1:val1,key2:val2)` | Khung chọn 1 giá trị từ danh sách (Select). | `gioi_tinh:select(1:Nam,2:Nữ)` |
| **Dropdown API** | `field_name:api(endpoint)` | *(Tính năng tương lai)* Lấy dữ liệu từ Backend. | `ma_nganh:api(dm_nganh)` |

### 3.2. Các cờ bổ trợ (Modifiers)

Các cờ này gắn trực tiếp vào cuối tên biến. Có thể kết hợp nhiều cờ (VD: `~*` là Textarea bắt buộc nhập).

| Cờ | Ý nghĩa | Hành vi trên UI | Cú pháp ví dụ |
| :---: | :--- | :--- | :--- |
| `*` | **Bắt buộc (Required)** | Render dấu hoa thị đỏ, chặn xuất file nếu để trống. | `ho_ten*` |
| `!` | **Chỉ đọc (Readonly)** | Render ô màu xám, khóa chỉnh sửa. (Dùng cho thông tin sinh viên). | `ma_sv!` |
| `~` | **Nhiều dòng (Textarea)** | Thay vì khung nhập nhỏ, sẽ render khung text lớn tự động co giãn. | `ly_do:text~*` |

---

## 4. Tự Động Điền Thông Tin Sinh Viên (Preset Syncing)

Hệ thống sẽ **tự động** lấy thông tin cá nhân của người dùng đang đăng nhập và điền vào Form nếu bạn đặt `Tag` khớp chính xác với một trong các từ khóa dưới đây. Khuyến nghị kết hợp với cờ `!` (Readonly) để chặn người dùng tự ý sửa.

| Thông tin cần lấy | Đặt Title (Nhãn hiển thị) | Đặt Tag (Dữ liệu & Cấu hình) | Kết quả giao diện sinh ra |
| :--- | :--- | :--- | :--- |
| **Họ và tên sinh viên** | `Họ và tên` | `ho_ten!` | `<TextInput>` màu xám (chỉ đọc), đã điền sẵn "Nguyễn Văn A" |
| **Mã Sinh Viên** | `Mã SV` | `ma_sv!` | `<TextInput>` màu xám (chỉ đọc), đã điền sẵn "2121050441" |
| **Ngày sinh** | `Ngày sinh` | `ngay_sinh:date!` | Bảng chọn lịch `<DatePickerInput>` màu xám, điền sẵn ngày |
| **Lớp quản lý** | `Lớp` | `lop!` | `<TextInput>` màu xám (chỉ đọc), điền sẵn "D21CQCN01-N" |
| **Khóa học/Niên khóa** | `Khóa học` | `nien_khoa!` | `<TextInput>` màu xám (chỉ đọc), điền sẵn "2021-2026" |
| **Khoa (Đơn vị)** | `Khoa quản lý` | `khoa!` | `<TextInput>` màu xám (chỉ đọc), điền sẵn "Công nghệ thông tin" |
| **Ngành học** | `Ngành` | `nganh!` | `<TextInput>` màu xám (chỉ đọc), điền sẵn "Kỹ thuật phần mềm" |

> **Lưu ý:** Chỉ cần nhập đúng ô Tag theo cột số 3 (ví dụ: `ho_ten!`), ô Title bạn có thể tự do đặt tùy ý (Họ tên, Tên sinh viên...). Dấu `!` là không bắt buộc, nhưng cực kỳ khuyến nghị để tránh sinh viên tự sửa sai lệch thông tin gốc.

---

## 5. Ví Dụ Cấu Hình Chuẩn (Thực Tế)

Dưới đây là một bảng mẫu về cách bạn nên cấu hình file Word để đạt hiệu quả tốt nhất:

| Yêu cầu thực tế | Cấu hình Title (Word) | Cấu hình Tag (Word) | UI kết quả |
| :--- | :--- | :--- | :--- |
| Sinh viên tự nhập **Lý do xin phép**, bắt buộc nhập và cần nhiều dòng. | `Trình bày lý do` | `ly_do:text~*` | Textarea, có sao đỏ `*`. |
| Sinh viên tự chọn **Giới tính** (Nam/Nữ). | `Giới tính sinh viên` | `gioi_tinh:select(1:Nam,2:Nữ)` | Dropdown List với 2 option. |
| Hiển thị **Mã SV**, lấy từ hệ thống, KHÔNG cho sinh viên sửa. | `Mã số sinh viên` | `mssv!` | TextInput khóa xám, tự động điền. |
| Tự nhập **Ngày vắng thi**, bắt buộc. | `Ngày xin vắng` | `ngay_vang:date*` | DatePicker (Lịch), có sao đỏ `*`. |
| Cam kết **Đồng ý điều khoản**, bắt buộc. | `Xác nhận cam kết` | `cam_ket:bool*` | Checkbox tích chọn. |

---

## 6. Hướng Dẫn Thao Tác Trực Tiếp Trên MS Word

1. **Bật Developer Mode:** Vào `File > Options > Customize Ribbon`. Ở cột phải, tick chọn hộp **Developer**.
2. **Chèn Biến:** Đặt con trỏ chuột vào vị trí muốn chèn dữ liệu. Mở tab **Developer**, bấm vào biểu tượng `Aa` (Rich Text Content Control) hoặc `Aa` chữ nhỏ (Plain Text).
3. **Cấu hình Biến:** Bấm nút **Properties** trên thanh công cụ.
   - Ô **Title**: Điền tiêu đề hiển thị (VD: `Lý do xin rút môn`).
   - Ô **Tag**: Điền chuẩn cú pháp (VD: `ly_do:text~*`).
4. **Lưu file** `.docx` và upload lên hệ thống.

---

## 7. Lưu ý về WPS Office

Hệ thống hỗ trợ 100% file `.docx` tạo từ **WPS Office**. WPS có cấu trúc lưu XML hơi khác so với MS Word (các Content Control rỗng không có thẻ `<w:t>`). Các developer khi bảo trì code `DocxViewer` hoặc `extractFieldsFromWord` cần lưu ý tuyệt đối không dùng giả định: *Mọi SDT đều có thẻ `<w:t>` bên trong*. Luôn phải có logic fallback để tự tạo `<w:t>` nếu field trống.
