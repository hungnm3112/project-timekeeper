# Tài liệu Đặc tả API Kiểm tra Phân quyền Chấm công

Tài liệu này mô tả cấu trúc Database và logic xử lý để xây dựng API kiểm tra phân quyền **Chấm công (Báo cáo / Xem báo cáo)** khi kết nối trực tiếp vào MongoDB `suachualaptop24h`.

---

## 1. Danh sách Mã Phân Quyền (`function_id`)

Các phân quyền được định nghĩa cố định trong bảng `functions` với `_id` (`ObjectId`) như sau:

| Tên quyền | `function_id` (`ObjectId`) | Cấp độ | `ID_Function_Parent` | Ý nghĩa nghiệp vụ |
| :--- | :--- | :--- | :--- | :--- |
| **Chấm công** | `69710af7090eb0bbb82a89b0` | Cấp 1 (Cha) | `null` | Nhóm quyền cha trên cây phân quyền |
| **Báo cáo** | `69710af7090eb0bbb82a89b1` | Cấp 2 (Con) | `69710af7090eb0bbb82a89b0` | Nhân viên được gán quyền này **phải thực hiện báo cáo** khi Check-in / Check-out |
| **Xem báo cáo** | `69710af7090eb0bbb82a89b2` | Cấp 2 (Con) | `69710af7090eb0bbb82a89b0` | Quản lý được gán quyền này **được phép xem danh sách báo cáo** Check-in / Check-out |

> **Lưu ý:** Trong hệ thống này, quyền Cha và quyền Con hoạt động độc lập (tích chọn quyền Cha không tự động cấp quyền Con trong DB). Khi kiểm tra quyền ở API, luôn kiểm tra trực tiếp theo mã `function_id` của cấp Con (`69710af7090eb0bbb82a89b1` hoặc `69710af7090eb0bbb82a89b2`).

---

## 2. Cấu trúc các Collection liên quan trong DB `suachualaptop24h`

> **CẢNH BÁO QUAN TRỌNG:** Tên bảng (Collection) và tên trường (Field) giữa bảng quyền Nhóm và bảng quyền Cá nhân có quy ước đặt tên khác nhau (`PascalCase` và `snake_case`). Cần truy vấn đúng chính xác tên trường như bảng dưới đây:

### 2.1. Bảng Nhân viên — Collection: `Employee` *(chữ E viết hoa, số ít)*
* `_id` (`ObjectId`): ID của nhân viên.
* `ID_GroupUser` (`ObjectId`): ID nhóm người dùng (chức vụ) của nhân viên.
* `Status` (`String`): Trạng thái hoạt động (chỉ hợp lệ khi `Status === "Đang hoạt động"`).
* `Phone` (`String`): Số điện thoại đăng nhập.

### 2.2. Bảng Phân quyền theo Nhóm — Collection: `permissions` *(chữ thường, số nhiều)*
* `ID_GroupUser` (`ObjectId` - *PascalCase*): Tham chiếu tới `Employee.ID_GroupUser`.
* `ID_Function` (`ObjectId` - *PascalCase*): Mã quyền (`function_id`).
* `Status` (`Boolean` - *PascalCase*): Trạng thái cấp quyền (`true`).

### 2.3. Bảng Phân quyền theo Cá nhân — Collection: `permissionemployees` *(chữ thường, số nhiều)*
* `id_employee` (`ObjectId` - *snake_case*): Tham chiếu tới `Employee._id`.
* `id_function` (`ObjectId` - *snake_case*): Mã quyền (`function_id`).
* `status` (`Boolean` - *snake_case*): Trạng thái cấp quyền (`true`).

---

## 3. Luồng Logic Kiểm tra Phân quyền

Một nhân viên được xác định là **CÓ QUYỀN** đối với một `function_id` nếu thỏa mãn các bước sau:

1. **Bước 1 (Kiểm tra tài khoản nhân viên):**
   * Truy vấn bảng `Employee` theo `_id = ObjectId(employee_id)`.
   * Nếu không tìm thấy hoặc `employee.Status !== "Đang hoạt động"` $\rightarrow$ Trả về `false` (Không có quyền).
2. **Bước 2 (Kiểm tra quyền Nhóm HOẶC quyền Cá nhân):**
   * **Điều kiện A (Quyền Nhóm):** Trong bảng `permissions` có tồn tại bản ghi thỏa mãn:
     * `ID_GroupUser == ObjectId(employee.ID_GroupUser)`
     * `ID_Function == ObjectId(function_id)`
     * `Status == true` *(hoặc không truyền Status nếu mặc định bản ghi tồn tại là true)*
   * **Điều kiện B (Quyền Cá nhân):** Trong bảng `permissionemployees` có tồn tại bản ghi thỏa mãn:
     * `id_employee == ObjectId(employee._id)`
     * `id_function == ObjectId(function_id)`
     * `status == true`
   * Nếu **(Điều kiện A đúng) HOẶC (Điều kiện B đúng)** $\rightarrow$ Trả về `true` (Có quyền). Ngược lại trả về `false`.

---

## 4. Code Mẫu Triển khai (Node.js / MongoDB)

```javascript
const { ObjectId } = require('mongodb'); // Hoặc const { Types: { ObjectId } } = require('mongoose');

const CHAM_CONG_PERMISSIONS = {
    PARENT: '69710af7090eb0bbb82a89b0',       // Chấm công (Cha)
    BAO_CAO: '69710af7090eb0bbb82a89b1',      // Báo cáo khi checkin/checkout
    XEM_BAO_CAO: '69710af7090eb0bbb82a89b2',  // Xem báo cáo checkin/checkout
};

/**
 * Kiểm tra nhân viên có một quyền cụ thể hay không
 * @param {import('mongodb').Db} db - Instance kết nối tới DB 'suachualaptop24h'
 * @param {string} employeeId - ID của nhân viên
 * @param {string} functionId - ID của quyền cần kiểm tra
 * @returns {Promise<boolean>}
 */
async function hasPermission(db, employeeId, functionId) {
    if (!ObjectId.isValid(employeeId) || !ObjectId.isValid(functionId)) {
        return false;
    }

    const empObjId = new ObjectId(employeeId);
    const funcObjId = new ObjectId(functionId);

    // 1. Kiểm tra nhân viên tồn tại và đang hoạt động
    const employee = await db.collection('Employee').findOne(
        { _id: empObjId },
        { projection: { ID_GroupUser: 1, Status: 1 } }
    );

    if (!employee || employee.Status !== 'Đang hoạt động') {
        return false;
    }

    // 2. Kiểm tra song song quyền Nhóm (permissions) và quyền Cá nhân (permissionemployees)
    const [groupCount, personalCount] = await Promise.all([
        employee.ID_GroupUser && ObjectId.isValid(employee.ID_GroupUser)
            ? db.collection('permissions').countDocuments({
                  ID_GroupUser: new ObjectId(employee.ID_GroupUser),
                  ID_Function: funcObjId,
                  Status: true,
              })
            : 0,
        db.collection('permissionemployees').countDocuments({
            id_employee: empObjId,
            id_function: funcObjId,
            status: true,
        }),
    ]);

    return groupCount > 0 || personalCount > 0;
}

/**
 * Hàm lấy đồng thời cả 2 quyền Báo cáo và Xem báo cáo của nhân viên
 * @param {import('mongodb').Db} db - Instance kết nối tới DB 'suachualaptop24h'
 * @param {string} employeeId - ID của nhân viên
 */
async function getTimekeepingReportPermissions(db, employeeId) {
    const [canReport, canViewReport] = await Promise.all([
        hasPermission(db, employeeId, CHAM_CONG_PERMISSIONS.BAO_CAO),
        hasPermission(db, employeeId, CHAM_CONG_PERMISSIONS.XEM_BAO_CAO),
    ]);

    return {
        employee_id: employeeId,
        can_report: canReport,           // true: nhân viên cần báo cáo khi checkin/checkout
        can_view_report: canViewReport,  // true: quản lý được phép xem báo cáo
    };
}
```

---

## 5. Thiết kế Endpoint API Đề xuất

### Endpoint 1: Kiểm tra theo mã quyền bất kỳ
* **Method / URL:** `GET /api/permission/check`
* **Query Params:**
  * `employee_id`: ID nhân viên (hoặc tự lấy từ `req.auth._id` qua Token).
  * `function_id`: Mã quyền cần kiểm tra (VD: `69710af7090eb0bbb82a89b1`).
* **Response mẫu:**
```json
{
  "status": 200,
  "has_permission": true
}
```

### Endpoint 2: Kiểm tra quyền Báo cáo Chấm công (Gộp cả 2 quyền)
* **Method / URL:** `GET /api/timekeeping/report-permission`
* **Response mẫu:**
```json
{
  "status": 200,
  "data": {
    "employee_id": "60ed4bef5fdfac189a4c116f",
    "can_report": true,
    "can_view_report": false
  }
}
```