# BƯỚC 5: ĐẶC TẢ THIẾT KẾ DATABASE (DATABASE DESIGN SPECIFICATION)
> **Dự án:** Project TimeKeeper (Module Quản lý Checklist Công việc tích hợp Chấm công)  
> **File thiết kế gốc (Draw.io):** [`md/db/db-option-2.drawio`](../db/db-option-2.drawio)  
> **Đầu vào:** [`md/analysis/4-fdd-and-rbac.md`](./4-fdd-and-rbac.md)  
> **Đầu ra phục vụ:** Bước 6 (Thiết kế & Viết Tài liệu RESTful API)

---

## 1. Tư duy Thiết kế & Lý do Sử dụng 1 Collection Duy nhất (`daily_checklist`)

1. **Ranh giới rõ ràng với chức năng Quản lý công việc sẵn có:**
   - Hệ thống SC24h đã có chức năng Quản lý công việc dành cho các dự án/task dài hạn.
   - Module **TimeKeeper Checklist** tập trung giải quyết bài toán **ghi chú công việc hàng ngày gắn với chấm công Check-in / Check-out**: nắm bắt nhân viên hôm nay làm gì và cuối ngày đạt tiến độ bao nhiêu `%`.
2. **Cơ chế kế thừa việc chưa xong mà không cần tách thêm bảng:**
   - Mỗi nhân viên có 1 bản ghi `daily_checklist` cho ngày làm việc (`date`), bên trong nhúng mảng `check_list[]` lưu các đầu mục công việc của ngày hôm đó.
   - Khi nhân viên Check-in ngày mới, hệ thống có thể truy vấn các công việc chưa hoàn thành (`status` chưa `DONE` / `progress < 100`) từ các ngày trước đó trong chính bảng `daily_checklist` để gợi ý nhân viên chọn tiếp tục làm (copy tạo thành mục mới cho ngày hôm nay) hoặc thêm công việc mới.

---

## 2. Bảng Đặc tả 4 Cột Chuẩn hóa

### 2.1. Collection: `daily_checklist`
Lưu trữ thông tin phiên Checklist công việc trong ngày của từng nhân viên.

| field | type | required | des |
|---|---|---|---|
| `_id` | `ObjectID` | `true` | id MongoDB tự sinh |
| `employee_id` | `ObjectID, Index` | `true` | id nhân viên trong bảng `Employee` (SC24h) |
| `branch_id` | `ObjectID, Index` | `true` | id phòng ban / chi nhánh trong bảng `Branch` (SC24h) |
| `check_list` | `Arrays` | `default = []` | Các đầu mục công việc trong ngày `[tasks]`. Snapshot task khi checkout |
| `date` | `Date, Index` | `default = Date.now()` | Ngày làm việc |

---

### 2.2. Cấu trúc Phần tử bên trong Mảng `check_list[]` (Embedded Sub-document)
Mỗi phần tử nằm trong mảng `check_list` của bảng `daily_checklist` có cấu trúc 4 cột như sau:

| field | type | required | des |
|---|---|---|---|
| `_id` | `ObjectID` | `true` | `_id` MongoDB tự sinh cho từng đầu mục công việc |
| `description` | `String` | `true` | Mô tả công việc |
| `progress` | `Number` | `default = 0, max = 100` | Tiến độ công việc khi checkout (`0% - 100%`) |
| `status` | `String` | `[TODO, PROGRESS, DONE, HOLD, CANCELLED]` | Trạng thái công việc sau khi checkout |
