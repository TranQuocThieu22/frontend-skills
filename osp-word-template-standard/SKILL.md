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
  - `@`: Auto-fill (Gắn cờ báo hiệu đây là trường sẽ tự động lấy dữ liệu từ hệ thống/API, kèm icon minh hoạ trên Web).
- `group_name(group_title).` (Tùy chọn): Dùng để nhóm dữ liệu thành mảng lặp (Ví dụ: danh sách môn học). `group_name` là khóa kỹ thuật gửi API (chữ thường không dấu), `group_title` trong ngoặc đơn là nhãn hiển thị của nhóm trên Web (Ví dụ: `ds_mon(Danh sách môn học).ma_mon`).

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
| **Dropdown API** | `field_name:api(endpoint)` | Lấy dữ liệu từ API Backend. Hỗ trợ `hoc_ky` và `hoc_phan` (hoặc `mon_hoc`). | `ky_hoc:api(hoc_ky)` |

### 3.2. Các cờ bổ trợ (Modifiers)

Các cờ này gắn trực tiếp vào cuối tên biến. Có thể kết hợp nhiều cờ (VD: `~*` là Textarea bắt buộc nhập).

| Cờ | Ý nghĩa | Hành vi trên UI | Cú pháp ví dụ |
| :---: | :--- | :--- | :--- |
| `*` | **Bắt buộc (Required)** | Render dấu hoa thị đỏ, chặn xuất file nếu để trống. | `ho_ten*` |
| `!` | **Chỉ đọc (Readonly)** | Render ô màu xám, khóa chỉnh sửa. (Dùng cho thông tin sinh viên). | `ma_sv!` |
| `~` | **Nhiều dòng (Textarea)** | Thay vì khung nhập nhỏ, sẽ render khung text lớn tự động co giãn. | `ly_do:text~*` |
| `@` | **Tự động điền (Auto-fill)** | Hiển thị icon tia chớp ⚡ hoặc đũa phép ở nhãn nhập liệu, báo hiệu dữ liệu tự động đồng bộ. | `ten_mon@!` |

### 3.3. Cấu hình bảng lặp (Array/Table)

Để hiển thị một bảng dữ liệu cho phép người dùng thêm/bớt dòng động trên Web (ví dụ: danh sách môn rút học phần, danh sách thiết bị v.v.), tất cả Content Control trong dòng đó phải được cấu hình tiền tố nhóm theo cú pháp:

```text
[group_name]([group_title]).[field_name]:[type][modifiers]
```

**Ví dụ cấu hình thực tế cho một dòng của bảng Danh sách môn học:**
- Ở cột "Mã học phần", đặt Tag: `ds_mon(Danh sách môn).ma_hp:api(hoc_phan)*`
- Ở cột "Tên môn học", đặt Tag: `ds_mon(Danh sách môn).ten_mon@!`
- Ở cột "Số tín chỉ", đặt Tag: `ds_mon(Danh sách môn).so_tin_chi:number@!`

**Giao diện sinh ra:** Hệ thống sẽ tự động hiển thị một Block có tiêu đề **"Danh sách môn"** kèm nút **"+ Thêm dòng"**, các ô nhập tương ứng sẽ được hiển thị gọn gàng bên dưới. Khi xuất file Word, hệ thống sẽ tự động nhân bản dòng dữ liệu này tương ứng với số dòng sinh viên nhập.

#### Tính năng Auto-fill (Tự động điền) khi liên kết API:
Khi bạn sử dụng kiểu dữ liệu `api(hoc_phan)` (hoặc `api(mon_hoc)`) cho cột mã học phần bên trong bảng dữ liệu mảng lặp (Ví dụ: `ds_mon(Danh sách môn).ma_mon:api(hoc_phan)`):
- Khi sinh viên chọn một môn học từ danh sách, hệ thống sẽ tự động đối chiếu và điền **Tên môn học** (cho các cột có nhãn chứa chữ `ten_mon` hoặc `ten_hp`) và **Số tín chỉ** (cho các cột có nhãn chứa chữ `tin_chi` hoặc `so_tc`) trên cùng dòng đó. Sinh viên không cần phải tự gõ thủ công các thông tin này.

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
| Sinh viên tự chọn **Học kỳ**, tự động hiển thị Năm học đi kèm. | `Học kỳ` | `hoc_ky:api(hoc_ky)` và `nam_hoc!` | Dropdown chọn Học kỳ. Tự hiển thị Số kỳ (VD: `1`) và Năm học (VD: `2022-2023`) trên bản xem trước. |
| Cam kết **Đồng ý điều khoản**, bắt buộc. | `Xác nhận cam kết` | `cam_ket:bool*` | Checkbox tích chọn. |

> **Mẹo cấu hình Học kỳ & Năm học:** 
> Chỉ cần tạo 2 ô Content Control trong Word:
> 1. Ô Học kỳ đặt Tag: `hoc_ky:api(hoc_ky)` (trên Web sẽ sinh ra 1 Select duy nhất có danh sách dạng *"Học kỳ 1 - Năm học 2022-2023"*).
> 2. Ô Năm học đặt Tag: `nam_hoc!` (chỉ đọc).
> Hệ thống sẽ tự động bóc tách: ô Học kỳ trong Word chỉ hiển thị số học kỳ (ví dụ: `1`), còn ô Năm học tự điền năm học tương ứng (ví dụ: `2022-2023`).

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
