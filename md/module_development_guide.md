# HƯỚNG DẪN QUY CHUẨN XÂY DỰNG MODULE MỚI TRONG `src/` (DỰ ÁN 24HGROUP_NOIBO)

Tài liệu này cung cấp toàn bộ **Khung quy chuẩn kiến trúc (Architectural Blueprint)** và các quy tắc bắt buộc phải tuân theo khi khởi tạo hoặc phát triển một module mới bên trong thư mục `src/` của dự án `24hgroup_noibo`.

---

## 1. Bản Đồ Cấu Trúc File Bắt Buộc Của Một Module Mới

Giả sử module mới có tên là `<module_name>` (ví dụ: `asset_management`), cấu trúc thư mục và file bắt buộc phải có đầy đủ 6 tầng:

```text
src/
├── models/<module_name>/
│   └── <entity>.js                        # 1. Model Mongoose (kết nối DB, dùng validate.js)
├── services/<module_name>/
│   └── <entity>_service.js                # 2. Service xử lý 100% Business Logic & DB queries
├── controllers/
│   ├── admin/<module_name>_controller.js  # 3a. Controller render View (Desktop / Mobile)
│   └── api/v1/<module_name>_controller.js # 3b. Controller tiếp nhận request, trả JSON API
├── routes/
│   ├── admin/<module_name>.js             # 4a. Route giao diện (Đăng ký tại src/routes/admin/index.js)
│   └── api/v1/<module_name>.js            # 4b. Route API RESTful (Đăng ký tại index.js)
views/<module_name>/
├── desktop.ejs                            # 5a. Giao diện Desktop (kế thừa partials/desktop)
└── mobile.ejs                             # 5b. Giao diện Mobile (kế thừa partials/mobile)
public/js/<module_name>/
├── model.js                               # 6a. Gọi AJAX API dùng chung (GET, POST, PUT, DELETE)
├── desktop.js                             # 6b. Xử lý sự kiện DOM, bảng dữ liệu trên Desktop
└── mobile.js                              # 6c. Xử lý sự kiện DOM, giao diện chạm vuốt trên Mobile
```

---

## 2. Các Quy Tắc Cốt Lõi Bắt Buộc Tuân Thủ

### Quy tắc 1: Đặt tên (Naming Conventions)
* **File, folder, biến, hàm:** 100% viết thường dùng `snake_case` (ví dụ: `asset_service.js`, `get_asset_list()`, `const asset_id = ...`). Tuyệt đối không dùng `camelCase` hay `PascalCase` cho tên file/thư mục.
* **Tên Class:** Dùng `PascalCase` theo chuẩn OOP (`class AssetService {}`, `class AssetController {}`).
* **Hằng số & Biến môi trường:** Dùng `UPPER_SNAKE_CASE` (`const MAX_RETRY_COUNT = 3;`).
* **Tên API & URL Route:** Dùng `kebab-case` (`/asset-management`, `/api/v1/asset-management`).

### Quy tắc 2: Tầng Model & Database (Bắt buộc dùng `validate.js`)
* **Chọn đúng kết nối Database:**
  * Module nội bộ mới: Dùng `mainDbConnection` (`src/config/mongodb.js` — database `group24h`).
  * Module liên quan đến dữ liệu gốc công ty (Nhân sự, Chi nhánh, GPS): Dùng `mongohethong24hdb` (`src/config/mongohethong24hdb.js`).
* **Khai báo Schema:** Không tự khai báo kiểu `type` thủ công cho các trường cơ bản. Bắt buộc import `validate.js` và dùng toán tử Spread `...`:
  * String: `...validate.schemaString`
  * ObjectId: `...validate.schemaObjectId`
  * Number: `...validate.schemaNumber`
  * Boolean: `...validate.schemaBoolean` hoặc `...validate.schemaBooleanFalse`
  * Date/Time: `...validate.schemaDateTime`
  * Index / Bắt buộc: `...validate.schemaIndex`, `...validate.schemaRequired`
* **Schema Options:** Tên Collection viết `PascalCase` tường minh: `{ collection: 'Asset', timestamps: true }`.
* **Truy vấn:** Luôn dùng `.lean()` và `.select()` cho các query đọc để tối ưu hiệu năng; kiểm tra `mongoose.Types.ObjectId.isValid(id)` trước khi query.

**Ví dụ Model chuẩn:**
```javascript
import mongoose from 'mongoose';
import { mainDbConnection } from '../../config/mongodb.js';
import * as validate from '../../config/validate.js';

const schema = new mongoose.Schema({
    name: {
        ...validate.schemaString,
        ...validate.schemaRequired
    },
    branch_id: {
        ...validate.schemaObjectId,
        ...validate.schemaIndex,
        ref: 'Branch'
    },
    price: {
        ...validate.schemaNumber
    },
    status: {
        ...validate.schemaEnumString(['active', 'inactive'], 'active')
    }
}, {
    collection: 'Asset',
    timestamps: true
});

export default mainDbConnection.model('Asset', schema);
```

### Quy tắc 3: Tầng Service & Controller (Tách biệt trách nhiệm)
* **Service:** Chứa **toàn bộ logic nghiệp vụ**, các câu lệnh `.find()`, `.aggregate()`, xử lý tính toán. Tuyệt đối không viết Aggregation Pipeline hay query DB trong Controller.
* **Controller:** Chỉ làm nhiệm vụ tiếp nhận dữ liệu từ `req.body`, `req.query`, `req.auth`, gọi Service và trả về response:
  * Admin Controller: Trả về `res.render(...)`.
  * API Controller: Luôn bọc `try/catch` và trả về JSON định dạng chuẩn:
    * Thành công: `{ success: true, message: '...', data: result }`
    * Lỗi: `{ success: false, message: error.message || 'Lỗi server' }`

### Quy tắc 4: Tách Biệt Giao Diện Desktop vs Mobile
* Hệ thống tự động nhận diện thiết bị qua `req.isMobile` / `res.locals.isMobile`.
* Controller render view:
  ```javascript
  renderIndex(req, res) {
      const view = res.locals.isMobile ? 'asset_management/mobile' : 'asset_management/desktop';
      res.render(view, {
          activeTab: 'departments',
          activeSidebar: 'home',
          title: 'Quản lý tài sản'
      });
  }
  ```
* Kế thừa hệ thống layout và partials:
  * Desktop: `<%- include("../partials/desktop/head.ejs") %>`, `sidebar.ejs`, `header.ejs`, `script.ejs`.
  * Mobile: `<%- include("../partials/mobile/head.ejs") %>`, `sidebar_drawer.ejs`, `header.ejs`, `bottom_nav.ejs`, `footer.ejs`.
* **Tránh lỗi IDE:** Không nhúng thẻ `<%= %>` hoặc `<%- %>` trực tiếp vào thuộc tính `style="..."` của HTML (dùng CSS Class thay thế).
* **Chống Stored XSS:** Dữ liệu người dùng hiển thị qua `<%= %>`, không dùng `<%- %>` trừ khi đã được lọc qua `sanitize-html`.

### Quy tắc 5: Tầng Frontend Client JS & UI
* **Tổ chức thư mục:** `public/js/<module_name>/` gồm 3 file:
  * `model.js`: Chứa các hàm gọi AJAX API (dùng helper tại `public/js/helper/ajax.js`).
  * `desktop.js`: Xử lý tương tác DOM trên Desktop.
  * `mobile.js`: Xử lý tương tác DOM trên Mobile.
* **Công nghệ JS:** 100% viết bằng JavaScript thuần (Vanilla JS), không dùng jQuery ở code mới.
* **Trải nghiệm người dùng (UX/UI):**
  * Không dùng `alert()` mặc định của trình duyệt; bắt buộc dùng **SweetAlert2** (`Swal.fire`).
  * Tất cả các nút bấm thao tác (Thêm, Sửa, Xóa, Lọc...) **bắt buộc phải có icon** (Material Symbols).
  * Các màn hình danh sách bắt buộc phải có: **Ô tìm kiếm, bộ lọc Từ ngày - Đến ngày, và Phân trang**.
  * Khi người dùng bấm Lưu, Thêm, Sửa, Xóa phải có **loading toàn trang** để tránh click đúp nhiều lần.

### Quy tắc 6: Đăng Ký Route & Middleware Xác Thực
* **Route Admin (giao diện):** Đăng ký vào `src/routes/admin/index.js`, luôn bọc qua middleware `requireAuth`:
  ```javascript
  import assetAdminRoutes from './asset.js';
  router.use('/asset-management', requireAuth, assetAdminRoutes);
  ```
* **Route API:** Đăng ký vào `index.js`, luôn kiểm tra qua middleware `requireAuthAPI`:
  ```javascript
  import assetRoutersAPI from './src/routes/api/v1/asset.js';
  app.use('/api/v1/asset-management', assetRoutersAPI);
  ```
* Bắt buộc kiểm tra quyền/xác thực ở tầng **Middleware**, không kiểm tra thủ công trong thân hàm Controller.

---

## 3. Mẫu Prompt Chuẩn Dùng Để Giao Việc Cho AI

Khi muốn AI viết một module mới, hãy copy đoạn prompt mẫu dưới đây và điền thông tin module:

> *"Tôi muốn viết một module mới tên là `[tên_module]` trong thư mục `src`. Hãy tuân thủ nghiêm ngặt các quy tắc kiến trúc của dự án `24hgroup_noibo`:*
> 1. *Viết theo mô hình MVC + Service Layer, phân tách rõ: Model (dùng spread từ `validate.js`), Service (chứa toàn bộ DB query/logic), Controller (admin render view & api v1 trả JSON chuẩn).*
> 2. *Tên file, thư mục, hàm, biến dùng `snake_case`, Class dùng `PascalCase`, URL route dùng `kebab-case`.*
> 3. *Tách riêng 2 file giao diện EJS: `desktop.ejs` và `mobile.ejs`.*
> 4. *Tách JS frontend vào `public/js/[tên_module]/`: gồm `model.js` (gọi AJAX API), `desktop.js` và `mobile.js` riêng biệt. Nút bấm có icon, thông báo bằng SweetAlert2, danh sách có search + date filter + phân trang.*
> 5. *Đăng ký route vào `src/routes/admin/index.js` (kèm `requireAuth`) và `index.js` (kèm `requireAuthAPI`)."*

