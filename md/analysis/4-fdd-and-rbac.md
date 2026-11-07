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
* **Số Tác vụ tự động ở Bước 2:** `1` tác vụ (`JOB-01: Cron Job Tổng hợp Báo cáo Cuối ngày` $\longrightarrow$ **`FDD-09`**).

$$\Longrightarrow \text{Tổng Chức năng Cấp 2} = 9 - 1 + 1 = \mathbf{9\text{ Chức năng Cấp 2 } (\text{FDD-01} \to \text{FDD-09})}$$

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

    F03["<b>FDD-03</b><br/>Quản lý công việc<br/>mẫu cá nhân"]
    F04["<b>FDD-04</b><br/>Thực hiện Check-in và<br/>lập Checklist ngày"]
    F05["<b>FDD-05</b><br/>Thực hiện Check-out và<br/>cập nhật tiến độ"]
    F06["<b>FDD-06</b><br/>Tra cứu lịch sử<br/>Checklist cá nhân"]
    M2 --> F03
    M2 --> F04
    M2 --> F05
    M2 --> F06

    F07["<b>FDD-07</b><br/>Giám sát tiến độ<br/>công việc trong ngày"]
    F08["<b>FDD-08</b><br/>Thống kê báo cáo<br/>tiến độ tuần/tháng"]
    F09["<b>FDD-09</b><br/>Tổng hợp số liệu<br/>tiến độ cuối ngày"]
    M3 --> F07
    M3 --> F08
    M3 --> F09

    %% Cấp 3: Tác vụ chi tiết - Nhịp IPO (Động từ + Danh từ)
    F01 --> IPO01["• <b>Input:</b> Tiếp nhận thông tin định danh nhân viên<br/>• <b>Process:</b> Gọi API gốc kiểm tra nghĩa vụ nộp báo cáo<br/>• <b>Output:</b> Trả kết quả cờ bắt buộc nộp Checklist"]
    F02 --> IPO02["• <b>Input:</b> Tiếp nhận thông tin định danh người quản lý<br/>• <b>Process:</b> Gọi API gốc kiểm tra quyền xem báo cáo<br/>• <b>Output:</b> Trả kết quả xác nhận quyền xem báo cáo"]
    F03 --> IPO03["• <b>Input:</b> Nhập thông tin công việc mẫu cá nhân<br/>• <b>Process:</b> Kiểm tra dữ liệu và xử lý thêm/sửa/xóa mẫu<br/>• <b>Output:</b> Lưu bản ghi mẫu và trả danh sách Template Task"]
    F04 --> IPO04["• <b>Input:</b> Gửi yêu cầu Check-in và chọn/tạo task ngày<br/>• <b>Process:</b> Gọi API chấm công ngay, check nghĩa vụ và xử lý task<br/>• <b>Output:</b> Trả kết quả Check-in, list mẫu và lưu Checklist ngày"]
    F05 --> IPO05["• <b>Input:</b> Gửi yêu cầu Check-out và nhập tiến độ % task<br/>• <b>Process:</b> Gọi API chấm công ngay và tính trạng thái tiến độ<br/>• <b>Output:</b> Trả kết quả Check-out, list task ngày và lưu tiến độ"]
    F06 --> IPO06["• <b>Input:</b> Chọn khoảng thời gian tra cứu lịch sử<br/>• <b>Process:</b> Truy vấn danh sách Checklist của cá nhân<br/>• <b>Output:</b> Trả lịch sử công việc và tiến độ % cá nhân"]
    F07 --> IPO07["• <b>Input:</b> Chọn bộ lọc ngày, phòng ban hoặc nhân viên<br/>• <b>Process:</b> Kiểm tra quyền xem qua API và lọc Checklist ngày<br/>• <b>Output:</b> Trả danh sách công việc và tiến độ trong ngày"]
    F08 --> IPO08["• <b>Input:</b> Chọn chu kỳ tuần/tháng, phòng ban hoặc nhân viên<br/>• <b>Process:</b> Kiểm tra quyền xem qua API và tính toán thống kê<br/>• <b>Output:</b> Trả báo cáo tổng hợp khối lượng và tỷ lệ hoàn thành"]
    F09 --> IPO09["• <b>Input:</b> Kích hoạt lịch chạy tự động cuối ngày<br/>• <b>Process:</b> Quét Checklist trong ngày và tính chỉ số tổng hợp<br/>• <b>Output:</b> Lưu bản ghi tổng hợp báo cáo ngày vào Database"]

    %% Cố định kích thước khung và độ rộng 4 cột
    classDef rootNode fill:#1e1b4b,stroke:#818cf8,stroke-width:2px,color:#ffffff,width:210px;
    classDef moduleNode fill:#312e81,stroke:#6366f1,stroke-width:1.5px,color:#ffffff,width:240px;
    classDef funcNode fill:#1f2937,stroke:#10b981,stroke-width:1.5px,color:#f4f4f5,width:220px;
    classDef ipoNode fill:#27272a,stroke:#52525b,stroke-width:1px,color:#e4e4e7,text-align:left,white-space:nowrap;

    class ROOT rootNode;
    class M1,M2,M3 moduleNode;
    class F01,F02,F03,F04,F05,F06,F07,F08,F09 funcNode;
    class IPO01,IPO02,IPO03,IPO04,IPO05,IPO06,IPO07,IPO08,IPO09 ipoNode;
```

---

## 3. Bảng Đặc tả Chi tiết Tác vụ Cấp 3 theo Nhịp IPO (`Input – Process – Output`)

| Mã FDD | Tên Chức năng Cấp 2 | Nhịp Input (Đầu vào) | Nhịp Process (Xử lý Nghiệp vụ) | Nhịp Output (Đầu ra) |
|:---:|---|---|---|---|
| **`FDD-01`** | **Kiểm tra nghĩa vụ nộp báo cáo Checklist** | Tiếp nhận thông tin định danh nhân viên (`user_id`, token) khi chấm công. | Gọi sang API của Hệ thống gốc SC24h (`[Hệ thống gốc - 02]`) để kiểm tra xem nhân viên đó có thuộc diện bắt buộc nộp báo cáo Checklist khi Check-in / Check-out hay không. | Trả về cờ `require_checklist` (`true / false`). |
| **`FDD-02`** | **Kiểm tra quyền xem báo cáo Checklist** | Tiếp nhận thông tin định danh người dùng (`user_id`, token) khi truy cập các chức năng báo cáo. | Gọi sang API của Hệ thống gốc SC24h (`[Hệ thống gốc - 03]`) để kiểm tra xem người dùng có quyền Quản lý được xem báo cáo Checklist hay không. | Trả về cờ `can_view_report` (`true / false`). |
| **`FDD-03`** | **Quản lý công việc mẫu cá nhân** | Nhập thông tin Template Task (tên công việc mẫu, mô tả) hoặc yêu cầu sửa/xóa/xem. | Kiểm tra tính hợp lệ dữ liệu (`sanitizeContent`), xác thực quyền sở hữu cá nhân (`user_id`) và thực thi thao tác thêm/sửa/xóa. | Lưu cập nhật vào bảng `task_templates` và trả về danh sách Template Task cá nhân của nhân viên. |
| **`FDD-04`** | **Thực hiện Check-in và lập Checklist ngày** | **Pha 1:** Nhận yêu cầu Check-in đầu ngày từ nhân viên.<br/>**Pha 2:** Nhận danh sách công việc chọn từ Template Task hoặc tạo task mới cho ngày hôm nay. | **Pha 1:** Gọi **ngay lập tức** API chấm công gốc (`[Hệ thống gốc - 01]`) để chốt thời gian Check-in $\to$ Gọi `FDD-01` kiểm tra nghĩa vụ nộp báo cáo.<br/>**Pha 2 (Độc lập):** Kiểm tra dữ liệu các đầu việc được chọn/tạo mới và khởi tạo bản ghi Checklist ngày. | **Pha 1:** Trả kết quả Check-in tức thời, cờ `require_checklist` và danh sách Template Task cá nhân (nếu có).<br/>**Pha 2:** Lưu danh sách công việc ngày vào `daily_checklists` (trạng thái khởi tạo `0%`). |
| **`FDD-05`** | **Thực hiện Check-out và cập nhật tiến độ** | **Pha 1:** Nhận yêu cầu Check-out cuối ngày từ nhân viên.<br/>**Pha 2:** Nhận dữ liệu cập nhật `% tiến độ (0-100%)` hoặc trạng thái `chưa làm` cho từng task trong ngày. | **Pha 1:** Gọi **ngay lập tức** API chấm công gốc (`[Hệ thống gốc - 01]`) để chốt thời gian Check-out $\to$ Gọi `FDD-01` kiểm tra nghĩa vụ nộp báo cáo và truy vấn list task hôm nay.<br/>**Pha 2 (Độc lập):** Kiểm tra giá trị `% tiến độ`, tính toán trạng thái từng task và tỷ lệ hoàn thành chung của ngày. | **Pha 1:** Trả kết quả Check-out tức thời, cờ `require_checklist` và danh sách công việc ngày hôm nay (nếu có).<br/>**Pha 2:** Cập nhật tiến độ vào `daily_checklists` và trả kết quả xác nhận. |
| **`FDD-06`** | **Tra cứu lịch sử Checklist cá nhân** | Chọn khoảng thời gian tra cứu (`fromDate`, `toDate`, tuần/tháng, phân trang) của cá nhân nhân viên. | Lọc các bản ghi Checklist theo `user_id` của chính nhân viên đang đăng nhập và khoảng thời gian yêu cầu. | Trả về danh sách lịch sử công việc và % tiến độ hoàn thành theo từng ngày của bản thân. |
| **`FDD-07`** | **Giám sát tiến độ công việc trong ngày** | Chọn ngày cần xem, lọc theo mã phòng ban hoặc mã nhân viên. | Gọi `FDD-02` kiểm tra quyền xem báo cáo qua API gốc (`can_view_report = true`) $\to$ Truy vấn danh sách Checklist và tiến độ các đầu việc trong ngày của cá nhân / phòng ban. | Trả về danh sách công việc đang làm trong ngày và tiến độ % của nhân sự / các nhân sự trong phòng ban. |
| **`FDD-08`** | **Thống kê báo cáo tiến độ tuần/tháng** | Chọn chu kỳ thống kê (tuần / tháng) và bộ lọc mã phòng ban, mã nhân viên. | Gọi `FDD-02` kiểm tra quyền xem báo cáo qua API gốc (`can_view_report = true`) $\to$ Truy vấn dữ liệu từ `daily_report_summaries` và `daily_checklists` để tính tổng số việc, tỷ lệ hoàn thành và tiến độ trung bình. | Trả về số liệu tổng hợp khối lượng công việc, tỷ lệ hoàn thành và tiến độ theo tuần/tháng của cá nhân / phòng ban. |
| **`FDD-09`** | **Tổng hợp số liệu tiến độ cuối ngày** | Kích hoạt tự động theo lịch Cron Job vào cuối mỗi ngày (`JOB-01`). | Quét toàn bộ dữ liệu `daily_checklists` phát sinh trong ngày, thống kê số lượng task theo trạng thái và tính tỷ lệ % tiến độ trung bình theo từng nhân viên và phòng ban. | Lưu/cập nhật bản ghi tổng hợp vào `daily_report_summaries` để tối ưu tốc độ truy vấn báo cáo tuần/tháng. |

---

## 4. Ma trận Truy vết RBAC & Phân tách Trách nhiệm (SoD)

> **Nguyên tắc Phân quyền & Kiểm soát Truy cập:**  
> - **Nhân viên (`ACT-01`):** Chỉ thao tác trên dữ liệu Template Task và Checklist của chính mình (`Scope: OWN`). Nghĩa vụ phải nộp báo cáo khi Check-in / Check-out được xác định qua `FDD-01` (`require_checklist = true`).  
> - **Quản lý (`ACT-02`):** Xem danh sách công việc trong ngày và báo cáo thống kê tiến độ tuần/tháng của cá nhân / phòng ban ở chế độ *Read-only* khi được xác thực qua `FDD-02` (`can_view_report = true`).

| STT | Mã FDD | Tên chức năng Cấp 2 (`Động từ + Danh từ`) | Nguồn Hành động / Tác vụ (Bước 2 & 3) | Bảng DB dự kiến (Bước 5) | Nhân viên (`ACT-01`) | Quản lý (`ACT-02`) | Hệ thống gốc (`EXT-01`) | Hệ thống (`Cron`) |
|:---:|:---:|---|---|---|---|---|---|---|
| 1 | **`FDD-01`** | Kiểm tra nghĩa vụ nộp báo cáo Checklist | `[Hệ thống gốc - 02]` | *(Không lưu DB - Gọi API gốc)* | **Trigger Check** | - | **Check Obligation API** | - |
| 2 | **`FDD-02`** | Kiểm tra quyền xem báo cáo Checklist | `[Hệ thống gốc - 03]` | *(Không lưu DB - Gọi API gốc)* | - | **Trigger Check** | **Check Permission API** | - |
| 3 | **`FDD-03`** | Quản lý công việc mẫu cá nhân | `[Nhân viên - 01]` | `task_templates` | **CRUD** (`OWN`) | - | - | - |
| 4 | **`FDD-04`** | Thực hiện Check-in và lập Checklist ngày | `[Nhân viên - 02]`, `[Hệ thống gốc - 01]` | `daily_checklists`, `task_templates` | **Create / View** (`OWN`) | - | **Attendance API** | - |
| 5 | **`FDD-05`** | Thực hiện Check-out và cập nhật tiến độ | `[Nhân viên - 03]`, `[Hệ thống gốc - 01]` | `daily_checklists` | **Update / View** (`OWN`) | - | **Attendance API** | - |
| 6 | **`FDD-06`** | Tra cứu lịch sử Checklist cá nhân | `[Nhân viên - 04]` | `daily_checklists` | **View** (`OWN`) | - | - | - |
| 7 | **`FDD-07`** | Giám sát tiến độ công việc trong ngày | `[Quản lý - 01]` | `daily_checklists` | - | **View** (`can_view_report = true`) | - | - |
| 8 | **`FDD-08`** | Thống kê báo cáo tiến độ tuần/tháng | `[Quản lý - 02]` | `daily_report_summaries`, `daily_checklists` | - | **View** (`can_view_report = true`) | - | - |
| 9 | **`FDD-09`** | Tổng hợp số liệu tiến độ cuối ngày | `JOB-01` (Bước 2) | `daily_checklists`, `daily_report_summaries` | - | - | - | **Auto Execute** (`CRON`) |
