---
name: srb-backend-payload
description: Thông tin về các Payload API Backend của dự án SRB (Edusoft.Api)
---

# Payload Backend SRB (Edusoft.Api)

Dự án SRB sử dụng chung Backend với `Edusoft.Api`. 

## 1. Lịch sử đăng ký phòng (`w-loadlichsudangkysudungphong`)

Endpoint: `POST /srb/w-loadlichsudangkysudungphong` (Controller: `PhongChucNangController.cs`)
Class model (Backend): `LichSuDangKyRequest` (File: `Edusoft.DataObjects\WEB\PhongChucNangResult.cs`)

**Các trường có thể truyền vào:**
- `tu_khoa` (string): Lọc theo mã đơn, mục đích sử dụng, hoặc loại phòng.
- `loai_don` (string): Được định nghĩa trong model nhưng **hiện tại Backend chưa code xử lý lọc theo trường này**.
- `trang_thai` (int?): `null` (Tất cả), `-1` (Chờ duyệt), `0` (Không duyệt), `1` (Được duyệt), `2` (Đã hủy), `3` (Quá hạn), `4` (Sắp quá hạn).
- `tu_ngay` (string): Lọc từ ngày (Định dạng: `dd/MM/yyyy`).
- `den_ngay` (string): Lọc đến ngày (Định dạng: `dd/MM/yyyy`).
- `additional` (AdditionalDO): Chứa thông tin phân trang `paging` (`page`, `limit`).

**Lưu ý quan trọng:**
- Mặc dù giao diện (Frontend) có bộ lọc "Loại phòng" và "Mục đích sử dụng" bằng ID (ví dụ: `-8826528838853969552`), Backend hiện tại **không có** trường `id_loai_phong` hay `id_ly_do_sd_ph` trong class `LichSuDangKyRequest`.
- Thay vào đó, Backend chỉ định nghĩa trường `loai_don`, nhưng trường này cũng đang bị bỏ qua trong logic LINQ của API `W_LoadLichSuDangKySuDungPhong`.
- Do đó, để bộ lọc hoạt động chính xác ở Backend, cần phải yêu cầu team Backend (hoặc tự sửa source C#) bổ sung thêm các trường `id_loai_phong` và `id_ly_do_sd_ph` vào `LichSuDangKyRequest`, đồng thời thêm điều kiện `Where` vào `query`.

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
