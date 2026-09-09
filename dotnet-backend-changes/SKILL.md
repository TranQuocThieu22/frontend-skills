---
name: dotnet-backend-changes
description: Quy trình sửa backend .NET (Source.NET) khi người dùng yêu cầu - tạo nhánh từ mã trường, sửa, commit để người dùng merge vào internal.
---

# Sửa Backend .NET (Source.NET)

Được phép sửa backend **khi người dùng yêu cầu rõ ràng**. Mặc định vẫn chỉ đọc để đối chiếu API.

## Vị trí

| | |
| --- | --- |
| Thư mục | `D:\AQ-Project\Source.NET\trunk2` |
| Loại | Git repo riêng, **không** phải repo frontend |
| Remote | `https://portal.aqtech.vn:1443/tfs/train/Source.NET/_git/Source.NET` |
| Nhánh tích hợp | `internal` — môi trường test, người dùng tự merge để build |
| Nhánh theo trường | `VersionSchool/<MÃ TRƯỜNG>/origin` (vd `VersionSchool/MOHN/origin`) |
| Nhánh schema | `DB/Database` |

Controller chính của phân hệ đơn từ: `Edusoft.NET/Edusoft.Api/Controllers/SMSController.cs`.
DTO: `Edusoft.NET/Edusoft.DataObjects/WEB/*.cs`. Entity: `AQDataTools/AQData.Entity/Entities/`.
Hằng tên cột: `Edusoft.NET/Edusoft.DataObjects/Common/AllFields.cs` (sinh từ schema — tra file này
để biết một cột **đã có thật trong CSDL** hay chưa).

## Quy trình

1. **Hỏi mã trường** nếu người dùng chưa cho (MOHN, DHNT, DHNL...). Mã này quyết định nhánh gốc.
2. **Cắt nhánh từ nhánh của trường đó**, không cắt từ `internal`:
   ```bash
   git fetch origin --quiet
   git switch -c "VersionSchool/<MÃ>/<caseId>-<mo-ta-khong-dau>" --no-track origin/VersionSchool/<MÃ>/origin
   ```
   Đặt tên theo đúng kiểu sẵn có trong repo: `VersionSchool/MOHN/63077-sua-loi-hien-thi-du-lieu-tu-file`.
3. **Sửa code.**
4. **Chỉ stage đúng file mình sửa.** Repo này thường có thay đổi dở dang của người khác
   (`git status` hay thấy file lạ) — tuyệt đối không `git add -A`.
5. **Commit** theo kiểu của repo backend, **không** dùng format `<type>(<scope>)` của frontend:
   ```
   <caseId>: mô tả ngắn bằng tiếng Việt

   Chi tiết từng thay đổi, nêu rõ lỗi cũ sai ở đâu.

   Ảnh hưởng: các API bị đụng tới, có giới hạn theo trường nào không.
   ```
6. **Không push** trừ khi người dùng bảo. Người dùng dùng nhánh này để merge vào `internal`.

## Không build kiểm chứng được — phải nói ra

Máy frontend không dựng đủ dependency của backend. Build `Edusoft.Api.csproj` bằng MSBuild sẽ ra
hàng nghìn lỗi `CS0246` ngay từ các dòng `using` (thiếu `AQData`, `AQFramework`) — **đó là lỗi môi
trường, không phải lỗi code**. Đừng báo là code hỏng, và cũng đừng nói code đã build được.

Việc kiểm được:
- Đọc lại kỹ từng vùng sửa.
- Đếm cân bằng ngoặc của file: `awk '{o+=gsub(/{/,"{"); c+=gsub(/}/,"}")} END{print o-c}' <file>`.

Luôn báo người dùng: **cần build lại bằng môi trường .NET đầy đủ trước khi merge vào `internal`.**

## Lưu ý riêng của codebase này

- Nhiều tính năng bị chặn theo trường bằng `DVSettingDA.IsMOHN`, `IsDHSG`... Sửa trong nhánh nào thì
  giữ nguyên điều kiện đó, đừng vô tình bật cho mọi trường.
- Ghi dữ liệu qua AQFramework: `db.BulkInsert<T>(arr, ViewList.X, now)`,
  `db.BulkEntityUpdate<T>(arr, ViewList.X, now, AllFields.COT1, ...)`,
  `db.BulkForceDelete<T>(arr, ViewList.X, now)`. `BulkEntityUpdate` **chỉ cập nhật các cột được liệt
  kê**, thiếu cột nào là cột đó không được ghi.
- Cột mới chỉ dùng được khi đã có trong `AllFields.cs`. Chưa có nghĩa là schema chưa có cột đó —
  phải làm script bên `DB/Database` trước, không tự thêm hằng vào `AllFields.cs`.
- `BigID` là alias của `System.Int64`.
