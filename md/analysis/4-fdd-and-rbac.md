# BƯỚC 4: SƠ ĐỒ PHÂN RÃ CHỨC NĂNG (FDD 3 CẤP) & MA TRẬN RBAC
> **Dự án:** Project TimeKeeper (Module Quản lý Checklist Công việc tích hợp Chấm công)  
> **Đầu vào:** [`md/analysis/2-actor-identification.md`](./2-actor-identification.md), [`md/analysis/3-dfd-context-diagram.md`](./3-dfd-context-diagram.md)  
> **Đầu ra phục vụ:** Bước 5 (Ánh xạ FDD sang Database), Bước 6 (Thiết kế RESTful API)

---

## 1. Kiểm chứng Nguyên tắc MECE & Công thức Bảo toàn Chức năng Cấp 2

### 1.1. Công thức Bảo toàn Chức năng Cấp 2
$$\text{Tổng Chức năng Cấp 2} = \text{Tổng Hành động Actor (Bước 3)} - \text{Số Hành động trùng/hợp nhất} + \text{Số Tác vụ tự động (Bước 2)}$$

* **Tổng Hành động Actor ở Bước 3:** `10` hành động (`[Nhân viên - 01..04]`, `[Ban Giám đốc - 01..02]`, `[Quản lý phòng ban - 01..02]`, `[Hệ thống gốc - 01..02]`).
* **Số Hành động hợp nhất theo nguyên tắc MECE:** `3` điểm hợp nhất:
  1. `[Ban Giám đốc - 01]` + `[Quản lý phòng ban - 01]` $\longrightarrow$ Hợp nhất thành **`FDD-06: Giám sát tiến độ công việc trong ngày`** *(phân biệt phạm vi dữ liệu qua API Hệ thống gốc)*.
  2. `[Ban Giám đốc - 02]` + `[Quản lý phòng ban - 02]` $\longrightarrow$ Hợp nhất thành **`FDD-07: Thống kê báo cáo tiến độ tuần/tháng`** *(phân biệt phạm vi dữ liệu qua API Hệ thống gốc)*.
  3. `[Hệ thống gốc - 01]` *(Xử lý Check-in / Check-out tức thời)* $\longrightarrow$ Hợp nhất vào nhịp gọi API chấm công tức thời bên trong **`FDD-03` (Check-in)** và **`FDD-04` (Check-out)**.
* **Số Tác vụ tự động ở Bước 2:** `1` tác vụ (`JOB-01: Cron Job Tổng hợp Báo cáo Cuối ngày` $\longrightarrow$ **`FDD-08`**).

$$\Longrightarrow \text{Tổng Chức năng Cấp 2} = 10 - 3 + 1 = \mathbf{8\text{ Chức năng Cấp 2 } (\text{FDD-01} \to \text{FDD-08})}$$

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
    M1["<b>[M1] PHÂN HỆ 1</b><br/>Xác thực & Kiểm tra<br/>Quyền hạn Tích hợp"]
    M2["<b>[M2] PHÂN HỆ 2</b><br/>Vận hành Chấm công &<br/>Checklist Công việc"]
    M3["<b>[M3] PHÂN HỆ 3</b><br/>Giám sát & Thống kê<br/>Báo cáo Tiến độ"]

    ROOT --> M1 & M2 & M3

    %% Cấp 2: Chức năng nghiệp vụ đơn nhiệm (Động từ + Danh từ)
    F01["<b>FDD-01</b><br/>Kiểm tra nghĩa vụ nộp &<br/>quyền xem báo cáo"]
    M1 --> F01

    F02["<b>FDD-02</b><br/>Quản lý công việc<br/>mẫu cá nhân"]
    F03["<b>FDD-03</b><br/>Thực hiện Check-in &<br/>lập checklist ngày"]
    F04["<b>FDD-04</b><br/>Thực hiện Check-out &<br/>cập nhật tiến độ"]
    F05["<b>FDD-05</b><br/>Tra cứu lịch sử<br/>checklist cá nhân"]
    M2 --> F02 & F03 & F04 & F05

    F06["<b>FDD-06</b><br/>Giám sát tiến độ<br/>công việc trong ngày"]
    F07["<b>FDD-07</b><br/>Thống kê báo cáo<br/>tiến độ tuần/tháng"]
    F08["<b>FDD-08</b><br/>Tổng hợp số liệu<br/>tiến độ cuối ngày"]
    M3 --> F06 & F07 & F08

    %% Cấp 3: Tác vụ chi tiết - Nhịp IPO (Động từ + Danh từ)
    F01 --> IPO01["• <b>Input:</b> Tiếp nhận thông tin định danh người dùng<br/>• <b>Process:</b> Gọi API gốc kiểm tra nghĩa vụ nộp & quyền xem<br/>• <b>Output:</b> Trả kết quả cờ bắt buộc nộp & phạm vi xem"]
    F02 --> IPO02["• <b>Input:</b> Nhập thông tin công việc mẫu cá nhân<br/>• <b>Process:</b> Kiểm tra dữ liệu & xử lý thêm/sửa/xóa mẫu<br/>• <b>Output:</b> Lưu bản ghi mẫu & trả danh sách Template Task"]
    F03 --> IPO03["• <b>Input:</b> Gửi yêu cầu Check-in & chọn/tạo task ngày<br/>• <b>Process:</b> Gọi API chấm công ngay, check nghĩa vụ & xử lý task<br/>• <b>Output:</b> Trả kết quả Check-in, list mẫu & lưu Checklist ngày"]
    F04 --> IPO04["• <b>Input:</b> Gửi yêu cầu Check-out & nhập % tiến độ task<br/>• <b>Process:</b> Gọi API chấm công ngay & tính trạng thái tiến độ<br/>• <b>Output:</b> Trả kết quả Check-out, list task ngày & lưu tiến độ"]
    F05 --> IPO05["• <b>Input:</b> Chọn khoảng thời gian tra cứu lịch sử<br/>• <b>Process:</b> Truy vấn danh sách checklist của cá nhân<br/>• <b>Output:</b> Trả lịch sử công việc & % tiến độ cá nhân"]
    F06 --> IPO06["• <b>Input:</b> Chọn bộ lọc ngày, phòng ban & nhân sự<br/>• <b>Process:</b> Kiểm tra quyền xem qua API & lọc checklist ngày<br/>• <b>Output:</b> Trả danh sách công việc & tiến độ trong ngày"]
    F07 --> IPO07["• <b>Input:</b> Chọn chu kỳ tuần/tháng & bộ lọc phòng ban<br/>• <b>Process:</b> Kiểm tra quyền xem qua API & tính toán thống kê<br/>• <b>Output:</b> Trả báo cáo tổng hợp khối lượng & tỷ lệ hoàn thành"]
    F08 --> IPO08["• <b>Input:</b> Kích hoạt lịch chạy tự động cuối ngày<br/>• <b>Process:</b> Quét checklist trong ngày & tính chỉ số tổng hợp<br/>• <b>Output:</b> Lưu bản ghi tổng hợp báo cáo ngày vào Database"]

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
| **`FDD-01`** | **Kiểm tra nghĩa vụ nộp & quyền xem báo cáo** | Tiếp nhận token / thông tin định danh `user_id`, `group_id` từ request. | Gọi sang API của Hệ thống gốc SC24h để kiểm tra: (1) User có thuộc diện bắt buộc nộp báo cáo Checklist khi chấm công không; (2) User có quyền xem báo cáo Checklist Task và phạm vi được xem là gì. | Trả về cờ `require_checklist` (`true/false`) hoặc quyền xem báo cáo (`can_view_report`, danh sách phòng ban/nhân sự được phép xem). |
| **`FDD-02`** | **Quản lý công việc mẫu cá nhân** | Nhập thông tin Template Task (tiêu đề công việc mẫu, mô tả, thứ tự ưu tiên) hoặc yêu cầu sửa/xóa/xem. | Kiểm tra tính hợp lệ dữ liệu (`sanitizeContent`), xác thực quyền sở hữu cá nhân (`user_id`) và thực thi thao tác thêm/sửa/xóa. | Lưu cập nhật vào bảng `task_templates` và trả về danh sách Template Task cá nhân của nhân viên. |
| **`FDD-03`** | **Thực hiện Check-in & lập checklist ngày** | **Pha 1:** Nhận yêu cầu Check-in từ User.<br/>**Pha 2:** Nhận danh sách công việc chọn từ Template Task hoặc tạo task mới cho ngày hôm nay. | **Pha 1:** Gọi **ngay lập tức** API chấm công gốc để chốt thời gian Check-in $\to$ Gọi `FDD-01` kiểm tra nghĩa vụ nộp báo cáo.<br/>**Pha 2 (Độc lập):** Kiểm tra dữ liệu các đầu việc được chọn/tạo mới và khởi tạo bản ghi Checklist ngày. | **Pha 1:** Trả kết quả Check-in tức thời + cờ `require_checklist` + danh sách Template Task cá nhân.<br/>**Pha 2:** Lưu danh sách công việc ngày vào `daily_checklists` (trạng thái khởi tạo `0%`). |
| **`FDD-04`** | **Thực hiện Check-out & cập nhật tiến độ** | **Pha 1:** Nhận yêu cầu Check-out từ User.<br/>**Pha 2:** Nhận dữ liệu cập nhật `% tiến độ (0-100%)` hoặc trạng thái `chưa làm` cho từng task trong ngày. | **Pha 1:** Gọi **ngay lập tức** API chấm công gốc để chốt thời gian Check-out $\to$ Gọi `FDD-01` kiểm tra nghĩa vụ nộp báo cáo & truy vấn list task hôm nay.<br/>**Pha 2 (Độc lập):** Kiểm tra giá trị `% tiến độ`, tính toán trạng thái từng task và tỷ lệ hoàn thành chung của ngày. | **Pha 1:** Trả kết quả Check-out tức thời + cờ `require_checklist` + danh sách công việc ngày hôm nay.<br/>**Pha 2:** Cập nhật tiến độ vào `daily_checklists` và trả kết quả xác nhận. |
| **`FDD-05`** | **Tra cứu lịch sử checklist cá nhân** | Chọn khoảng thời gian tra cứu (`fromDate`, `toDate`, phân trang) của cá nhân nhân viên. | Lọc các bản ghi checklist theo `user_id` của chính nhân viên đang đăng nhập và khoảng thời gian yêu cầu. | Trả về danh sách lịch sử công việc và % tiến độ hoàn thành theo từng ngày của bản thân. |
| **`FDD-06`** | **Giám sát tiến độ công việc trong ngày** | Chọn ngày cần xem, lọc theo mã phòng ban (`group_id`), mã nhân viên hoặc trạng thái tiến độ. | Gọi `FDD-01` kiểm tra quyền và phạm vi xem báo cáo qua API gốc $\to$ Truy vấn danh sách checklist và tiến độ các đầu việc trong ngày theo đúng phạm vi được phép. | Trả về danh sách nhân sự, các công việc đang làm trong ngày và % tiến độ chi tiết. |
| **`FDD-07`** | **Thống kê báo cáo tiến độ tuần/tháng** | Chọn chu kỳ thống kê (tuần / tháng / khoảng ngày) và bộ lọc phòng ban / nhân sự. | Gọi `FDD-01` kiểm tra quyền xem báo cáo qua API gốc $\to$ Truy vấn dữ liệu tổng hợp từ `daily_report_summaries` & `daily_checklists` để tính tổng số việc, việc hoàn thành, việc dở dang/chưa làm và % trung bình. | Trả về số liệu thống kê tổng quan và chi tiết tiến độ công việc theo tuần/tháng. |
| **`FDD-08`** | **Tổng hợp số liệu tiến độ cuối ngày** | Kích hoạt tự động theo lịch Cron Job vào cuối mỗi ngày (`JOB-01`). | Quét toàn bộ dữ liệu `daily_checklists` phát sinh trong ngày, thống kê số lượng task theo trạng thái và tính điểm/tỷ lệ % tiến độ trung bình theo từng nhân viên & phòng ban. | Lưu/cập nhật bản ghi tổng hợp vào `daily_report_summaries` để tối ưu tốc độ truy vấn báo cáo tuần/tháng. |

---

## 4. Ma trận Truy vết RBAC & Phân tách Trách nhiệm (SoD)

> **Nguyên tắc Phân quyền & Kiểm soát Truy cập:**  
> - **Nhân viên (`ACT-01`):** Chỉ thao tác trên dữ liệu Template Task và Checklist của chính mình (`Scope: OWN`). Nghĩa vụ phải nộp báo cáo khi Check-in/Check-out được quyết định qua API Hệ thống gốc (`FDD-01`).  
> - **Ban Giám đốc (`ACT-02`) & Quản lý phòng ban (`ACT-03`):** Chỉ có quyền xem báo cáo (*Read-only*). Quyền được phép truy cập báo cáo và phạm vi dữ liệu (`ALL` toàn công ty hay `DEPARTMENT` nội bộ phòng) được xác thực qua API Hệ thống gốc (`FDD-01`).

| STT | Mã FDD | Tên chức năng Cấp 2 (`Động từ + Danh từ`) | Nguồn Hành động / Tác vụ (Bước 2 & 3) | Bảng DB dự kiến (Bước 5) | Nhân viên (`ACT-01`) | Ban Giám đốc (`ACT-02`) | Quản lý phòng (`ACT-03`) | Hệ thống gốc (`EXT-01`) | Hệ thống (`Cron`) |
|:---:|:---:|---|---|---|---|---|---|---|---|
| 1 | **`FDD-01`** | Kiểm tra nghĩa vụ nộp & quyền xem báo cáo | `[Hệ thống gốc - 02]` | *(Không lưu DB - Gọi API gốc)* | **Trigger Check** | **Trigger Check** | **Trigger Check** | **Permission API** | - |
| 2 | **`FDD-02`** | Quản lý công việc mẫu cá nhân | `[Nhân viên - 01]` | `task_templates` | **CRUD** (`OWN`) | - | - | - | - |
| 3 | **`FDD-03`** | Thực hiện Check-in & lập checklist ngày | `[Nhân viên - 02]`, `[Hệ thống gốc - 01]` | `daily_checklists`, `task_templates` | **Create / View** (`OWN`) | - | - | **Attendance API** | - |
| 4 | **`FDD-04`** | Thực hiện Check-out & cập nhật tiến độ | `[Nhân viên - 03]`, `[Hệ thống gốc - 01]` | `daily_checklists` | **Update / View** (`OWN`) | - | - | **Attendance API** | - |
| 5 | **`FDD-05`** | Tra cứu lịch sử checklist cá nhân | `[Nhân viên - 04]` | `daily_checklists` | **View** (`OWN`) | - | - | - | - |
| 6 | **`FDD-06`** | Giám sát tiến độ công việc trong ngày | `[Ban Giám đốc - 01]`, `[Quản lý phòng ban - 01]` | `daily_checklists` | - | **View** (`API Scope: ALL`) | **View** (`API Scope: DEPT`) | **Verify Scope** | - |
| 7 | **`FDD-07`** | Thống kê báo cáo tiến độ tuần/tháng | `[Ban Giám đốc - 02]`, `[Quản lý phòng ban - 02]` | `daily_report_summaries`, `daily_checklists` | - | **View** (`API Scope: ALL`) | **View** (`API Scope: DEPT`) | **Verify Scope** | - |
| 8 | **`FDD-08`** | Tổng hợp số liệu tiến độ cuối ngày | `JOB-01` (Bước 2) | `daily_checklists`, `daily_report_summaries` | - | - | - | - | **Auto Execute** (`CRON`) |

