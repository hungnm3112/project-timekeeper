# BƯỚC 7: KẾ HOẠCH XÂY DỰNG MODULE `daily_report_timekeeper` (IMPLEMENTATION PLAN)
> **Dự án:** `24hgroup_noibo` — Module Báo cáo Công việc Hàng ngày (TimeKeeper)  
> **Tên Module (`<module_name>`):** `daily_report_timekeeper`  
> **Phạm vi phụ trách:** Đầy đủ 6 tầng kiến trúc cho **3 API (`API 01`, `API 07`, `API 08`)**  
> **Tài liệu tham chiếu:**  
> - [`md/module_development_guide.md`](../module_development_guide.md) (Quy chuẩn kiến trúc 6 tầng của dự án `24hgroup_noibo`)  
> - [`md/analysis/5-database-design.md`](./5-database-design.md) (Thiết kế Collection `DailyChecklist`)  
> - [`md/analysis/6-api-specification.md`](./6-api-specification.md) (Đặc tả chi tiết `API 01`, `API 07`, `API 08`)

---

## 1. Bảng Tổng Hợp Các Quyết Định Kiến Trúc Đã Chốt

| STT | Hạng mục | Quy chuẩn áp dụng |
|:---:|---|---|
| **1** | **Tên Module (`snake_case`)** | `daily_report_timekeeper` |
| **2** | **Phạm vi triển khai** | Xây dựng đầy đủ **6 tầng** theo `module_development_guide.md` (`models`, `services`, `controllers`, `routes`, `views`, `public/js`), tập trung vào **3 API: `01`, `07`, `08`** (Kiểm tra quyền, Giám sát tiến độ ngày & Báo cáo lịch tuần/tháng). |
| **3** | **Kết nối Database** | Toàn bộ các bảng (`DailyChecklist`, `Employee`, `Branch`, `permissions`, `permissionemployees`) đều nằm chung trên kết nối **`mongohethong24hdb`** (`src/config/mongohethong24hdb.js` — DB `suachualaptop24h`). |
| **4** | **Định dạng JSON Response** | Theo chuẩn `6-api-specification.md` và Postman:<br>- **Thành công (`200 OK`):** `{ "data": ... }`<br>- **Thất bại (`400 / 403 / 500`):** `{ "error": { "code": "...", "message": "...", "details": [...] } }` |
| **5** | **Quy ước Đặt tên** | - **Thư mục, file, biến, hàm, Query Params, JSON keys:** `snake_case` (`daily_report_timekeeper_service.js`, `get_daily_reports`, `from_date`, `branch_id`).<br>- **Tên Collection (Bảng) & Class:** `PascalCase` (`{ collection: 'DailyChecklist' }`, `'Employee'`, `'Branch'`, `DailyReportTimekeeperService`).<br>- **URL Route:** `kebab-case` (`/daily-report-timekeeper`, `/api/v1/daily-checklists`). |

---

## 2. Bản Đồ Cấu Trúc File Bắt Buộc Của Module `daily_report_timekeeper`

```text
src/
├── models/daily_report_timekeeper/
│   └── daily_checklist.js                               # 1. Model Mongoose (dùng mongohethong24hdb & validate.js)
├── services/daily_report_timekeeper/
│   └── daily_report_timekeeper_service.js               # 2. Service xử lý 100% Business Logic & DB queries (API 01, 07, 08)
├── controllers/
│   ├── admin/daily_report_timekeeper_controller.js      # 3a. Controller render View (Desktop / Mobile)
│   └── api/v1/daily_report_timekeeper_controller.js     # 3b. Controller tiếp nhận request, trả JSON API
├── routes/
│   ├── admin/daily_report_timekeeper.js                 # 4a. Route giao diện (Đăng ký tại src/routes/admin/index.js)
│   └── api/v1/daily_report_timekeeper.js                # 4b. Route API RESTful (Đăng ký tại index.js)
views/daily_report_timekeeper/
├── desktop.ejs                                          # 5a. Giao diện Desktop (kế thừa partials/desktop)
└── mobile.ejs                                           # 5b. Giao diện Mobile (kế thừa partials/mobile)
public/js/daily_report_timekeeper/
├── model.js                                             # 6a. Gọi AJAX API dùng chung (API 01, 07, 08)
├── desktop.js                                           # 6b. Xử lý sự kiện DOM, bảng Giám sát & Lịch trên Desktop
└── mobile.js                                            # 6c. Xử lý sự kiện DOM, giao diện chạm vuốt trên Mobile
```

---

## 3. Kế Hoạch Triển Khai Chi Tiết Từng Tầng (6 Giai Đoạn)

### Giai đoạn 1: Xây dựng Tầng Model (`src/models/daily_report_timekeeper/daily_checklist.js`)
- **Kết nối & Validate:**
  - Import `mongoose` từ `'mongoose'`.
  - Import `{ mongohethong24hdb }` từ `'../../config/mongohethong24hdb.js'`.
  - Import `* as validate` từ `'../../config/validate.js'`.
- **Cấu trúc Sub-schema `task_schema` (nhúng trong mảng `tasks[]`):**
  - `_id`: `{ ...validate.schemaObjectId, auto: true }`
  - `content`: `{ ...validate.schemaString, ...validate.schemaRequired }`
  - `progress`: `{ ...validate.schemaNumber, default: 0, min: 0, max: 100 }`
  - `status`: `{ ...validate.schemaEnumString(['TODO', 'PROGRESS', 'DONE', 'HOLD', 'CANCELLED'], 'TODO') }`
- **Cấu trúc Main Schema `daily_checklist_schema`:**
  - `employee_id`: `{ ...validate.schemaObjectId, ...validate.schemaRequired, ...validate.schemaIndex, ref: 'Employee' }`
  - `branch_id`: `{ ...validate.schemaObjectId, ...validate.schemaRequired, ...validate.schemaIndex, ref: 'Branch' }`
  - `tasks`: `{ type: [task_schema], default: [] }`
  - `date`: `{ ...validate.schemaDateTime, ...validate.schemaIndex, default: Date.now }`
- **Options & Indexes:**
  - Cấu hình tên Collection tường minh theo chuẩn `PascalCase` của `mongohethong24hdb`:
    ```javascript
    {
      collection: 'DailyChecklist',
      timestamps: true
    }
    ```
  - Export model trên kết nối `mongohethong24hdb`:
    ```javascript
    export default mongohethong24hdb.model('DailyChecklist', daily_checklist_schema);
    ```
  - Compound Index phục vụ truy vấn báo cáo ngày và lịch tuần/tháng:
    ```javascript
    daily_checklist_schema.index({ date: -1, branch_id: 1, employee_id: 1 });
    ```

---

### Giai đoạn 2: Xây dựng Tầng Service (`src/services/daily_report_timekeeper/daily_report_timekeeper_service.js`)
Khởi tạo class `DailyReportTimekeeperService` chứa toàn bộ logic xử lý của 3 API:

#### 2.1. Hằng số Mã Quyền (`UPPER_SNAKE_CASE`)
- `FUNCTION_REPORT_ID = '69710af7090eb0bbb82a89b1'` (Quyền Báo cáo — `can_report`)
- `FUNCTION_VIEW_REPORT_ID = '69710af7090eb0bbb82a89b2'` (Quyền Xem báo cáo — `can_view_report`)
- `VALID_TASK_STATUSES = ['TODO', 'PROGRESS', 'DONE', 'HOLD', 'CANCELLED']`

#### 2.2. Nghiệp vụ `API 01` — Kiểm tra Quyền hạn (`get_user_permissions(employee_id)` & `has_permission(employee_id, function_id)`)
1. Kiểm tra `mongoose.Types.ObjectId.isValid(employee_id)`.
2. Truy vấn collection `Employee` trên `mongohethong24hdb` với `{ _id: ObjectId(employee_id), Status: 'Đang hoạt động' }` (dùng `.select('ID_GroupUser Status').lean()`).
3. Kiểm tra song song (`Promise.all`) trên 2 bảng `permissions` (theo `ID_GroupUser` + `ID_Function`) và `permissionemployees` (theo `id_employee` + `id_function` + `status: true`).
4. Trả về:
   ```json
   {
     "employee_id": "64f926eb1d222677f7242a8d",
     "can_report": true,
     "can_view_report": true
   }
   ```

#### 2.3. Nghiệp vụ `API 07` — Quản lý Giám sát Tiến độ Ngày (`get_daily_reports(query_params)`)
1. **Nhận tham số (`snake_case`):** `{ date, branch_id, employee_id, status, page = 1, limit = 20 }`.
2. **Validate đầu vào:**
   - `date`: Chuẩn ISO 8601 (`YYYY-MM-DD`). Nếu không truyền, mặc định lấy ngày hôm nay theo giờ Việt Nam (`Asia/Ho_Chi_Minh`). Nếu sai định dạng $\to$ ném lỗi `VALIDATION_ERROR` (`400`).
   - `branch_id`, `employee_id`: Nếu có truyền, kiểm tra `ObjectId.isValid(...)`.
   - `status`: Nếu có truyền, kiểm tra thuộc `VALID_TASK_STATUSES`.
   - `page`, `limit`: Ép kiểu số nguyên (`page >= 1`, `1 <= limit <= 100`).
3. **Truy vấn & Tính toán:**
   - Lọc các bản ghi `DailyChecklist` có `date` nằm trong khoảng `start_of_day(date)` $\to$ `end_of_day(date)` (theo giờ VN `GMT+7`), kết hợp điều kiện `branch_id`, `employee_id`, `tasks.status`.
   - Đếm `total` bản ghi và phân trang (`.skip((page - 1) * limit).limit(limit).lean()`).
   - Tra cứu thông tin `employee_name` từ bảng `Employee` và `branch_name` từ bảng `Branch` (bằng `$in` + `.select().lean()` để tối ưu tốc độ).
   - Tính toán trực tiếp trên từng bản ghi trong ngày:
     - `total_tasks = tasks.length`
     - `done_tasks = tasks.filter(t => t.status === 'DONE').length`
     - `average_progress = total_tasks > 0 ? Number((sum_progress / total_tasks).toFixed(2)) : 0`
   - Trả về `{ items, pagination: { page, limit, total, total_pages } }`.

#### 2.4. Nghiệp vụ `API 08` — Báo cáo Tổng hợp Lịch Tuần / Tháng (`get_schedule_summary(query_params)`)
1. **Nhận tham số (`snake_case`):** `{ from_date, to_date, branch_id, employee_id, status }`.
2. **Validate đầu vào:**
   - Bắt buộc truyền `from_date` và `to_date` đúng định dạng `YYYY-MM-DD`, đảm bảo `to_date >= from_date` và khoảng cách không vượt quá 93 ngày.
   - Kiểm tra `ObjectId.isValid` cho `branch_id`, `employee_id` và kiểm tra `status` (nếu có).
3. **Truy vấn & Gom nhóm theo Ngày trên lịch (`date`):**
   - Lọc các bản ghi trong khoảng `start_of_day(from_date)` $\to$ `end_of_day(to_date)` kèm bộ lọc `branch_id`, `employee_id`, `status`.
   - Tra cứu `employee_name` (`Employee`) và `branch_name` (`Branch`).
   - Nếu có bộ lọc `status`, lọc mảng `tasks` chỉ giữ các phần tử khớp trạng thái (hoặc giữ nguyên tùy nhu cầu hiển thị).
   - Gom nhóm theo từng ngày `YYYY-MM-DD` (theo giờ VN), sắp xếp theo thứ tự ngày:
     - Mỗi phần tử trong `items` đại diện cho **1 ngày trên lịch (`date`)**, bên trong là mảng **`employees`** gồm `{ checklist_id, employee_id, employee_name, branch_id, branch_name, tasks }`.
   - Trả về `{ from_date, to_date, items }`.

---

### Giai đoạn 3: Xây dựng Tầng Controller (`admin` & `api/v1`)

1. **Admin Controller (`src/controllers/admin/daily_report_timekeeper_controller.js`):**
   - Class `DailyReportTimekeeperAdminController`:
     - Hàm `render_index(req, res)`: Kiểm tra `res.locals.isMobile` để render view `'daily_report_timekeeper/mobile'` hoặc `'daily_report_timekeeper/desktop'`, truyền `title: 'Báo cáo công việc hàng ngày'`, `activeTab`, `activeSidebar`.
2. **API v1 Controller (`src/controllers/api/v1/daily_report_timekeeper_controller.js`):**
   - Class `DailyReportTimekeeperApiController`:
     - `get_permissions(req, res)` (`API 01`): Lấy `req.auth._id`, gọi `daily_report_timekeeper_service.get_user_permissions(req.auth._id)` và trả `res.status(200).json({ data })`.
     - `get_reports(req, res)` (`API 07`): Gọi `daily_report_timekeeper_service.get_daily_reports(req.query)` và trả `res.status(200).json({ data })`.
     - `get_summary(req, res)` (`API 08`): Gọi `daily_report_timekeeper_service.get_schedule_summary(req.query)` và trả `res.status(200).json({ data })`.
     - Bọc `try/catch` thống nhất: nếu là lỗi nghiệp vụ/validate (`error.status_code`), trả về đúng HTTP status kèm `{ error: { code: error.code, message: error.message, details: error.details || [] } }`.

---

### Giai đoạn 4: Xây dựng Tầng Route & Middleware Phân Quyền

1. **Middleware kiểm tra quyền Xem báo cáo (`require_view_report_permission`):**
   - Đặt tại `src/routes/api/v1/daily_report_timekeeper.js` (hoặc middleware dùng chung):
   - Kiểm tra `const allowed = await daily_report_timekeeper_service.has_permission(req.auth._id, FUNCTION_VIEW_REPORT_ID);`
   - Nếu `!allowed`, trả ngay `403 Forbidden`:
     ```json
     {
       "error": {
         "code": "FORBIDDEN_REPORT_ACCESS",
         "message": "Bạn không có quyền xem báo cáo công việc (yêu cầu quyền 69710af7090eb0bbb82a89b2).",
         "details": []
       }
     }
     ```
2. **Route API (`src/routes/api/v1/daily_report_timekeeper.js`):**
   - `GET /permissions` $\to$ `requireAuthAPI`, `api_controller.get_permissions`
   - `GET /reports` $\to$ `requireAuthAPI`, `require_view_report_permission`, `api_controller.get_reports`
   - `GET /summary` $\to$ `requireAuthAPI`, `require_view_report_permission`, `api_controller.get_summary`
   - Đăng ký vào `index.js`:
     ```javascript
     import daily_report_timekeeper_routers_api from './src/routes/api/v1/daily_report_timekeeper.js';
     app.use('/api/v1/daily-checklists', daily_report_timekeeper_routers_api);
     ```
3. **Route Admin (`src/routes/admin/daily_report_timekeeper.js`):**
   - `GET /` $\to$ `admin_controller.render_index`
   - Đăng ký vào `src/routes/admin/index.js`:
     ```javascript
     import daily_report_timekeeper_admin_routes from './daily_report_timekeeper.js';
     router.use('/daily-report-timekeeper', requireAuth, daily_report_timekeeper_admin_routes);
     ```

---

### Giai đoạn 5: Xây dựng Tầng Giao Diện EJS (`views/daily_report_timekeeper/`)

1. **`views/daily_report_timekeeper/desktop.ejs`:**
   - Kế thừa `../partials/desktop/head.ejs`, `sidebar.ejs`, `header.ejs`, `script.ejs`.
   - Giao diện gồm 2 chế độ xem (Tab chuyển đổi có icon Material Symbols):
     - **Tab 1 — Giám sát Tiến độ Ngày (`API 07`):** Thanh bộ lọc (Chọn ngày `date`, Chi nhánh `branch_id`, Nhân viên `employee_id`, Trạng thái `status`, nút Lọc kèm icon `filter_alt`), bảng dữ liệu hiển thị nhân viên, chi nhánh, tổng số việc (`total_tasks`), số việc hoàn thành (`done_tasks`), thanh tiến độ trung bình (`average_progress`), danh sách chi tiết `tasks[]` và thanh Phân trang (`pagination`).
     - **Tab 2 — Báo cáo Lịch Tuần / Tháng (`API 08`):** Thanh bộ lọc khoảng ngày (`from_date` – `to_date`), Chi nhánh (`branch_id`), Nhân viên (`employee_id`), hiển thị lưới Lịch theo từng ngày (`date`) $\to$ danh sách nhân viên và tiến độ từng công việc trong ngày.
2. **`views/daily_report_timekeeper/mobile.ejs`:**
   - Kế thừa `../partials/mobile/head.ejs`, `sidebar_drawer.ejs`, `header.ejs`, `bottom_nav.ejs`, `footer.ejs`.
   - Thiết kế dạng thẻ Card thu gọn (Accordion) tối ưu thao tác chạm vuốt trên điện thoại cho cả 2 tab Giám sát ngày và Lịch tuần/tháng.

---

### Giai đoạn 6: Xây dựng Tầng Client Vanilla JS (`public/js/daily_report_timekeeper/`)

1. **`public/js/daily_report_timekeeper/model.js`:**
   - Chứa 3 hàm gọi AJAX API tới `/api/v1/daily-checklists/permissions`, `/api/v1/daily-checklists/reports`, `/api/v1/daily-checklists/summary`.
2. **`public/js/daily_report_timekeeper/desktop.js`:**
   - Khi tải trang: gọi `API 01` kiểm tra `can_view_report`. Nếu không có quyền, hiển thị cảnh báo bằng **SweetAlert2 (`Swal.fire`)**.
   - Xử lý lọc dữ liệu, chuyển tab Giám sát Ngày $\leftrightarrow$ Lịch Tuần/Tháng, phân trang, hiển thị loading toàn trang khi đang tải và chống XSS khi render nội dung công việc (`content`).
3. **`public/js/daily_report_timekeeper/mobile.js`:**
   - Xử lý tương tác lọc, đóng/mở chi tiết công việc theo từng nhân viên / từng ngày trên giao diện Mobile.

