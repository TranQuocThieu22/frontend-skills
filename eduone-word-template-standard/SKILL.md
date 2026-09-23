---
name: eduone-word-template-standard
description: Quy tắc đặt Title và Tag cho file Word mẫu của EduOne (đơn từ), danh sách các tên biến hệ thống tự điền, để hệ thống tự sinh form nhập liệu và xuất đơn.
---

# Chuẩn mẫu Word của EduOne

Người soạn mẫu đặt biến vào file `.docx` bằng **Content Control** của Word. Hệ thống đọc các biến này để:

1. Tự sinh form nhập liệu trên web cho sinh viên.
2. Ghi giá trị sinh viên nhập ngược lại vào file Word khi nộp đơn.

> **Bản gốc** của tài liệu nằm ở `apps/eduone/docs/eduone-word-template-standard.md`. File
> `.agents/skills/eduone-word-template-standard/SKILL.md` là bản sao y hệt cho AI agent — sửa bên nào
> thì chép sang bên kia.
>
> **Dành cho lập trình viên:** mọi hằng số (kiểu dữ liệu, khóa tự điền, nguồn API, nhãn nhóm) khai
> báo tại `apps/eduone/src/features/service-registration/shared/templateSchema.ts`. Thêm hoặc đổi thì
> sửa ở đó rồi cập nhật tài liệu này.

---

## 0. Tra cứu nhanh — các tên biến hệ thống hiểu sẵn

Đặt Tag **đúng y** các tên dưới đây thì hệ thống tự làm việc tương ứng. Mọi tên khác đều là ô sinh
viên tự nhập, theo kiểu dữ liệu bạn khai (mục 3).

### A. Tự điền từ hồ sơ sinh viên

| Tên biến | Nội dung | Tag khuyên dùng |
| :--- | :--- | :--- |
| `ho_ten` | Họ và tên | `ho_ten!` |
| `ma_sv` | Mã sinh viên | `ma_sv!` |
| `ngay_sinh` | Ngày sinh | `ngay_sinh:date!` |
| `noi_sinh` | Nơi sinh | `noi_sinh!` |
| `lop` | Lớp | `lop!` |
| `nien_khoa` | Niên khóa | `nien_khoa!` |
| `khoa` | Khoa quản lý | `khoa!` |
| `nganh` | Ngành | `nganh!` |
| `dien_thoai` | Điện thoại | `dien_thoai` |
| `email` | Email | `email` |
| `so_cmnd` | Số CMND | `so_cmnd` |
| `ngay_cap_cmnd` | Ngày cấp CMND | `ngay_cap_cmnd:date` |
| `noi_cap_cmnd` | Nơi cấp CMND | `noi_cap_cmnd` |
| `so_cccd` | Số CCCD | `so_cccd` |
| `ngay_cap_cccd` | Ngày cấp CCCD | `ngay_cap_cccd:date` |
| `noi_cap_cccd` | Nơi cấp CCCD | `noi_cap_cccd` |
| `so_ho_chieu` | Số hộ chiếu | `so_ho_chieu` |
| `ngay_cap_ho_chieu` | Ngày cấp hộ chiếu | `ngay_cap_ho_chieu:date` |
| `ngay_het_han_ho_chieu` | Ngày hết hạn hộ chiếu | `ngay_het_han_ho_chieu:date` |

Chi tiết và lưu ý ở [mục 5](#5-thông-tin-sinh-viên-tự-điền).

### B. Dropdown lấy dữ liệu từ hệ thống

| Tên biến | Sinh ra | Ghi ra file Word |
| :--- | :--- | :--- |
| `hoc_ky` | Dropdown học kỳ của sinh viên | Số kỳ, ví dụ `1` |
| `ma_mon` | Dropdown môn học của sinh viên | Mã môn, ví dụ `IT123` |

Chi tiết ở [mục 6](#6-học-kỳ-và-môn-học).

### C. Tự điền theo ô khác

| Tên biến | Điền khi nào | Giá trị |
| :--- | :--- | :--- |
| `nam_hoc` | Mẫu có ô `hoc_ky` và sinh viên đã chọn học kỳ | `2023-2024` |
| `ten_mon` | Nằm **cùng dòng bảng lặp** với ô `ma_mon`, sinh viên đã chọn môn | Tên môn |
| `so_tin_chi` | như trên | Số tín chỉ |
| `nhom_to` | như trên | Nhóm tổ |
| `ten_mon_eg` | như trên — chỉ có với môn **đã có điểm** | Tên môn tiếng Anh |

Chỉ nhận **đúng** các tên trên. `ma_hp`, `ten_hp`, `tin_chi`, `nhom`, `fullname`, `mssv`, `class`,
`ngay_cap`... đều **không** được điền tự động.

---

## 1. Hai ô cần điền trong Word

Mỗi Content Control có hai ô trong **Properties**:

| Ô | Vai trò | Ví dụ |
| :--- | :--- | :--- |
| **Title** | Nhãn hiển thị cho sinh viên. Đặt tự do, có dấu tiếng Việt. | `Lý do xin nghỉ học` |
| **Tag** | Định nghĩa cho máy: tên biến, kiểu dữ liệu, cờ. **Phải đúng cú pháp.** | `ly_do:textarea*` |

**Luôn điền cả hai ô.** Bỏ trống Tag thì hệ thống lấy Title làm tên biến, chữ có dấu và khoảng trắng
bị đổi thành `_`: `Họ và tên` thành biến `h__v__t_n`. Vẫn chạy, nhưng sẽ **không** được tự điền và
không ai đọc hiểu được tên biến đó.

---

## 2. Cú pháp Tag

```
[nhóm(Tiêu đề nhóm).]tên_biến[:kiểu[(tham số)]][cờ]
```

Chỉ **tên_biến** là bắt buộc. Tách một Tag đầy đủ ra từng phần:

```
ds_mon(Danh sách môn học).ma_mon:api(hoc_phan)*#2
└──────────┬────────────┘ └─┬──┘ └─────┬─────┘└┬┘
   nhóm (bảng lặp)      tên biến  kiểu + tham số  cờ: * bắt buộc, #2 hiện ở vị trí 2
```

Một số Tag hay gặp:

```
ho_ten!                            → ô chữ, tự điền họ tên, khóa sửa
ly_do:textarea*                    → khung nhiều dòng, bắt buộc
ngay_nghi:date*                    → chọn ngày, bắt buộc
so_tien:number                     → ô nhập số
cam_ket:bool*                      → ô tích chọn
ca_hoc:select(Sáng,Chiều)*         → dropdown cố định
hoc_ky*                            → dropdown học kỳ
ds_mon(Danh sách môn học).ma_mon*  → cột "mã môn" trong bảng lặp
```

**Quy tắc đặt tên biến:** chữ thường, không dấu, ngăn cách bằng `_` (`ho_ten`, `so_tin_chi`). Ký
tự khác bị đổi thành `_`.

---

## 3. Kiểu dữ liệu

| Kiểu | Giao diện sinh ra | Ghi ra file Word | Ví dụ Tag |
| :--- | :--- | :--- | :--- |
| `text` *(mặc định)* | Ô nhập chữ một dòng | Chữ đã nhập | `noi_tam_tru` |
| `textarea` | Khung nhập nhiều dòng, tự co giãn | Chữ đã nhập | `ly_do:textarea` |
| `number` | Ô nhập số | Số, **không** có dấu phân cách hàng nghìn (`1500000`) | `so_tien:number` |
| `date` | Bảng chọn lịch | `DD/MM/YYYY` | `ngay_nop:date` |
| `bool` | Ô tích chọn | `☑` hoặc `☐` | `cam_ket:bool` |
| `phone` | Ô nhập chữ *(chưa kiểm tra định dạng)* | Chữ đã nhập | `sdt_phu_huynh:phone` |
| `email` | Ô nhập chữ *(chưa kiểm tra định dạng)* | Chữ đã nhập | `email_lien_he:email` |
| `select` | Dropdown danh sách cố định | Xem bên dưới | `ca_hoc:select(Sáng,Chiều)` |
| `api` | Dropdown lấy dữ liệu từ hệ thống | Xem [mục 6](#6-học-kỳ-và-môn-học) | `ky_hoc:api(hoc_ky)` |
| `array` | Bảng thêm/bớt dòng — **không viết tay**, xem [mục 7](#7-bảng-lặp) | — | — |

Gõ sai tên kiểu (ví dụ `:datetime`, `:text~`) thì tự lùi về `text` và ghi cảnh báo
`[EduOne Template]` trong Console trình duyệt (F12). Đơn vẫn nộp được, nhưng nên sửa.

### Danh sách lựa chọn của `select`

| Cách viết | Dropdown hiện | Lưu và ghi ra Word |
| :--- | :--- | :--- |
| `select(Sáng,Chiều)` | Sáng / Chiều | `Sáng` / `Chiều` |
| `select(1:Sáng,2:Chiều)` | Sáng / Chiều | `1` / `2` |

> **Khuyên dùng dạng rút gọn** `select(Sáng,Chiều)`. Với dạng `giá_trị:nhãn`, file Word và màn xử lý
> thủ tục hiện **mã** (`1`, `2`) chứ không hiện nhãn — chỉ dùng khi thật sự cần lưu mã.

Nhãn chứa dấu phẩy hoặc hai chấm thì đặt `\` phía trước:
`select(Còn hạn,Hết hạn\, cần gia hạn)`.

---

## 4. Cờ

Gắn vào **cuối** Tag, sau phần kiểu. Ghép được nhiều cờ, thứ tự bất kỳ (`@!` và `!@` như nhau).

| Cờ | Tên | Tác dụng thực tế | Ví dụ |
| :---: | :--- | :--- | :--- |
| `*` | Bắt buộc | Hiện dấu `*` đỏ cạnh nhãn. ⚠️ Hiện **chưa chặn** lưu khi bỏ trống. | `ly_do:textarea*` |
| `!` | Chỉ đọc | Ô bị khóa (xám), sinh viên không sửa được | `ma_sv!` |
| `@` | Hệ thống điền | Báo đây là ô do hệ thống điền. Ở chế độ **"Chỉ nhập liệu"** ô này bị ẩn khỏi danh sách cho gọn (giá trị vẫn ghi ra file) | `ten_mon@!` |
| `#` | Hiện ở danh sách | Đưa trường vào cột **Dữ liệu từ file** của màn xử lý thủ tục | `ly_do:textarea#` |
| `#N` | Hiện, ép vị trí | Như `#`, nhưng đứng ở vị trí `N` | `ly_do:textarea#2` |

✅ `ly_do:textarea*` — ❌ `ly_do*:textarea` (cờ đứng trước kiểu sẽ bị hiểu thành một phần tên biến).

Cờ `~` (khung nhiều dòng) của bản chuẩn cũ **đã bỏ** — dùng kiểu `textarea`. Tag cũ `ly_do:text~*` giờ
chỉ ra ô một dòng kèm cảnh báo.

**Nên dùng `@!` cho mọi ô do hệ thống điền** (năm học, tên môn, số tín chỉ...): `!` để sinh viên
không sửa sai, `@` để form gọn lại.

### Cờ `#` — trường hiện ở màn xử lý thủ tục

Cán bộ xử lý cần xem nhanh vài thông tin chính ngay trên danh sách, không phải mở từng đơn:

```
Title: Lý do xin rút   Tag: ly_do:textarea*#
Title: Từ ngày         Tag: tu_ngay:date#
Title: Đến ngày        Tag: den_ngay:date#
```

Thứ tự lấy theo thứ tự các ô trong file Word. Muốn ép thứ tự khác thì ghi số:

```
Title: Từ ngày         Tag: tu_ngay:date#1
Title: Lý do xin rút   Tag: ly_do:textarea*#2
```

- **Số chỉ để sắp xếp**, không phải hạn mức — `#6`, `#9` đều được.
- **Đã ghi số thì ghi số cho tất cả.** `#` không số được đánh số tự động 1, 2, 3... theo thứ tự trong
  mẫu, nên trộn `#` với `#1` dễ ra hai trường cùng vị trí.
- **Không đánh cờ thì ẩn** khỏi danh sách. Trường vẫn lưu đủ, mở chi tiết đơn vẫn thấy.
- **Mẫu không đánh cờ nào thì cột ghi "Không có".** Hệ thống không tự đoán.
- **Tối đa 5 mục**, vượt thì lấy 5 mục đầu.
- **Cả một bảng lặp tính là một mục**, đứng ở vị trí số nhỏ nhất của nó. Đánh `#` lên từng cột muốn
  hiện; ô trong danh sách vẽ tối đa 3 dòng, thừa thì ghi *"… còn N dòng nữa"*.

```
Title: Mã học phần   Tag: ds_mon(Danh sách học phần).ma_mon*#4
Title: Tên học phần  Tag: ds_mon(Danh sách học phần).ten_mon@!#5
Title: Số TC         Tag: ds_mon(Danh sách học phần).so_tin_chi@!#6
```

Ba cột trên chiếm **một** trong năm mục, ở vị trí 4.

---

## 5. Thông tin sinh viên tự điền

Đặt Tag đúng tên ở [bảng 0.A](#a-tự-điền-từ-hồ-sơ-sinh-viên) là hệ thống lấy hồ sơ của người đang
đăng nhập điền vào. Mỗi thông tin chỉ có **một** tên biến duy nhất, không nhận tên thay thế.

**Thông tin học vụ** (`ho_ten`, `ma_sv`, `ngay_sinh`, `noi_sinh`, `lop`, `nien_khoa`, `khoa`, `nganh`):
luôn có sẵn, **nên gắn `!`** để sinh viên không sửa lệch thông tin gốc.

**Liên lạc và giấy tờ tùy thân** (`dien_thoai`, `email`, các trường CMND/CCCD/hộ chiếu): lấy từ màn
**Cập nhật thông tin lý lịch**, sinh viên chưa khai thì trống. **Không gắn `!`** cho nhóm này.

> ⚠️ Mọi trường trong bảng 0.A đều bị ẩn ở chế độ **"Chỉ nhập liệu"** (chế độ mặc định khi mở form),
> kể cả khi đang trống. Sinh viên thiếu CCCD phải chuyển sang **"Điền trên biểu mẫu"** hoặc
> **"Chia đôi"** mới nhập được.

Tên biến CMND luôn có hậu tố `_cmnd`: viết `ngay_cap_cmnd`, **không** viết `ngay_cap`.

```
Title: Họ và tên      Tag: ho_ten!
Title: Mã SV          Tag: ma_sv!
Title: Ngày sinh      Tag: ngay_sinh:date!
Title: Số CCCD        Tag: so_cccd
Title: Ngày cấp       Tag: ngay_cap_cccd:date
Title: Nơi cấp        Tag: noi_cap_cccd
```

---

## 6. Học kỳ và môn học

Hệ thống có **hai** nguồn dữ liệu, khai bằng `:api(tên_nguồn)`. Hai tên biến `hoc_ky` và `ma_mon` là
viết tắt, tự hiểu thành `api`, không cần ghi.

| Viết tắt | Viết đầy đủ tương đương |
| :--- | :--- |
| `hoc_ky` | `hoc_ky:api(hoc_ky)` |
| `ma_mon` | `ma_mon:api(hoc_phan)` |

Khai nguồn không tồn tại (ví dụ `:api(khoa)`) thì ô đó thành ô nhập tay kèm icon cảnh báo — đơn vẫn
nộp được.

### 6.1. Học kỳ — `hoc_ky`

Dropdown các học kỳ của sinh viên, hiện dạng "Học kỳ 1 - 2023-2024". Trong file Word ô này ghi **số
kỳ** (`1`), không ghi mã 5 số.

Kèm ô `nam_hoc` để có năm học:

```
Title: Học kỳ    Tag: hoc_ky*
Title: Năm học   Tag: nam_hoc@!
```

Sinh viên chọn "Học kỳ 1 - 2023-2024", ô Năm học tự hiện `2023-2024`.

> **Luôn đặt tên biến là `hoc_ky`.** Đặt tên khác (`ky_bat_dau:api(hoc_ky)`) thì dropdown vẫn chạy,
> nhưng file Word ghi mã 5 số (`20231`) thay vì `1`, và danh sách môn **không** lọc theo học kỳ đó.

### 6.2. Môn học — `ma_mon` hoặc `api(hoc_phan)`

Dropdown tìm kiếm trong bảng điểm và danh mục môn học của sinh viên.

- Mẫu **có** ô `hoc_ky`: phải chọn học kỳ trước, danh sách chỉ gồm các môn đã học trong kỳ đó.
- Mẫu **không có** ô `hoc_ky`: chọn trong toàn bộ danh mục môn học.
- Tên biến chứa chữ `ten` (ví dụ `ten_mon_xin_hoc:api(hoc_phan)`) thì tìm và lưu **tên môn**; còn lại
  tìm và lưu **mã môn**.

**Tự điền các cột khác chỉ chạy trong bảng lặp.** Chọn môn ở cột `ma_mon`, các cột **cùng dòng** có
tên trong [bảng 0.C](#c-tự-điền-theo-ô-khác) (`ten_mon`, `so_tin_chi`, `nhom_to`, `ten_mon_eg`) được
điền theo. Tên phải trùng **đúng** — `ma_hp`, `tin_chi`, `nhom` không được điền.

Một ô chọn môn đứng riêng (ngoài bảng) chỉ là dropdown, không kéo theo ô nào:

```
Title: Môn học thay thế   Tag: mon_thay_the:api(hoc_phan)*
```

---

## 7. Bảng lặp

Khi cần danh sách nhiều dòng (danh sách môn xin rút, danh sách thành viên...), đặt **cùng một tiền
tố nhóm** cho mọi Content Control trong dòng đó:

```
nhóm(Tiêu đề nhóm).tên_biến[:kiểu][cờ]
```

Ví dụ dòng bảng "Danh sách môn học" trong Word:

| STT | Mã môn | Tên môn | Số TC | Nhóm |
| :--- | :--- | :--- | :--- | :--- |
| | `[Mã học phần]` | `[Tên môn học]` | `[Số TC]` | `[Nhóm]` |

Các Content Control trong dòng:

| Cột | Title | Tag |
| :--- | :--- | :--- |
| Mã môn | `Mã học phần` | `ds_mon(Danh sách môn học).ma_mon*` |
| Tên môn | `Tên môn học` | `ds_mon(Danh sách môn học).ten_mon@!` |
| Số TC | `Số TC` | `ds_mon(Danh sách môn học).so_tin_chi@!` |
| Nhóm | `Nhóm` | `ds_mon(Danh sách môn học).nhom_to@!` |

Trên web hiện một khối "Danh sách môn học" kèm nút **+ Thêm dòng**. Sinh viên chỉ chọn mã môn, ba cột
còn lại tự điền. Khi xuất Word, dòng trong bảng được nhân bản theo số dòng đã nhập.

- **Tiêu đề nhóm** lấy từ ô **đầu tiên** của nhóm trong file. Để chắc chắn, khai giống nhau ở mọi ô.
- Tiêu đề nhóm **không được chứa** dấu `.` hoặc `:` — sẽ làm hỏng cả Tag.
- Bỏ trống tiêu đề thì hệ thống dùng nhãn có sẵn cho `ds_mon`, `ds_mon_hoc`, `ds_hoc_phan`,
  `ds_sinh_vien`, `ds_thanh_vien`; nhóm khác lấy chính tên nhóm làm nhãn (`ds_thiet_bi` →
  "Ds thiet bi").
- Dòng sinh viên bấm thêm nhưng bỏ trống hết thì không được lưu.

---

## 8. Ví dụ trọn một mẫu: Đơn xin rút học phần

Văn bản trong Word (`[...]` là Content Control, ghi theo Title):

> **ĐƠN XIN RÚT HỌC PHẦN**
>
> Họ và tên: `[Họ và tên]` Mã SV: `[Mã SV]` Lớp: `[Lớp]`
> Khoa: `[Khoa]` Điện thoại: `[Điện thoại]`
>
> Em xin rút các học phần sau trong học kỳ `[Học kỳ]` năm học `[Năm học]`:
>
> | Mã HP | Tên học phần | Số TC | Nhóm |
> | --- | --- | --- | --- |
> | `[Mã HP]` | `[Tên HP]` | `[Số TC]` | `[Nhóm]` |
>
> Lý do: `[Lý do]`
>
> `[Cam kết]` Em cam kết thông tin trên là đúng sự thật.
>
> Ngày `[Ngày làm đơn]`

Bảng cấu hình:

| Title | Tag | Ghi chú |
| :--- | :--- | :--- |
| Họ và tên | `ho_ten!` | Tự điền, khóa |
| Mã SV | `ma_sv!` | Tự điền, khóa |
| Lớp | `lop!` | Tự điền, khóa |
| Khoa | `khoa!` | Tự điền, khóa |
| Điện thoại | `dien_thoai` | Tự điền nếu có, cho sửa |
| Học kỳ | `hoc_ky*` | Dropdown học kỳ |
| Năm học | `nam_hoc@!` | Tự điền theo học kỳ |
| Mã HP | `ds_mon(Học phần xin rút).ma_mon*#2` | Dropdown môn, lọc theo học kỳ |
| Tên HP | `ds_mon(Học phần xin rút).ten_mon@!#2` | Tự điền theo mã |
| Số TC | `ds_mon(Học phần xin rút).so_tin_chi@!` | Tự điền theo mã |
| Nhóm | `ds_mon(Học phần xin rút).nhom_to@!` | Tự điền theo mã |
| Lý do | `ly_do:textarea*#1` | Nhập tay, hiện ở danh sách vị trí 1 |
| Cam kết | `cam_ket:bool*` | Ô tích |
| Ngày làm đơn | `ngay_lam_don:date*` | Chọn ngày |

Kết quả: sinh viên mở form chỉ phải chọn học kỳ, chọn môn, nhập lý do, tích cam kết và chọn ngày.
Cán bộ xử lý thấy ngay "Lý do" và bảng "Mã HP / Tên HP" trên danh sách.

---

## 9. Lỗi thường gặp

| Sai | Hậu quả | Đúng |
| :--- | :--- | :--- |
| Tag để trống | Tên biến thành `h__v__t_n`, không tự điền | `ho_ten!` |
| `ly_do*:textarea` | Cờ bị hiểu thành tên biến | `ly_do:textarea*` |
| `ly_do:text~*` | Cờ `~` đã bỏ, ra ô một dòng | `ly_do:textarea*` |
| `mssv!`, `fullname!` | Không tự điền | `ma_sv!`, `ho_ten!` |
| `ngay_cap:date` | Không tự điền | `ngay_cap_cmnd:date` |
| `ds_mon.ma_hp:api(hoc_phan)` + `ds_mon.ten_hp@!` | Cột tên không tự điền | `ds_mon.ma_mon` + `ds_mon.ten_mon@!` |
| `ky_hoc:api(hoc_ky)` | Word ghi `20231`, môn không lọc theo kỳ | `hoc_ky` |
| `gioi_tinh:select(1:Nam,2:Nữ)` | Word ghi `1`/`2` | `gioi_tinh:select(Nam,Nữ)` |
| `ds_mon(DS môn. HK1).ma_mon` | Dấu `.` trong tiêu đề làm hỏng Tag | `ds_mon(DS môn HK1).ma_mon` |
| Hai bảng khác nhau cùng tên nhóm `ds_mon` | Gộp thành một bảng | `ds_mon_rut`, `ds_mon_dang_ky` |

---

## 10. Thao tác trong Word

1. **Bật tab Developer:** `File → Options → Customize Ribbon` → tick **Developer** ở cột phải.
2. **Chèn biến:** đặt con trỏ vào vị trí cần điền, tab **Developer** → **Plain Text Content Control**
   (hoặc Rich Text).
3. **Đặt Title và Tag:** chọn control vừa chèn → **Properties** → điền **Title** và **Tag**.
4. **Lưu** `.docx` và upload lên hệ thống.
5. **Kiểm tra:** vào **Đăng ký dịch vụ trực tuyến**, chọn đúng loại dịch vụ, bấm **Sửa** ở ô
   "Điền thông tin đơn từ". Rê chuột lên nhãn một ô sẽ thấy Tag gốc của nó — dùng để đối chiếu với
   file Word.

Cùng một tên biến đặt ở nhiều chỗ trong file là hợp lệ: form chỉ hiện **một** ô, mọi vị trí trong
Word nhận cùng giá trị (ví dụ `khoa` ở cả "Kính gửi" lẫn dòng thông tin sinh viên).

### Tự kiểm trước khi bàn giao mẫu

- [ ] Mọi Content Control đều có **cả Title lẫn Tag**.
- [ ] Tên biến viết thường không dấu, dùng `_`; tên tự điền khớp đúng [mục 0](#0-tra-cứu-nhanh--các-tên-biến-hệ-thống-hiểu-sẵn).
- [ ] Trường bắt buộc có `*`; ô hệ thống điền có `@!`; thông tin học vụ có `!`.
- [ ] Ô trong cùng một bảng dùng **cùng một tên nhóm**, bảng khác nhau dùng tên nhóm khác nhau.
- [ ] Đã đánh `#` cho các trường cán bộ cần xem nhanh (tối đa 5).
- [ ] Mở F12 → Console, không có cảnh báo `[EduOne Template]`.

---

## 11. Ghi chú kỹ thuật (cho lập trình viên)

**File liên quan** (trong `apps/eduone/src/features/service-registration/`):

| File | Vai trò |
| :--- | :--- |
| `shared/templateSchema.ts` | Hằng số: `FIELD_TYPES`, `MAGIC_KEYS`, `GROUP_TITLE_MAP`, `STUDENT_PRESET_KEYS` |
| `utils/word-utils.ts` | `parseWordTag`, `buildFieldKey`, `extractFieldsFromWord` |
| `components/api-selects/` | `API_SELECT_REGISTRY`: `HocKySelect`, `HocPhanSelect`, `FallbackApiSelect` |
| `shared/fieldValue.ts` | `getDisplayValue`: suy `hoc_ky` → số kỳ, `nam_hoc` từ mã học kỳ |
| `shared/thongTinBoSung.ts` | Trải phẳng dữ liệu gửi `sms/w-savesvdkgcn`, tính vị trí cờ `#` |
| `components/DocxViewer.tsx` | Xem trước và ghi giá trị vào file docx |

**Khóa của trường sinh ở hai chiều** — đọc mẫu ra form và ghi token `{{...}}` vào file. Cả hai
**bắt buộc** dùng chung `buildFieldKey()`. Lệch nhau một quy tắc nhỏ là ô trong file bị bỏ trống mà
không báo lỗi.

**WPS Office.** Content Control rỗng trong file WPS không có `<w:t>`. Code trong `DocxViewer` và
`extractFieldsFromWord` **không được** giả định mọi `w:sdt` đều chứa `w:t`; phải tự tạo `w:t` khi
thiếu, và đặt **bên trong** `w:p` sẵn có — nếu không `docx-preview` bỏ qua nội dung.

**Thêm nguồn API mới:** viết component trong `components/api-selects/`, đăng ký vào
`API_SELECT_REGISTRY`, rồi cập nhật mục 0.B và mục 6 của tài liệu này.

**Hạn chế đã biết của code hiện tại** (tài liệu mô tả đúng hành vi hiện có, sửa code thì cập nhật lại):

- Cờ `*` chỉ hiện dấu sao, `useForm` trong `RegistrationExtraModal` chưa có `validate` nên không chặn lưu.
- `select` dạng `giá_trị:nhãn` ghi **giá trị** ra Word và payload; `formatDisplayValue` trong `DocxViewer` chưa tra nhãn.
- `isAutoFilledField` ẩn mọi khóa trong `STUDENT_PRESET_KEYS` ở chế độ "Chỉ nhập liệu" kể cả khi rỗng.
- Suy số kỳ / năm học trong `getDisplayValue` và lọc môn theo kỳ trong `HocPhanSelect` chỉ nhận ô tên `hoc_ky` (hoặc `ma_hk`).
- Tự điền theo môn trong `HocPhanSelect` chỉ chạy khi có `arrayPath` (trong bảng lặp), và không cần cờ `@` — khớp theo tên cột.
- Tiêu đề nhóm lấy từ ô đầu tiên gặp trong `extractFieldsFromWord`; các ô sau khai khác thì bị bỏ qua.
- `phone`, `email` chưa có kiểm tra định dạng.
