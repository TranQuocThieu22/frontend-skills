---
name: rrs-gam-api-integration
description: Hướng dẫn tích hợp API, sử dụng axios, hook lấy dữ liệu React Query và đồng bộ cấu trúc dữ liệu giữa UI và API trong dự án RRS và GAM.
---

# Kỹ năng Tích hợp API cho RRS & GAM

Kỹ năng này giải thích cách xây dựng và duy trì các dịch vụ API, quản lý trạng thái máy chủ (server-state) sử dụng `@tanstack/react-query` và `useCustomReactQuery`, cũng như cách liên kết trực tiếp cấu trúc dữ liệu từ backend vào các model form trên giao diện người dùng (UI) trong các dự án `RRS` và `gam`.

> [!NOTE]
> **Đường dẫn thư mục Backend**: Mã nguồn backend cho dự án RRS và GAM nằm ở thư mục local: `D:\AQ-Project\Source.NET`.
>
> - **QUY TẮC BẮT BUỘC**: Trước khi đọc hoặc phân tích bất kỳ đoạn code backend nào trong `D:\AQ-Project\Source.NET`, bạn **PHẢI** chạy lệnh `git pull` hoặc `git fetch` (dùng tool `run_command`) tại thư mục đó để đảm bảo bạn đang làm việc với mã nguồn mới nhất.
> - Khi tích hợp API mới hoặc nếu có bất kỳ sự mơ hồ nào về cấu trúc request/response, hãy chủ động kiểm tra các controller, DTO và service ở backend để đảm bảo khớp nối chính xác.

## Thư mục Dịch vụ API (`apps/RRS/src/shared/APIs/` và `apps/gam/src/shared/APIs/`)

Tất cả các file dịch vụ API (API services) phải được khai báo thành các file riêng biệt nằm trong thư mục `src/shared/APIs/` của từng ứng dụng tương ứng. Các file này sử dụng `axiosInstance` (import từ `@/shared/configs/axiosInstance`) và có thể dùng các hàm hỗ trợ cơ bản như `createBaseAPI` (từ `@aq-fe/aq-core-framework/shared/libs/createBaseAPI`).

### Cách viết một API Service

> [!WARNING]
> **QUY TẮC BẮT BUỘC: KHÔNG ĐƯỢC THÊM TIỀN TỐ `/api/`**: Cấu hình `axiosInstance` đã tự động thiết lập sẵn base URL có chứa `/api/`. Do đó, khi định nghĩa hằng số `CONTROLLER` trong API service, bạn **KHÔNG ĐƯỢC** thêm `/api/` vào đường dẫn (ví dụ: hãy dùng `const CONTROLLER = "/RoomBooking";` thay vì `const CONTROLLER = "/api/RoomBooking";`). Nếu vi phạm, đường dẫn sẽ bị nhân đôi thành `/api/api/...` gây lỗi.

```typescript
import axiosInstance from "../configs/axiosInstance";
import { CustomAPIResponse } from "@aq-fe/aq-core-framework/shared/interfaces/CustomAPIResponse";
import { createBaseAPI } from "@aq-fe/aq-core-framework/shared/libs/createBaseAPI";

const CONTROLLER = "/myController";

export const myService = {
  // Tự động kế thừa các endpoint CRUD cơ bản (getById, create, update, delete)
  ...createBaseAPI<MyModel>(CONTROLLER, axiosInstance),

  // Tự định nghĩa các endpoint riêng khác
  getCustomData: (tenantId: string, pageNumber: number) => {
    return axiosInstance.get<CustomAPIResponse<MyModel[]>>(
      `${CONTROLLER}/${tenantId}/custom`,
      { params: { PageNumber: pageNumber } },
    );
  },
};
```

## Lấy dữ liệu với `useCustomReactQuery`

Dự án sử dụng hook `useCustomReactQuery` (từ `@aq-fe/aq-core-framework/shared/hooks/useCustomReactQuery`) đóng vai trò là wrapper bọc ngoài `useQuery` tiêu chuẩn của `@tanstack/react-query`.

> [!IMPORTANT]
> **Bắt buộc sử dụng để xử lý lỗi**: Bạn PHẢI sử dụng `useCustomReactQuery` và `useCustomReactMutation` thay vì các hook gốc của `@tanstack/react-query` đối với mọi tương tác gọi API backend. Điều này giúp tự động bắt và xử lý các lỗi (exceptions) tiêu chuẩn trả về từ backend.

### Cấu trúc khai báo Custom Queries

**QUY TẮC BẮT BUỘC**: KHÔNG được tạo riêng thư mục `hooks/` hoặc các file `use*Queries.ts` riêng biệt cho từng tính năng. Thay vào đó, component nào cần gọi API lấy dữ liệu thì phải khai báo trực tiếp `useCustomReactQuery` ngay bên trong file của component đó.

### Lưu ý về Ánh xạ dữ liệu (Axios + React Query)

> [!WARNING]
> **CẨN THẬN VỚI VIỆC LỒNG `.data`**: Nếu API service của bạn trả về `axiosInstance.post(...).then(res => res.data)`, thì payload (kết quả) truyền vào React Query ĐÃ LÀ nội dung chính (body) của response (ví dụ như `CustomAPIResponse`).
> Do đó khi bạn bóc tách dữ liệu từ `useQuery`:
> `const { data: myDataRes } = useQuery(...)`
> Biến `myDataRes` ở đây CHÍNH LÀ thân response (API response body). TUYỆT ĐỐI KHÔNG vô tình chấm tiếp `.data.data` (ví dụ `myDataRes.data.data`) trừ khi backend thực sự bọc dữ liệu của họ qua 2 lớp (double wrap). Lỗi này rất phổ biến và sẽ dẫn đến việc biến bị `undefined`.

## Mô hình Đồng nhất dữ liệu UI và API

Để tối đa hoá tính dễ bảo trì và giảm thiểu độ phức tạp của code, **tuyệt đối không xây dựng các cơ chế ánh xạ (translation mapping) chuyển đổi qua lại** giữa cấu trúc giao diện UI (form states, table columns) và các model API từ Backend.

### Các Quy tắc Cốt lõi:

1. **Ràng buộc Interface Trực tiếp**: Đảm bảo các giá trị `useForm` của Mantine, các interface TypeScript, và các thuộc tính component sử dụng trực tiếp mô hình dữ liệu request/entity chính xác của API Backend.
   - **ĐẶC BIỆT QUAN TRỌNG**: Khi định nghĩa cột bảng (Table Columns) bằng `accessorKey` hoặc các trường dữ liệu trên UI, **PHẢI sử dụng chính xác tên field của backend**. Tuyệt đối **không tạo ra các props riêng trên UI** rồi map lại thủ công từ field của backend.
2. **Xử lý các giá trị mặc định động của Backend**: Chỉ tính toán và bổ sung các trường dữ liệu phái sinh cần thiết bên trong payload lúc `onSubmit` (submit form) mà không làm thay đổi cấu trúc gốc của model form.
3. **Giữ nguyên các Giao diện Nguyên mẫu (CRITICAL & MANDATORY)**: Nếu API backend bị thiếu một vài trường hoặc tính năng đang hiện diện trên bản thiết kế giao diện (UI prototype) gốc, **KHÔNG ĐƯỢC xoá các trường UI đó đi hoặc disable (vô hiệu hoá) chúng**.
   - Hãy giữ cho các trường UI đó hoạt động đầy đủ ở client (ví dụ: bằng cách kế thừa và mở rộng interface TypeScript của backend ở local).
   - Hãy bỏ qua (omit) các trường bị thiếu khi gửi request payload lúc submit, hoặc cứ gửi lên để backend tự lờ đi.
   - **Báo cáo bắt buộc (API Status Tracker)**: Mỗi khi bạn phát triển/tích hợp một API, bạn **PHẢI** tạo ra một file báo cáo `API_STATUS_PENDING.md` hoặc `API_STATUS_DONE.md` nằm ngay trong thư mục feature tương ứng để theo dõi tiến độ và ghi nhận lại sự sai lệch giữa UI/API (gap).
   - **Hiển thị trực quan cho các trường bị thiếu (Icon + Tooltip)**: Khi tích hợp API, nếu giao diện (UI) có trường dữ liệu (field) nhưng backend không có field đó, bạn **bắt buộc** phải hiển thị rõ icon cảnh báo (ví dụ icon warning dấu chấm than) kèm theo tooltip giải thích trên giao diện, đồng thời **phải note rõ** vấn đề này vào file báo cáo tích hợp (API_STATUS).

## Lưu ý Tích hợp chung cho RRS & GAM

- Tương tự như các ứng dụng khác, luôn phải triển khai tìm kiếm (search), phân trang (pagination) và bộ lọc (filtering) ở phía server (server-side) bất cứ khi nào có thể.
- Tất cả các tham số lọc ngày tháng (date filtering payload) phải được định dạng chính xác theo chuẩn UTC.
- Với các trang nguyên mẫu (Mock prototype), một khi đã được tích hợp API đầy đủ, BẮT BUỘC phải gỡ bỏ thuộc tính `isPrototype: true` khỏi cấu hình menu (trong `layout.tsx` hoặc `routes.config.tsx`).

## Gỡ lỗi Backend & Quy tắc Báo cáo (RRS)

Khi thực hiện tích hợp API trong ứng dụng `rrs` (Room Reservation System) và gặp hiện tượng API trả về thành công nhưng dữ liệu không thay đổi, hãy thực hiện các bước kiểm tra sau để xác định lỗi Backend:

1. **Kiểm tra Payload gửi đi:** 
   - Sử dụng tab Network trên trình duyệt (hoặc log) để đảm bảo cấu trúc Payload (tên biến dạng `snake_case` hoặc `camelCase`) và giá trị gửi lên API là chính xác tuyệt đối so với tài liệu mô tả hoặc cấu trúc DB.

2. **Dấu hiệu nhận biết lỗi Backend:**
   - Nếu Frontend gửi đúng Payload, API trả về Code 200 (Thành công), nhưng gọi API `GET` ngay sau đó lại vẫn trả về dữ liệu cũ => Lỗi 100% thuộc về Backend.

3. **Các lỗi Backend thường gặp trong dự án (Đặc tả lỗi):**
   - **Quên Commit Transaction:** Backend sử dụng `db.Database.BeginTransaction()` để cập nhật dữ liệu (`db.BulkEntityUpdateOne`) nhưng lập trình viên quên gọi `trans.Commit()`. Hậu quả là dữ liệu bị rollback tự động khi kết thúc block `using`.
   - **Lỗi Cache:** Dữ liệu GET bị lưu Cache (Redis/Memory) nhưng không được Invalidate/Clear sau khi thực hiện thao tác PUT/POST/DELETE.
   - **Gắn sai Map Field:** Cấu trúc biến của Frontend không được ánh xạ đúng vào Model ở Controller (đôi khi do khác biệt `snake_case` và `PascalCase` trong C#).

### Cấu trúc mã nguồn Backend RRS
- Toàn bộ API nghiệp vụ của hệ thống RRS (Room Reservation System / Đăng ký phòng chức năng) được đặt tại: `Source.NET/trunk2/Edusoft.NET/Edusoft.Api/Controllers/Customer/`
- Controller chính xử lý đăng ký phòng, quản lý phòng, bộ lọc tìm kiếm: `PhongChucNangController.cs`

**Quy tắc:**
- Không tự ý sửa mã nguồn Backend của dự án C#. Chỉ cần phân tích, khoanh vùng chính xác vị trí lỗi (ví dụ file Controller tương ứng) và đưa ra đặc tả (specification) để báo cho team Backend tự khắc phục.

### Quy tắc báo cáo khi tích hợp (Integration Reporting Rules)
- **BẮT BUỘC:** Mỗi khi người dùng yêu cầu "tích hợp API" (integrate API) cho một Form, Bảng, hoặc một tính năng bất kỳ trên một trang (page), sau khi hoàn thành tích hợp, bạn phải quét lại **toàn bộ các thành phần trên trang đó** (không chỉ giới hạn ở component vừa làm).
- Nếu phát hiện ra **bất kỳ dữ liệu nào trên trang vẫn còn đang bị hard code (dữ liệu giả/mock)** (ví dụ: các tuỳ chọn của Select, filter, text hiển thị tĩnh, dữ liệu bảng chưa được map với API do thiếu trường trả về), bạn **PHẢI** liệt kê danh sách chi tiết các chỗ hard code đó và báo cáo lại ngay cho người dùng trong câu trả lời kết luận của bạn.
- Bạn cần nêu rõ nguyên nhân (ví dụ: "Do API hiện tại chưa hỗ trợ trường này", hoặc "Thiếu API chuyên biệt cho tính năng này", hoặc "Nghi ngờ lỗi cấu trúc Backend"). Điều này giúp người dùng dễ dàng theo dõi và tạo task cho đội Backend xử lý bổ sung.
- **Xoá nhãn Prototype:** Khi đã hoàn tất tích hợp API cho một trang (tức là trang đó không còn dùng Mock Data cho các luồng chính nữa), bạn BẮT BUỘC phải vào file cấu hình menu (thường là `routes.config.tsx`) và xoá bỏ thuộc tính `isPrototype: true` của route tương ứng, nhằm gỡ bỏ icon chữ P (Bản nháp) trên thanh sidebar.
