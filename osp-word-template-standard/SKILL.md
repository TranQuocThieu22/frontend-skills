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

#### Tính năng Auto-fill (Tự động điền) và Cơ chế Khớp Dữ Liệu:
Khi cấu hình cờ `@` (ví dụ `ten_mon@!`), hệ thống Web sẽ không yêu cầu sinh viên tự nhập mà sẽ tự động điền giá trị dựa vào trường API Select nằm cùng cấp (ví dụ `ma_mon:api(hoc_phan)`).

**Quy tắc ưu tiên tự động điền (Ví dụ cho API `hoc_phan`):**
1. **Khớp chính xác (Exact Mapping - Khuyên dùng):** Hệ thống sẽ ưu tiên tìm trong cấu trúc JSON trả về từ API xem có thuộc tính nào trùng tên chính xác với `field_name` của bạn hay không. 
   - Ví dụ: API trả về object `{ "ma_mon": "IT123", "ten_mon": "Toán", "so_tin_chi": 3, "nhom_to": "01" }`.
   - Nếu bạn đặt Tag là `ds_mon.so_tin_chi@!`, hệ thống tự động điền `3`.
   - Nếu bạn đặt Tag là `ds_mon.nhom_to@!`, hệ thống tự điền `"01"`.

2. **Khớp tương đối (Heuristic Fallback):** Nếu không tìm thấy key chính xác, hệ thống sẽ cố gắng đoán nội dung thông qua tên thẻ của bạn:
   - Nếu tên thẻ chứa `ten_mon` hoặc `ten_hp`: Lấy tên môn học.
   - Nếu tên thẻ chứa `ma_mon` hoặc `ma_hp`: Lấy mã môn học.
   - Nếu tên thẻ chứa `tin_chi` hoặc `so_tc`: Lấy số tín chỉ.
   - Nếu tên thẻ chứa `nhom_to`, `nhom_hp`, hoặc `nhom`: Lấy nhóm tổ.

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

## 5. Đặc tả API Select và Thuộc tính tự động điền (Auto-fill Properties)

Hệ thống cung cấp một số API chuyên dụng để tự động kéo dữ liệu (ví dụ: Học kỳ, Học phần). Để dữ liệu đồng bộ chính xác giữa Form và Word, hãy dùng đúng tên biến (field_name).

> **💡 MẸO - Magic Keys (Cú pháp viết tắt / Auto-inference):**
> Để tăng tốc độ làm mẫu Word, hệ thống hỗ trợ tự động nội suy (auto-inference) cho các biến phổ biến. 
> - Nếu bạn đặt đúng tên biến là `hoc_ky` (ví dụ `hoc_ky` hoặc `hoc_ky*`), hệ thống sẽ tự động ngầm hiểu là `hoc_ky:api(hoc_ky)`. Bạn không cần gõ dài dòng!
> - Nếu bạn muốn đặt tên biến khác đi (ví dụ `ky_bat_dau`), bạn **vẫn phải** dùng cú pháp tường minh `ky_bat_dau:api(hoc_ky)` để gọi API.

### 5.1. API `hoc_ky`
- **Cách dùng viết tắt (Khuyên dùng):** Chỉ cần đặt Tag là `hoc_ky` (nếu bắt buộc thì `hoc_ky*`). Hệ thống tự ép kiểu thành Select API.
- **Cách dùng tường minh:** Đặt Tag `ky_hoc:api(hoc_ky)` (dành cho khi bạn muốn dùng tên biến khác `hoc_ky`).
- **Hoạt động:** Hiển thị dropdown chứa danh sách các học kỳ, ví dụ "Học kỳ 1 - 2023-2024". Khi xuất Word, thẻ này sẽ hiển thị số Học kỳ (vd: `1`).
- **Auto-fill liên quan:** 
  - `nam_hoc` (Năm học): Khi người dùng chọn một học kỳ, nếu trong mẫu Word có Content Control mang Tag `nam_hoc!` hoặc `nam_hoc@!`, hệ thống tự động trích xuất chuỗi năm học (vd: `2023-2024`) và điền vào thẻ này.

### 5.2. API `hoc_phan` (hoặc `mon_hoc`)
- **Cách dùng viết tắt:** Chỉ cần đặt Tag `ma_mon*`. Hệ thống sẽ tự động ép kiểu thành `api(hoc_phan)`.
- **Cách dùng tường minh:** Đặt Tag `mon_hoc_thay_the:api(hoc_phan)*` (dành cho tên biến tùy chỉnh).
- **Hoạt động:** Hiển thị dropdown tìm kiếm toàn bộ danh sách điểm và danh mục môn học của sinh viên. Tùy vào tên thẻ (chứa chữ `ten` hay không) mà giao diện sẽ cho tìm theo Tên hay Mã môn.
- **Các thuộc tính Auto-fill (`@`) trả về từ API:**
  Khi cấu hình các thẻ cùng cấp (cùng 1 dòng trong bảng), bạn có thể dùng các tên thẻ sau kèm cờ `@!` để lấy thẳng dữ liệu từ API:
  - `ma_mon` hoặc `ma_hp`: Mã môn học (vd: "IT123").
  - `ten_mon` hoặc `ten_hp`: Tên môn học (vd: "Kế toán doanh nghiệp mỏ").
  - `so_tin_chi` hoặc `tin_chi`: Số tín chỉ (vd: "3").
  - `nhom_to` hoặc `nhom`: Nhóm tổ (vd: "01").

---

## 6. Tổng hợp Ví Dụ Cấu Hình Chuẩn (Thực Tế)

Dưới đây là bảng ví dụ tổng hợp cho **tất cả** các trường hợp thiết lập Tag phổ biến, đảm bảo tương thích 100% với WPS Office và hệ thống tự động điền:

| Yêu cầu / Ngữ cảnh | Cấu hình Title (Nhãn) | Cấu hình Tag (Data & Cấu hình) | Kết quả UI / Word |
| :--- | :--- | :--- | :--- |
| **1. Thông tin sinh viên (Auto-sync)** | `Họ và tên` | `ho_ten!` | Text xám (chỉ đọc), điền sẵn thông tin. |
| **2. Bắt buộc nhập, nhiều dòng** | `Lý do xin phép` | `ly_do:text~*` | Textarea, có sao đỏ `*`. |
| **3. Dropdown tĩnh** | `Ca học` | `ca_hoc:select(1:Sáng,2:Chiều)*` | Select tĩnh (Sáng / Chiều). |
| **4. Chọn Học kỳ & tự điền Năm học** | `Học kỳ` (Ô 1) <br> `Năm học` (Ô 2) | `hoc_ky:api(hoc_ky)` <br> `nam_hoc@!` | Ô Học kỳ cho phép chọn API. Ô Năm học tự động điền (VD: `2023-2024`). |
| **5. Bảng: Thêm danh sách môn (Có API)** | `Mã học phần` (Cột 1) <br> `Tên môn` (Cột 2) <br> `Số TC` (Cột 3) <br> `Nhóm` (Cột 4) | `ds_mon.ma_hp:api(hoc_phan)*` <br> `ds_mon.ten_mon@!` <br> `ds_mon.so_tin_chi@!` <br> `ds_mon.nhom_to@!` | Nút `+ Thêm dòng`. Khi chọn mã môn ở Cột 1, Cột 2, 3, 4 tự động được điền dữ liệu (có icon ⚡ auto-fill). |
| **6. Checkbox xác nhận** | `Cam kết` | `cam_ket:bool*` | Ô Checkbox bắt buộc tích. |
| **7. Nhập Ngày tháng** | `Ngày nộp` | `ngay_nop:date*` | Bảng DatePicker. |

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
