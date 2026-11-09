# BƯỚC 4: SƠ ĐỒ PHÂN RÃ CHỨC NĂNG (FDD 3 CẤP) & MA TRẬN RBAC
> **Dự án:** Project TimeKeeper (Module Quản lý Checklist Công việc tích hợp Chấm công)  
> **Đầu vào:** [`md/analysis/2-actor-identification.md`](./2-actor-identification.md), [`md/analysis/3-dfd-context-diagram.md`](./3-dfd-context-diagram.md)  
> **Đầu ra phục vụ:** Bước 5 (Ánh xạ FDD sang Database), Bước 6 (Thiết kế RESTful API)

---

## 1. Kiểm chứng Nguyên tắc MECE & Công thức Bảo toàn Chức năng Cấp 2

### 1.1. Công thức Bảo toàn Chức năng Cấp 2
$$\text{Tổng Chức năng Cấp 2} = \text{Tổng Hành động Actor (Bước 3)} - \text{Số Hành động trùng/hợp nhất} + \text{Số Tác vụ tự động (Bước 2)}$$

* **Tổng Hành động Actor ở Bước 3:** `9` hành động (`[Nhân viên - 01..04]`, `[Quản lý - 01..02]`, `[Hệ thống gốc - 01..03]`).
* **Số Hành động hợp nhất theo nguyên tắc MECE:** `1` điểm hợp nhất:
  * `[Hệ thống gốc - 01]` *(Xử lý chấm công Check-in / Check-out tức thời)* $\longrightarrow$ Hợp nhất vào nhịp gọi API chấm công tức thời bên trong **`FDD-04` (Thực hiện Check-in và lập Checklist ngày)** và **`FDD-05` (Thực hiện Check-out và cập nhật tiến độ)**.
* **Số Tác vụ tự động ở Bước 2:** `0` tác vụ (truy vấn và thống kê trực tiếp trên collection `daily_checklist`).

$$\Longrightarrow \text{Tổng Chức năng Cấp 2} = 9 - 1 + 0 = \mathbf{8\text{ Chức năng Cấp 2 } (\text{FDD-01} \to \text{FDD-08})}$$

---

## 2. Sơ đồ Cây Phân rã Chức năng FDD 4 Tầng (`Cấp 0 ➔ Cấp 1 ➔ Cấp 2 ➔ Cấp 3`)

```mermaid
%%{init: {
  'theme': 'dark',
  'themeVariables': { 'fontSize': '14px' },
  'flowchart': {
    'rankSpacing': 45,
    'nodeSpacing': 16,
    'padding': 16
  }
}}%%
flowchart LR
    %% Cấp 0: Root Node
    ROOT["<b>[0] QUẢN LÝ CHECKLIST<br/>CÔNG VIỆC TIMEKEEPER</b>"]

    %% Cấp 1: Các Phân hệ lớn (Động từ + Danh từ)
    M1["<b>[M1] PHÂN HỆ 1</b><br/>Xác thực Quyền hạn<br/>Tích hợp"]
    M2["<b>[M2] PHÂN HỆ 2</b><br/>Vận hành Chấm công và<br/>Checklist Công việc"]
    M3["<b>[M3] PHÂN HỆ 3</b><br/>Giám sát và Thống kê<br/>Báo cáo Tiến độ"]

    ROOT --> M1
    ROOT --> M2
    ROOT --> M3

    %% Cấp 2: Chức năng nghiệp vụ đơn nhiệm (Động từ + Danh từ)
    F01["<b>FDD-01</b><br/>Kiểm tra nghĩa vụ nộp<br/>báo cáo Checklist"]
    F02["<b>FDD-02</b><br/>Kiểm tra quyền xem<br/>báo cáo Checklist"]
    M1 --> F01
    M1 --> F02

    F03["<b>FDD-03</b><br/>Tra cứu công việc<br/>chưa hoàn thành trước đó"]
    F04["<b>FDD-04</b><br/>Thực hiện Check-in và<br/>lập Checklist ngày"]
    F05["<b>FDD-05</b><br/>Thực hiện Check-out và<br/>cập nhật tiến độ"]
    F06["<b>FDD-06</b><br/>Tra cứu lịch sử<br/>Checklist cá nhân"]
    M2 --> F03
    M2 --> F04
    M2 --> F05
    M2 --> F06

    F07["<b>FDD-07</b><br/>Giám sát tiến độ<br/>công việc trong ngày"]
    F08["<b>FDD-08</b><br/>Thống kê báo cáo<br/>tiến độ tuần/tháng"]
    M3 --> F07
    M3 --> F08

    %% Cấp 3: Tác vụ chi tiết - Nhịp IPO (Động từ + Danh từ)
    F01 --> IPO01["• <b>Input:</b> Tiếp nhận thông tin định danh nhân viên<br/>• <b>Process:</b> Gọi API gốc kiểm tra nghĩa vụ nộp báo cáo<br/>• <b>Output:</b> Trả kết quả cờ bắt buộc nộp Checklist"]
    F02 --> IPO02["• <b>Input:</b> Tiếp nhận thông tin định danh người quản lý<br/>• <b>Process:</b> Gọi API gốc kiểm tra quyền xem báo cáo<br/>• <b>Output:</b> Trả kết quả xác nhận quyền xem báo cáo"]
    F03 --> IPO03["• <b>Input:</b> Gửi yêu cầu hoặc từ khóa tìm việc cũ chưa xong<br/>• <b>Process:</b> Lọc các việc chưa DONE từ những ngày trước<br/>• <b>Output:</b> Trả danh sách việc chưa hoàn thành để chọn làm tiếp"]
    F04 --> IPO04["• <b>Input:</b> Gửi yêu cầu Check-in và chọn việc cũ/tạo việc mới<br/>• <b>Process:</b> Gọi API chấm công ngay, check nghĩa vụ và tạo việc mới<br/>• <b>Output:</b> Trả kết quả Check-in và lưu bản ghi daily_checklist"]
    F05 --> IPO05["• <b>Input:</b> Gửi yêu cầu Check-out, nhập progress và status<br/>• <b>Process:</b> Gọi API chấm công ngay và cập nhật tiến độ check_list<br/>• <b>Output:</b> Trả kết quả Check-out, list việc ngày và lưu tiến độ"]
    F06 --> IPO06["• <b>Input:</b> Chọn khoảng thời gian tra cứu lịch sử<br/>• <b>Process:</b> Truy vấn danh sách daily_checklist của cá nhân<br/>• <b>Output:</b> Trả lịch sử công việc và tiến độ % cá nhân"]
    F07 --> IPO07["• <b>Input:</b> Chọn bộ lọc ngày, chi nhánh hoặc nhân viên<br/>• <b>Process:</b> Kiểm tra quyền xem qua API và lọc Checklist ngày<br/>• <b>Output:</b> Trả danh sách công việc và tiến độ trong ngày"]
    F08 --> IPO08["• <b>Input:</b> Chọn chu kỳ tuần/tháng, chi nhánh hoặc nhân viên<br/>• <b>Process:</b> Kiểm tra quyền xem qua API và tính toán thống kê<br/>• <b>Output:</b> Trả báo cáo tổng hợp khối lượng và tỷ lệ hoàn thành"]

    %% Cố định kích thước khung và độ rộng 4 cột
    classDef rootNode fill:#1e1b4b,stroke:#818cf8,stroke-width:2px,color:#ffffff,width:210px;
    classDef moduleNode fill:#312e81,stroke:#6366f1,stroke-width:1.5px,color:#ffffff,width:240px;
    classDef funcNode fill:#1f2937,stroke:#10b981,stroke-width:1.5px,color:#f4f4f5,width:220px;
    classDef ipoNode fill:#27272a,stroke:#52525b,stroke-width:1px,color:#e4e4e7,text-align:left,white-space:nowrap;

    class ROOT rootNode;
    class M1,M2,M3 moduleNode;
    class F01,F02,F03,F04,F05,F06,F07,F08 funcNode;
    class IPO01,IPO02,IPO03,IPO04,IPO05,IPO06,IPO07,IPO08 ipoNode;
```

---

## 3. Bảng Đặc tả Chi tiết Tác vụ Cấp 3 theo Nhịp IPO (`Input – Process – Output`)

| Mã FDD | Tên Chức năng Cấp 2 | Nhịp Input (Đầu vào) | Nhịp Process (Xử lý Nghiệp vụ) | Nhịp Output (Đầu ra) |
|:---:|---|---|---|---|
| **`FDD-01`** | **Kiểm tra nghĩa vụ nộp báo cáo Checklist** | Tiếp nhận thông tin định danh nhân viên (`employee_id`, token) khi chấm công. | Gọi sang API của Hệ thống gốc SC24h (`[Hệ thống gốc - 02]`) để kiểm tra xem nhân viên đó có thuộc diện bắt buộc nộp báo cáo Checklist khi Check-in / Check-out hay không. | Trả về cờ `require_checklist` (`true / false`). |
| **`FDD-02`** | **Kiểm tra quyền xem báo cáo Checklist** | Tiếp nhận thông tin định danh người dùng (`employee_id`, token) khi truy cập các chức năng báo cáo. | Gọi sang API của Hệ thống gốc SC24h (`[Hệ thống gốc - 03]`) để kiểm tra xem người dùng có quyền Quản lý được xem báo cáo Checklist hay không. | Trả về cờ `can_view_report` (`true / false`). |
| **`FDD-03`** | **Tra cứu công việc chưa hoàn thành trước đó** | Nhận yêu cầu lấy gợi ý hoặc từ khóa tìm kiếm công việc của nhân viên (`employee_id`) khi chuẩn bị điền Checklist đầu ngày. | Truy vấn trên `daily_checklist` của nhân viên từ những ngày trước để lọc các phần tử trong mảng `check_list` có trạng thái chưa hoàn thành (`TODO`, `PROGRESS`, `HOLD`). | Trả về danh sách các đầu việc chưa xong trước đó để nhân viên chọn tiếp tục làm (không phải gõ lại). |
| **`FDD-04`** | **Thực hiện Check-in và lập Checklist ngày** | **Pha 1:** Nhận yêu cầu Check-in đầu ngày từ nhân viên.<br/>**Pha 2:** Nhận danh sách các đầu việc (`description`) được chọn tiếp tục từ việc cũ hoặc nhập mới cho ngày hôm nay. | **Pha 1:** Gọi **ngay lập tức** API chấm công gốc (`[Hệ thống gốc - 01]`) để chốt thời gian Check-in $\to$ Gọi `FDD-01` kiểm tra nghĩa vụ nộp báo cáo.<br/>**Pha 2 (Độc lập):** Khởi tạo document mới trong `daily_checklist` (nếu chọn tiếp tục việc cũ thì tạo phần tử mới y hệt cho ngày hôm nay với `progress = 0`, `status = TODO`). | **Pha 1:** Trả kết quả Check-in tức thời, cờ `require_checklist` và danh sách gợi ý việc chưa xong (`FDD-03`).<br/>**Pha 2:** Lưu danh sách công việc ngày vào `daily_checklist`. |
| **`FDD-05`** | **Thực hiện Check-out và cập nhật tiến độ** | **Pha 1:** Nhận yêu cầu Check-out cuối ngày từ nhân viên.<br/>**Pha 2:** Nhận dữ liệu cập nhật `progress` (`0-100`) và `status` (`TODO, PROGRESS, DONE, HOLD, CANCELLED`) cho từng phần tử trong `check_list`. | **Pha 1:** Gọi **ngay lập tức** API chấm công gốc (`[Hệ thống gốc - 01]`) để chốt thời gian Check-out $\to$ Gọi `FDD-01` kiểm tra nghĩa vụ nộp báo cáo và truy vấn bản ghi `daily_checklist` hôm nay.<br/>**Pha 2 (Độc lập):** Kiểm tra tính hợp lệ (`progress` từ `0` đến `100`, `status` hợp lệ) và cập nhật mảng `check_list` của ngày hôm đó. | **Pha 1:** Trả kết quả Check-out tức thời, cờ `require_checklist` và danh sách công việc ngày hôm nay (nếu có).<br/>**Pha 2:** Cập nhật snapshot tiến độ vào `daily_checklist` và trả kết quả xác nhận. |
| **`FDD-06`** | **Tra cứu lịch sử Checklist cá nhân** | Chọn khoảng thời gian tra cứu (`fromDate`, `toDate`, tuần/tháng, phân trang) của cá nhân nhân viên. | Lọc các bản ghi trong `daily_checklist` theo `employee_id` của chính nhân viên đang đăng nhập và khoảng thời gian `date` yêu cầu. | Trả về danh sách lịch sử công việc và % tiến độ hoàn thành theo từng ngày của bản thân. |
| **`FDD-07`** | **Giám sát tiến độ công việc trong ngày** | Chọn ngày cần xem (`date`), lọc theo mã chi nhánh (`branch_id`) hoặc mã nhân viên (`employee_id`). | Gọi `FDD-02` kiểm tra quyền xem báo cáo qua API gốc (`can_view_report = true`) $\to$ Truy vấn danh sách `daily_checklist` và tiến độ các đầu việc trong ngày của cá nhân / chi nhánh. | Trả về danh sách công việc đang làm trong ngày và tiến độ % của nhân sự / các nhân sự trong chi nhánh. |
| **`FDD-08`** | **Thống kê báo cáo tiến độ tuần/tháng** | Chọn chu kỳ thống kê (tuần / tháng) và bộ lọc mã chi nhánh (`branch_id`), mã nhân viên (`employee_id`). | Gọi `FDD-02` kiểm tra quyền xem báo cáo qua API gốc (`can_view_report = true`) $\to$ Truy vấn và Aggregation trực tiếp trên `daily_checklist` để tính tổng số đầu việc, tỷ lệ theo `status` và `progress` trung bình. | Trả về số liệu tổng hợp khối lượng công việc, tỷ lệ hoàn thành và tiến độ theo tuần/tháng của cá nhân / chi nhánh. |

---

## 4. Ma trận Truy vết RBAC & Phân tách Trách nhiệm (SoD)

> **Nguyên tắc Phân quyền & Kiểm soát Truy cập:**  
> - **Nhân viên (`ACT-01`):** Chỉ thao tác trên dữ liệu Checklist của chính mình (`Scope: OWN`). Nghĩa vụ phải nộp báo cáo khi Check-in / Check-out được xác định qua `FDD-01` (`require_checklist = true`).  
> - **Quản lý (`ACT-02`):** Xem danh sách công việc trong ngày và báo cáo thống kê tiến độ tuần/tháng của cá nhân / chi nhánh ở chế độ *Read-only* khi được xác thực qua `FDD-02` (`can_view_report = true`).

| STT | Mã FDD | Tên chức năng Cấp 2 (`Động từ + Danh từ`) | Nguồn Hành động / Tác vụ (Bước 2 & 3) | Bảng DB (Bước 5) | Nhân viên (`ACT-01`) | Quản lý (`ACT-02`) | Hệ thống gốc (`EXT-01`) |
|:---:|:---:|---|---|---|---|---|---|
| 1 | **`FDD-01`** | Kiểm tra nghĩa vụ nộp báo cáo Checklist | `[Hệ thống gốc - 02]` | *(Không lưu DB - Gọi API gốc)* | **Trigger Check** | - | **Check Obligation API** |
| 2 | **`FDD-02`** | Kiểm tra quyền xem báo cáo Checklist | `[Hệ thống gốc - 03]` | *(Không lưu DB - Gọi API gốc)* | - | **Trigger Check** | **Check Permission API** |
| 3 | **`FDD-03`** | Tra cứu công việc chưa hoàn thành trước đó | `[Nhân viên - 01]` | `daily_checklist` | **View** (`OWN`) | - | - |
| 4 | **`FDD-04`** | Thực hiện Check-in và lập Checklist ngày | `[Nhân viên - 02]`, `[Hệ thống gốc - 01]` | `daily_checklist` | **Create / View** (`OWN`) | - | **Attendance API** |
| 5 | **`FDD-05`** | Thực hiện Check-out và cập nhật tiến độ | `[Nhân viên - 03]`, `[Hệ thống gốc - 01]` | `daily_checklist` | **Update / View** (`OWN`) | - | **Attendance API** |
| 6 | **`FDD-06`** | Tra cứu lịch sử Checklist cá nhân | `[Nhân viên - 04]` | `daily_checklist` | **View** (`OWN`) | - | - |
| 7 | **`FDD-07`** | Giám sát tiến độ công việc trong ngày | `[Quản lý - 01]` | `daily_checklist` | - | **View** (`can_view_report = true`) | - |
| 8 | **`FDD-08`** | Thống kê báo cáo tiến độ tuần/tháng | `[Quản lý - 02]` | `daily_checklist` | - | **View** (`can_view_report = true`) | - |
