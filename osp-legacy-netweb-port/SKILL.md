---
name: osp-legacy-netweb-port
description: Quy trình port một màn hình từ dự án Angular netweb cũ sang app OSP (Next.js + Mantine) - tìm code nguồn, dùng đúng API legacy, dựng bảng/form theo core-ui và quy ước đặt bộ lọc.
---

# Port màn hình từ netweb (Angular) sang OSP (Next.js)

Dùng khi có yêu cầu dạng "kiểm tra code màn hình X bên netweb cũ và tích hợp cho OSP".

## 1. Tìm code nguồn bên netweb

Mã nguồn Angular cũ: `D:\AQ-Project\Source.NET\netweb`.

- Màn hình nghiệp vụ đơn từ một cửa: `projects/web-main/src/app/main/chuc-nang/quan-ly-don-tu-mot-cua/<ten-man-hinh>/`
  - `<ten>.component.ts` / `.html`: màn danh sách (load, search, sort, export, print).
  - `edit-<ten>/edit-<ten>.component.ts` / `.html`: modal thêm/sửa/xem.
- Tên endpoint tra ở `projects/libs/src/lib/libs.service.ts` (class `ApiList`), ví dụ
  `public static get locdanhmucloaidichvu() { return ApiList.apiPrefix + 'sms/w-locdanhmucloaidichvu'; }`.
- Các hàm `this.dataServices.<ten>()` tra ở `projects/web-main/src/app/services/dataservices.service.ts`.

> [!WARNING]
> Đọc kỹ trước khi bê nguyên: code cũ có phần **chết hoặc sai** (ví dụ modal Import của Danh mục loại dịch vụ không có nút nào gọi tới, và map sang field của *nhóm* dịch vụ chứ không phải loại dịch vụ). Gặp phần như vậy thì bỏ qua và báo lại cho người dùng, đừng port lỗi sang app mới.

## 2. Chọn đúng endpoint

Một nghiệp vụ thường có 2 nhóm API khác nhau — **rất dễ nhầm**:

- API cho **màn hình quản trị/danh mục** (trả kèm toàn bộ danh mục phụ): ví dụ `sms/w-locdanhmucloaidichvu` trả về `ds_dichvu`, `ds_nhom`, `ds_bo_dem`, `ds_mau_in`, `ds_dt_hien_thi` trong **một lần gọi**.
- API cho **màn hình người dùng cuối** (sinh viên đăng ký): ví dụ `sms/w-locdsdanhmucgcn` chỉ trả `ds_GiayCN`.

Nếu dùng nhầm API người dùng cuối cho màn quản trị, các cột nhóm/đối tượng sẽ trống và các select trong form không có dữ liệu.

Quy ước gọi API trong OSP (`src/features/<feature>/api/<ten>.api.ts`):

```ts
const getApiBase = () => (APP_CONFIG.isDevMode ? "/api" : APP_CONFIG.apiUrl);

axiosInstance.post(`${getApiBase()}/sms/w-locdanhmucloaidichvu`, '', { baseURL: '' });
```

- `baseURL: ''` là bắt buộc để interceptor không nối thêm base lần nữa.
- Giữ **nguyên body** như netweb gửi (nhiều endpoint legacy nhận body rỗng `''`, hoặc query string như `?idgcn=...`).
- Dev mode đi qua proxy khai báo ở `apps/osp/src/middleware.ts` (matcher `/api/:path*`), không có rewrite trong `next.config.ts`.
- API legacy trả lỗi **trong payload 200**: luôn kiểm tra `res.data.code === 200` chứ không chỉ HTTP status.
- File đã lưu trên server xem qua `${apiBase}/fileManager?src=<url_file>`; file mới upload gửi base64 (bỏ tiền tố `data:...;base64,`).

## 3. Map dữ liệu ở client

API legacy chỉ trả id, phần hiển thị phải tự ghép — bê nguyên logic map của component cũ:

- id nhóm → mã/tên nhóm (chú ý API trả **cả 2 kiểu hoa/thường**: `id_nhom_GCN` và `id_nhom_gcn`, phải `??` cả hai).
- `quyen_xem` → `mieu_ta` trong `ds_dt_hien_thi` (lấy từ server, **không hard-code**).
- `loai_gcn` → `0: Hành chính`, `2: Tài chính` (hằng số cố định ở client).
- Cờ boolean từ backend có thể là `true/1/'1'` → dùng helper `isTruthyFlag`.
- Ngày: API trả `dd/MM/yyyy`, Mantine DateInput dùng `yyyy-MM-dd`, body lưu gửi `yyyy-MM-dd`.

## 4. Cấu trúc feature trong OSP

```
src/features/<feature>/
  api/<ten>.api.ts        # axios + useQuery/useMutation, invalidate sau khi lưu/xóa
  components/<Ten>Table.tsx
  modal/<Ten>CreateUpdate.tsx
  shared/types.ts         # giữ NGUYÊN tên field của API legacy
  shared/utils.ts         # normalize, format ngày, export excel, print
```

Component dùng lại từ `@aq-fe/core-ui/shared/components`: `CustomTanstackTable`, `CustomModalTrigger`, `CustomButton`, `CustomActionIcon`, `CustomTextInput`, `CustomSelect`, `CustomNumberInput`, `CustomDateInput`, `CustomFileInput`. Xác nhận xóa dùng `modals.openConfirmModal`, thông báo dùng `notifications.show`.

## 5. Quy ước bảng (CustomTanstackTable)

> [!IMPORTANT]
> **Bộ lọc phải nằm CÙNG HÀNG với ô tìm kiếm** — truyền qua `renderTopToolbarLeftActions`, dùng `placeholder` thay cho `label` để không lệch chiều cao với ô tìm kiếm.
> KHÔNG đặt bộ lọc vào `renderTopContent`: slot đó render **phía trên** toolbar, tách rời khỏi ô tìm kiếm và trông rời rạc.

> [!WARNING]
> **Search "controlled" thì bảng KHÔNG tự lọc.** Trong `CustomTanstackTable`, nếu truyền `searchValue` + `onSearchChange` thì `isControlledSearch = true` và bộ lọc theo `searchKeys` bị bỏ qua hoàn toàn (`if (search.trim() && searchKeys && !isControlledSearch)`), tức là gõ vào ô tìm kiếm sẽ **không có tác dụng gì** nếu parent không tự lọc.
> - Có bộ lọc riêng (select) → giữ controlled và tự lọc trong `useMemo` của parent; lúc đó `searchKeys` là thừa, nên bỏ.
> - Không có bộ lọc gì thêm → để uncontrolled (chỉ truyền `enableSearch` + `searchKeys`), bảng tự lo.
> - Tìm kiếm nên bỏ dấu (`normalizeText`) như bản netweb cũ, và nhớ reset `pageIndex` về 0 khi đổi từ khóa/bộ lọc.

- `renderTopToolbarCustomActions`: các nút In / Xuất Excel / Thêm mới (bên phải).
- `renderRowActions`: Xem → Sửa → Xóa, đúng thứ tự bản cũ.
- Nút In và Xuất Excel phải chạy thật trên **dữ liệu đang lọc**, không để làm cảnh:
  - Excel: `xlsx` (`utils.json_to_sheet` + `writeFile`), đúng bộ cột của bản netweb cũ.
  - In: tự dựng HTML từ dữ liệu rồi `window.open` + `print` (A4 landscape), không clone DOM như Angular.

## 6. Modal thêm/sửa/xem

Bản cũ dùng `loai_thaotac`: `1 = thêm`, `2 = xem`, `3 = sửa`. Trong OSP đổi thành prop `mode: 'create' | 'update' | 'view'`:

- `view`: khóa toàn bộ input, ẩn nút Lưu.
- `update`: khóa trường mã (`ma_gcn`), các trường khác mở.
- Master data (bộ đếm, mẫu in, nhóm xử lý, đối tượng hiển thị) truyền từ bảng xuống, lấy từ cùng một lần gọi API danh sách.

## 7. Sau khi port xong

- Gỡ mock data cũ của màn hình (`shared/mockData.ts`) khi đã nối API thật.
- Gỡ cờ `isPrototype: true` của route tương ứng trong `src/shared/configs/routes.config.tsx` (cờ này chỉ hiện badge "P" ở sidebar).
- Chạy `npx tsc --noEmit -p tsconfig.json` trong `apps/osp` và lọc lỗi theo tên feature — repo còn nhiều lỗi type cũ ở feature khác, đừng nhận nhầm.
