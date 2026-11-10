# Báo cáo công việc hàng ngày (Daily Checklist)

Tài liệu mô tả cách chức năng hoạt động và bộ REST API `/api/v1/daily-checklists`.
Đường dẫn Web group.suachualaptop24h.com : `/cham-cong-web` đang dùng chung bộ API này.
Kèm theo file Postman: `Báo cáo công việc hàng ngày — -api-v1-daily-checklists.postman_collection.json`.

---

## 1. Tổng quan

Mỗi nhân viên có **1 checklist cho mỗi ngày làm việc**. Một checklist gồm nhiều đầu mục công việc.

| Thời điểm | Việc người dùng làm |
|---|---|
| Sau khi **chấm công vào** | Lập checklist hôm nay: thêm việc mới và/hoặc **tiếp tục việc cũ** chưa xong của checklist gần nhất |
| Trong ngày | Xem, thêm, sửa, xóa việc trong checklist **hôm nay** |
| Sau khi **chấm công ra** | Chốt tiến độ (%) và trạng thái của từng việc trong ngày |
| Bất kỳ lúc nào | Quản lý (có quyền Xem báo cáo) xem báo cáo của tất cả mọi người |

Ba điểm cần nắm:
-  Muốn làm tiếp việc chưa xong hôm trước thì tạo một mục mới trong checklist hôm nay. Mục mới giữ nội dung và tiến độ trước đó ( Ko liên kết mục cũ)
- Server tự tính ngày theo **giờ Việt Nam**. Client không gửi ngày khi thêm hoặc sửa việc.


### Các file liên quan

Chức năng là một module chạy trong gateway: `Route → Controller → Service → MongoDB`.

| File | Vai trò |
|---|---|
| `services/daily_task_progress/daily_task_progress_router.js` | Khai báo route REST và kiểm tra quyền |
| `services/daily_task_progress/daily_task_progress_controller.js` | Nhận request, trả JSON |
| `services/daily_task_progress/daily_task_progress_service.js` | Nghiệp vụ và aggregate báo cáo |
| `services/daily_task_progress/daily_task_progress_model.js` | Model `DailyChecklist` |
| `services/daily_task_progress/daily_task_progress_rules.js` | Các hàm thuần: trạng thái, ngày giờ VN, gợi ý việc cũ… |
| `services/daily_task_progress/daily_task_progress_permission.js` | Mã quyền và cách kiểm tra quyền |
| `services/daily_task_progress/tests/` | Unit test + test tích hợp (gọi HTTP thật trên DB tạm) |

---

## 2. Phân quyền

Có 3 quyền, nằm trong collection `functions` của DB `suachualaptop24h` và được cấp qua màn hình phân quyền của ver3:

| `_id` | Tên | Quyền cha | Cờ trả về | Mở được gì |
|---|---|---|---|---|
| `69710af7090eb0bbb82a89b0` | Công việc hàng ngày | không có (`null`) | không có | Chỉ để gom nhóm trên cây quyền. Quyền cha
| `69710af7090eb0bbb82a89b1` | Báo cáo | `…89b0` | `bao_cao` | Lập và cập nhật checklist **của mình**: mọi API checklist |
| `69710af7090eb0bbb82a89b2` | Xem báo cáo | `…89b0` | `xem_bao_cao` | Xem báo cáo của **tất cả mọi người**: `/reports`, `/reports/filters` |

**Cách tính quyền** giống `check_permission_by_employee` của ver3. Một người có quyền khi thỏa **một trong hai** điều kiện:
1. Nhóm người dùng (`Employee.ID_GroupUser`) của họ có dòng trong `permissions` với `ID_Function` là mã quyền (không xét `Status`).
2. Bản thân họ có dòng trong `permissionemployees` với `id_function` là mã quyền và `status: true`.

Server đọc lại quyền ở **mỗi request**, nên vừa tick cấp quyền trên ver3 là có hiệu lực ngay.

---

## 3. Dữ liệu: collection `DailyChecklist` (DB `suachualaptop24h`)

| Trường | Kiểu | Ý nghĩa |
|---|---|---|
| `_id` | ObjectId | |
| `employee_id` | ObjectId | Mã nhân viên (`Employee._id`) |
| `branch_id` | ObjectId \| null | Chi nhánh của nhân viên lúc tạo checklist |
| `date` | String `YYYY-MM-DD` | Ngày làm việc (giờ VN) |
| `tasks[]._id` | ObjectId | Mã của đầu mục công việc (`task_id`) |
| `tasks[].content` | String (≤ 2000 ký tự, nhiều dòng) | Nội dung công việc |
| `tasks[].progress` | Number 0–100 | Tiến độ % |
| `tasks[].status` | String `[TODO, PROGRESS, DONE, HOLD, CANCELLED]` | Trạng thái công việc sau khi checkout, xem bảng dưới |

- Index **unique** `(employee_id, date)`: mỗi người chỉ có 1 checklist mỗi ngày. Thêm index `(date, branch_id)` cho báo cáo.

| `status` | Nhãn (`status_label`) | Ghi chú |
|---|---|---|
| `TODO` | Chưa làm | Mặc định khi thêm việc mới |
| `PROGRESS` | Đang làm | Việc chép từ ngày cũ luôn ở trạng thái này |
| `DONE` | Xong | Chọn `DONE` thì `progress` tự thành 100 |
| `HOLD` | Tạm hoãn | |
| `CANCELLED` | Đã hủy | **Chỉ** có được qua `DELETE /tasks/{id}`, không gửi trực tiếp được |

- Giá trị **viết hoa, phân biệt hoa/thường**: gửi `done` hoặc `Done` sẽ bị 400 `Trạng thái không hợp lệ`.

Output mẫu:
```json
{
  "_id": "6af27f2f826e2f332832cdd6",
  "employee_id": "6af27f2ff2784b809958c3ec",
  "branch_id": "6af27f2ff2784b809958c3ea",
  "date": "2026-11-10",
  "tasks": [
    { "_id": "6af27f2ff2784b809958c3fb", "content": "Sửa main Dell Latitude 5490\nThay IC nguồn, test 2 tiếng", "progress": 100, "status": "DONE" },
    { "_id": "6af27f2ff2784b809958c3fc", "content": "Gọi lại khách hẹn lấy máy", "progress": 60, "status": "HOLD" },
    { "_id": "6af27f2ff2784b809958c401", "content": "Vệ sinh máy cho khách lẻ", "progress": 0, "status": "CANCELLED" }
  ]
}
```

---

## 4. Quy tắc nghiệp vụ

1. **Ngày theo giờ VN** (`Asia/Ho_Chi_Minh`), server tự tính.
2. **Mỗi người 1 checklist mỗi ngày.** Lần thêm việc đầu tiên trong ngày sẽ tạo checklist, các lần sau thêm vào cùng checklist đó. Hai request gửi cùng lúc cũng không tạo ra 2 checklist.
3. **Chỉ sửa và xóa được việc trong checklist hôm nay**, và chỉ việc của chính mình. Checklist ngày cũ chỉ xem được: sửa sẽ trả **403**.
4. **Xóa mềm (hủy).** Công việc chuyển sang trạng thái `CANCELLED`, vẫn nằm trong checklist và vẫn hiện trong báo cáo. Việc đã hủy không sửa được nữa.
5. **Chọn "Xong" thì tiến độ tự lên 100%**, kể cả khi client gửi `progress` khác.
6. **Tiếp tục việc cũ:**
   - Lấy các việc `TODO`, `PROGRESS`, `HOLD` của **checklist gần nhất trước hôm nay** (không nhất thiết là hôm qua).
   - Bỏ các việc trùng nội dung với việc đã có hôm nay. Khi so sánh, không phân biệt hoa/thường và bỏ qua khoảng trắng thừa.
   - Chép sang thì tạo **mục mới** (`_id` mới), giữ `content` và `progress`, trạng thái thành `PROGRESS`.
7. **Giới hạn:** tối đa **30 việc mỗi ngày**; nội dung tối đa **2000 ký tự**; báo cáo xem tối đa **93 ngày** mỗi lần, mỗi trang tối đa 100 dòng.
8. **Nội dung là chữ thuần.** Thẻ HTML và ký tự `< >` bị bỏ khi lưu. Ký tự xuống dòng (`\n`) được giữ.
9. **Câp nhật cuối ngày (`PATCH /tasks`) kiểm tra toàn bộ trước khi ghi:**.

---

## 5. API

### 5.1 Quy ước chung

- **Đường dẫn gốc:** `{base_url}/api/v1/daily-checklists`. Ở local là `http://localhost:3000`.
- **Đăng nhập:** dùng chung cơ chế của gateway, gửi header `token`. 
- **Body:** gửi JSON, header `Content-Type: application/json`.
- **Dạng trả về:**
  - Thành công: `{ "success": true, "data": … }`
  - Lỗi: `{ "success": false, "message": "<câu báo lỗi tiếng Việt, hiện được cho người dùng>" }`

| HTTP | Khi nào |
|---|---|
| 200 | Thành công |
| 201 | Thêm việc thành công (`POST /tasks`) |
| 400 | Dữ liệu gửi lên sai: thiếu nội dung, tiến độ ngoài 0–100, trạng thái sai, mã sai định dạng, việc đã hủy… |
| 401 | Chưa đăng nhập hoặc hết phiên |
| 403 | Không có quyền (`Bạn không có quyền sử dụng chức năng này`), hoặc sửa / xóa việc của ngày cũ |
| 404 | Không tìm thấy việc (hoặc việc của người khác), hoặc URL không tồn tại (`Không tìm thấy API`) |
| 500 | Lỗi server (`Có lỗi xảy ra, vui lòng thử lại`) |

**Object `task`** (dùng chung cho mọi API trả về việc):
```json
{ "_id": "6af27f2ff2784b809958c3fb", "content": "Sửa main Dell Latitude 5490\nThay IC nguồn", "progress": 40, "status": "PROGRESS", "status_label": "Đang làm" }
```
App dùng `status` để xử lý logic, dùng `status_label` để hiển thị.

### 5.2 Danh sách endpoint

| # | Method + URL | Quyền | Dùng khi |
|---|---|---|---|
| 1 | `GET /?date=&status=` | bao_cao | Xem checklist của mình theo ngày (mặc định hôm nay) |
| 2 | `GET /carry-over-suggestions` | bao_cao | Lấy danh sách việc cũ để tiếp tục |
| 3 | `POST /tasks` | bao_cao | Thêm việc mới và/hoặc chép việc cũ vào hôm nay (lúc chấm công vào, hoặc trong tab Danh sách) |
| 4 | `PATCH /tasks/{taskId}` | bao_cao | Sửa 1 việc của hôm nay |
| 5 | `DELETE /tasks/{taskId}` | bao_cao | Xóa (hủy) 1 việc của hôm nay |
| 6 | `PATCH /tasks` | bao_cao | Chốt nhiều việc 1 lần (lúc chấm công ra) |

### 5.3 Chi tiết

#### 1. `GET /?date=YYYY-MM-DD&status=`: Lấy checklist của tôi theo ngày ( trả về danh sách các công việc trong danh sách của ngày hôm đó lọc theo ngày và theo trạng thái)

| Query | Bắt buộc | Mô tả |
|---|---|---|
| `date` | không | Ngày cần xem, mặc định hôm nay. Sai định dạng → 400 `Ngày không hợp lệ (định dạng YYYY-MM-DD)` |
| `status` | không | Lọc theo **1 hoặc nhiều** trạng thái, cách nhau dấu phẩy, không cần khoảng trắng. Ví dụ `?status=TODO,PROGRESS,DONE,HOLD` lấy mọi việc **trừ việc đã hủy**. Cũng nhận dạng lặp tham số `?status=DONE&status=HOLD`. Chỉ cần 1 giá trị sai là trả 400 `Trạng thái không hợp lệ` |

Ví dụ màn hình chốt lúc chấm công ra (chỉ lấy việc chưa hủy):
```
GET /api/v1/daily-checklists?status=TODO,PROGRESS,DONE,HOLD
```

```json
{
  "success": true,
  "data": {
    "date": "2026-11-10",
    "today": "2026-11-10",
    "is_today": true,
    "total_all": 4,
    "tasks": [
      { "_id": "6af27f2ff2784b809958c3fb", "content": "Sửa main Dell Latitude 5490\nThay IC nguồn", "progress": 40, "status": "PROGRESS", "status_label": "Đang làm" },
      { "_id": "6af27f2ff2784b809958c3fc", "content": "Gọi lại khách hẹn lấy máy", "progress": 50, "status": "PROGRESS", "status_label": "Đang làm" }
    ]
  }
}
```
- `is_today: false` → checklist ngày cũ, app ẩn các nút thêm, sửa, xóa.
- `total_all` là tổng số việc trong ngày, **không tính bộ lọc** `status`. `total_all = 0` nghĩa là hôm nay chưa lập checklist.
- Ngày chưa có checklist vẫn trả 200, với `tasks: []`.

#### 2. `GET /carry-over-suggestions`: Đề xuát công việc **chưa hoàn thành**  để tiếp tục từ checklist hôm qua
```json
{
  "success": true,
  "data": {
    "day": "2026-11-10",
    "suggest_from_date": "2026-11-09",
    "suggestions": [
      { "_id": "6af27f2ff2784b809958c3f7", "content": "Sửa main Dell Latitude 5490\nThay IC nguồn", "progress": 40, "status": "PROGRESS", "status_label": "Đang làm" },
      { "_id": "6af27f2ff2784b809958c3f8", "content": "Gọi lại khách hẹn lấy máy", "progress": 50, "status": "HOLD", "status_label": "Tạm hoãn" }
    ]
  }
}
```
- `suggest_from_date` là ngày của checklist gần nhất được dùng để gợi ý. Không có gợi ý thì là `null` và `suggestions: []`.
- Các `_id` ở đây là mã việc **của ngày cũ**. Chỉ dùng chúng để gửi vào `carry_over_ids`, không dùng để sửa.

#### 3. `POST /tasks`: thêm việc vào checklist hôm nay → **201**

| Body | Kiểu | Mô tả |
|---|---|---|
| `new_tasks` | `[{ content, status?, progress? }]` | Việc mới. `content` bắt buộc. `status` mặc định `TODO`; app nên gửi `PROGRESS` khi tạo ở màn hình sau chấm công vào. `progress` mặc định 0 |
| `carry_over_ids` | `[string]` | Mã việc lấy từ `GET /carry-over-suggestions` |

Danh sách Cần ít nhất 1 trong 2 trường trên.

**Gọi được nhiều lần trong ngày.** Lần đầu tạo checklist hôm nay, các lần sau thêm việc vào checklist đó (tối đa 30 việc/ngày).
- `carry_over_ids` phải lấy từ lần gọi `GET /carry-over-suggestions` **mới nhất**.
- Việc nào đã chép sang hôm nay thì không còn trong gợi ý. Gửi lại mã của việc đó sẽ bị lỗi 400, và **cả request bị từ chối**, kể cả phần `new_tasks`.
- Chỉ thêm việc mới thì bỏ trường `carry_over_ids` hoặc gửi `[]`.

Ví dụ lúc chấm công vào: chép 2 việc cũ và thêm 1 việc mới.
```json
{
  "carry_over_ids": ["6af27f2ff2784b809958c3f7", "6af27f2ff2784b809958c3f8"],
  "new_tasks": [{ "content": "Báo giá thay màn hình cho khách", "status": "PROGRESS" }]
}
```
```json
{
  "success": true,
  "data": {
    "day": "2026-11-10",
    "carried_over": 2,
    "created": 1,
    "tasks": [
      { "_id": "6af27f2ff2784b809958c3fb", "content": "Sửa main Dell Latitude 5490\nThay IC nguồn", "progress": 40, "status": "PROGRESS", "status_label": "Đang làm" },
      { "_id": "6af27f2ff2784b809958c3fc", "content": "Gọi lại khách hẹn lấy máy", "progress": 50, "status": "PROGRESS", "status_label": "Đang làm" },
      { "_id": "…", "content": "Báo giá thay màn hình cho khách", "progress": 0, "status": "PROGRESS", "status_label": "Đang làm" }
    ]
  }
}
```
Thêm 1 việc từ tab Danh sách: body `{ "new_tasks": [{ "content": "Vệ sinh máy cho khách lẻ" }] }`. Kết quả là việc mới với `status: "TODO"`, `progress: 0`.

`tasks` trong kết quả là **các mục vừa tạo**, có `_id` mới để sửa hoặc xóa về sau.

Các lỗi thường gặp:
- `Hãy chọn hoặc nhập ít nhất 1 công việc`
- `Vui lòng nhập nội dung công việc`
- `Trạng thái không hợp lệ` (khi gửi `CANCELLED`)
- `Có công việc không còn trong danh sách gợi ý (đã chép sang hôm nay hoặc không phải của bạn). Vui lòng tải lại.`
- `Mỗi ngày tối đa 30 công việc (hôm nay đã có N)`

#### 4. `PATCH /tasks/{taskId}`: sửa 1 việc của hôm nay

Body gồm các trường cần đổi, trường nào không gửi thì giữ nguyên: `content`, `progress` (0–100), `status` (`TODO` / `PROGRESS` / `HOLD` / `DONE`).
```json
{ "progress": 70, "content": "Sửa main Dell Latitude 5490\nThay IC nguồn, test 2 tiếng" }
```
```json
{ "success": true, "data": { "_id": "6af27f2ff2784b809958c3fb", "content": "Sửa main Dell Latitude 5490\nThay IC nguồn, test 2 tiếng", "progress": 70, "status": "PROGRESS", "status_label": "Đang làm" } }
```
Các lỗi thường gặp:
- 403 `Chỉ sửa hoặc xóa được công việc trong checklist hôm nay`
- 404 `Không tìm thấy công việc`
- 400 `Công việc đã bị hủy, không sửa được nữa`
- 400 `Tiến độ phải từ 0 đến 100`
- 400 `Mã công việc không hợp lệ`

#### 5. `DELETE /tasks/{taskId}`: xóa (hủy) 1 việc của hôm nay
```json
{ "success": true, "data": { "_id": "6af27f2ff2784b809958c401", "content": "Vệ sinh máy cho khách lẻ", "progress": 0, "status": "CANCELLED", "status_label": "Đã hủy" } }
```
Lỗi giống mục 5. Xóa việc đã hủy rồi thì trả 400.

#### 6. `PATCH /tasks`: chốt nhiều việc 1 lần (sau khi chấm công ra)

Body `{ "items": [{ "id", "progress?", "status?" }] }`, tối đa 30 dòng. App nên gửi các việc **chưa hủy** của hôm nay. Dòng có `status: "DONE"` thì không cần gửi `progress`.
```json
{
  "items": [
    { "id": "6af27f2ff2784b809958c3fb", "status": "DONE" },
    { "id": "6af27f2ff2784b809958c3fc", "progress": 60, "status": "HOLD" },
    { "id": "6af27f2ff2784b809958c3fd", "progress": 30, "status": "PROGRESS" }
  ]
}
```
```json
{ "success": true, "data": { "day": "2026-11-10", "updated": 3 } }
```
`updated` là số việc thực sự thay đổi; gửi lại y nguyên thì `updated: 0`.

Các lỗi thường gặp:
- `Danh sách công việc bị lặp`
- `Hôm nay bạn chưa có công việc nào trong checklist`
- `Có công việc không thuộc checklist hôm nay của bạn (hoặc đã hủy). Vui lòng tải lại.`
- `Tiến độ phải từ 0 đến 100` (gặp lỗi thì **không dòng nào** được lưu)


## 6. Luồng dữ liệu hiện tại Web `/cham-cong-web`

1. Web gọi `GET /permissions`.
   - `bao_cao = false` → ẩn chức năng checklist.
   - `xem_bao_cao = true` → hiện mục Báo cáo.
2. **Chấm công vào thành công:** gọi `GET /` (hôm nay).
   - Nếu `total_all = 0` thì mở màn hình "Công việc hôm nay": gọi `GET /carry-over-suggestions`, hiện các việc cũ có ô tick, và ô nhập việc mới.
   - Bấm "Bắt đầu làm việc" thì gửi `POST /tasks`, việc mới kèm `status: "PROGRESS"`.
   - Nếu người dùng bấm "Để sau" thì lưu đánh dấu theo ngày ngay trên máy để không hỏi lại.
3. **Màn hình Danh sách:** chọn ngày rồi gọi `GET /?date=`.
   - Chỉ khi `is_today = true` mới hiện các nút Thêm (`POST /tasks`), Sửa (`PATCH /tasks/{id}`), Xóa (`DELETE /tasks/{id}`).
   - Nút "Tiếp tục việc cũ" gọi `GET /carry-over-suggestions`, sau đó `POST /tasks` với `carry_over_ids`.
4. **Chấm công ra thành công:** gọi `GET /` (hôm nay). API trả về danh sách **tất cả** việc.
   - Nếu còn việc thì mở màn hình chốt: mỗi việc có thanh tiến độ riêng và ô chọn trạng thái.
   - Bấm Gửi thì gọi `PATCH /tasks`.
   - Lưu đánh dấu "đã gửi" hoặc "để sau" theo ngày trên máy.
5. **Ngày cũ:** checklist ngày cũ luôn chỉ xem. Muốn làm tiếp thì dùng "Tiếp tục việc cũ".

---

#Note
 các collection liên quan: `Employee`, `Branch`, `permissions`, `permissionemployees`, `functions`. Collection sẽ làm việc chính **ghi** vào `DailyChecklist`.

