---
name: srb-gam-backend-payload
description: Thông tin về các Payload API Backend của dự án SRB và GAM (Edusoft.Api)
---

# Payload Backend SRB & GAM (Edusoft.Api)

Dự án SRB và GAM sử dụng chung Backend với `Edusoft.Api`.
2121

## 1. Lịch sử đăng ký phòng (`w-loadlichsudangkysudungphong`)

Endpoint: `POST /srb/w-loadlichsudangkysudungphong` (Controller: `PhongChucNangController.cs`)
Class model (Backend): `LichSuDangKyRequest` (File: `Edusoft.DataObjects\WEB\PhongChucNangResult.cs`)

**Các trường có thể truyền vào:**

- `tu_khoa` (string): Lọc theo mã đơn, mục đích sử dụng, hoặc loại phòng.
- `id_loai_phong` (string): Lọc theo loại phòng (Đã được Backend bổ sung).
- `id_ly_do_sd_ph` (string): Lọc theo mục đích sử dụng (Đã được Backend bổ sung).
- `loai_don` (string): Hiện tại Backend có nhận tham số này nhưng ít sử dụng.
- `trang_thai` (int?): `null` (Tất cả), `-1` (Chờ duyệt), `0` (Không duyệt), `1` (Được duyệt), `2` (Đã hủy), `3` (Quá hạn), `4` (Sắp quá hạn).
- `tu_ngay_gui_don` (string): Lọc từ ngày gửi đơn (Định dạng: `dd/MM/yyyy`).
- `den_ngay_gui_don` (string): Lọc đến ngày gửi đơn (Định dạng: `dd/MM/yyyy`).
- `ngay_su_dung` (string): Lọc theo ngày sử dụng phòng (Định dạng: `dd/MM/yyyy`). **Lưu ý:** Chỉ nhận 1 ngày duy nhất, không hỗ trợ khoảng thời gian (Range) như thiết kế giao diện gốc.
- `additional` (AdditionalDO): Chứa thông tin phân trang `paging` (`page`, `limit`).

**Kết quả trả về bổ sung:**
Backend đã bổ sung đối tượng `thong_ke` trong dữ liệu trả về của API này nhằm thống kê số lượng đơn theo từng trạng thái. Frontend đọc dữ liệu `thong_ke` để hiển thị số đếm (badge) lên các Tab (thay vì đọc từ API danh mục bộ lọc).

```json
"thong_ke": {
  "tat_ca": 1,
  "cho_duyet": 0,
  "khong_duyet": 0,
  "duoc_duyet": 1,
  "da_huy": 0,
  "qua_han": 0,
  "sap_qua_han": 0
}
```

## 2. Lỗi Logic Trạng Thái Lưới Phòng Đã Duyệt (`w-loadphongchucnangtheokhunggio`)

**Vấn đề:**
Khi duyệt một đơn đăng ký, API `w-capnhattrangthaidondangkypcn` sẽ cập nhật `IsDuyet = 1` (DaDuyet) và đồng thời thêm các bản ghi vào `KSDPHHK`, `KPHSDPHHK`, `KTUANSDPHHK` để đánh dấu đây là Lịch học chính khoá.
Sau đó, khi tải lại lưới phòng thông qua `w-loadphongchucnangtheokhunggio` (Method `BuildQueryLocPhongChucNang`), do Backend kiểm tra ưu tiên sự tồn tại của lịch chính khoá (`ban != null`) **TRƯỚC KHI** kiểm tra đơn đăng ký của user hiện tại (`dadk != null`), nên phòng vừa được duyệt luôn trả về `trang_thai_ban = 3` (`TrungLichChinhKhoa`). Frontend hiển thị trạng thái `3` bằng Dấu X Đỏ (Unavailable) thay vì Dấu Tick Xanh (Approved) đối với chính người dùng đã đặt phòng/người vừa duyệt phòng.

**Yêu cầu xử lý cho Backend (File `PhongChucNangController.cs` - dòng 1239):**
Cần đảo ngược thứ tự ưu tiên của toán tử bậc 3 (`?:`) trong việc gán `trang_thai_ban`:

1. Phải ưu tiên kiểm tra đơn của user hiện tại đã duyệt chưa: `(dadk != null && dadk.IsDuyet == (int)TRANG_THAI_DUYET_NCKH.DaDuyet)` => Trả về `(int)TrangThaiBanPhongChucNang.DaDuyet` (2)
2. Nếu không, mới kiểm tra xem phòng có vướng lịch bận/lịch chính khóa không: `ban != null` => Trả về `(int)TrangThaiBanPhongChucNang.TrungLichChinhKhoa` (3)
3. Tiếp theo là kiểm tra đơn đang chờ duyệt: `(dadk != null && (dadk.IsDuyet == null || dadk.IsDuyet == (int)TRANG_THAI_DUYET_NCKH.ChuaDuyet))` => Trả về `(int)TrangThaiBanPhongChucNang.ChoDuyet` (1)
4. Cuối cùng mới là Trống `(int)TrangThaiBanPhongChucNang.Trong` (0).

## 3. Lỗi Trùng Lặp Tiết Học Giữa Các Ca (API `w-locdsloaitgphongchucnang`)

**Vấn đề:**
API `w-locdsloaitgphongchucnang` hiện tại đang trả về mảng `ds_ca_hoc` bị trùng lặp các tiết học ở giữa các ca (Ví dụ: Ca 3 có tiết 10,11,12,13 nhưng Ca 4 lại tiếp tục có tiết 10,11,12,13,14). Hậu quả là Frontend bị duplicate giao diện cột lưới phòng và React báo lỗi trùng key.

**Nguyên nhân chi tiết:**

- Trong `PhongChucNangController.cs` (khoảng dòng 410), API truy vấn lấy danh sách ca thi từ bảng `LTenGoiCaThi` với điều kiện duy nhất là `NHHK == 0`.
- Do không lọc theo `IDLoaiTG` hoặc cấu hình riêng, kết quả gom nhóm (`group f by f.TenGoiCaThi`) đã gộp tất cả các dòng cấu hình ca thi (có thể thuộc các hệ đào tạo hoặc cơ sở khác nhau) lại làm một.
- Nếu hệ thống có nhiều loại cấu hình cho "Ca 3" (ví dụ: Ca 3 bắt đầu từ tiết 7 kéo dài 4 tiết, và Ca 3 khác bắt đầu từ tiết 10), phép `Union` / `SelectMany` sẽ gộp tất cả các tiết đan xen lại vào cùng một mảng `ds_tiet_theo_ca`.

**Yêu cầu xử lý cho Backend:**

- Nếu nghiệp vụ Đăng ký phòng học sử dụng **Ca Học (Sáng, Chiều, Tối)** theo thời khoá biểu thông thường, Backend không nên lấy từ `LTenGoiCaThi` (Ca thi thường chồng chéo). Thay vào đó, nên sinh `ds_ca_hoc` thông qua cấu hình số tiết Sáng/Chiều/Tối của chính `LDMLoaiTG` (Các trường `STSang`, `STChieu`, `STToi`).
- Nếu bắt buộc phải dùng `LTenGoiCaThi`, Backend phải bổ sung thêm điều kiện lọc phù hợp (như theo `IDLoaiTG`, Hệ đào tạo, Cơ sở, v.v.) để đảm bảo mỗi "Ca" trả về một mảng Tiết duy nhất và không giao nhau.
