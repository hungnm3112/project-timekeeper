# BƯỚC 3: THIẾT KẾ SƠ ĐỒ NGỮ CẢNH (DFD LEVEL 0 — CONTEXT DIAGRAM)
> **Dự án:** Project TimeKeeper (Module Quản lý Checklist Công việc tích hợp Chấm công)  
> **Đầu vào:** [`md/analysis/2-actor-identification.md`](./2-actor-identification.md)  
> **Đầu ra phục vụ:** Bước 4 (Cây chức năng FDD & RBAC), Bước 5 (Database Design)

---

## 1. Bảng Đặc tả Actor – Hành động & Luồng Dữ liệu (Bảng 5 Cột Chuẩn hóa)

Bảng dưới đây định nghĩa toàn bộ 9 tương tác vào/ra (Input/Output) giữa 3 Actor chuẩn hóa ở Bước 2 với hệ thống **TimeKeeper API (`Process 0`)**:

| Actor | Mã | Hành động | Dữ liệu Input (Actor gửi vào) | Dữ liệu Output (Actor nhận lại) |
|---|:---:|---|---|---|
| **Nhân viên**<br/>(`ACT-01`) | `[Nhân viên - 01]` | Tra cứu công việc chưa hoàn thành từ các ngày trước | Yêu cầu lấy gợi ý hoặc từ khóa tìm kiếm các đầu việc cũ chưa xong (`TODO`, `PROGRESS`, `HOLD`) | Danh sách các đầu việc chưa hoàn thành từ những ngày trước để chọn làm tiếp |
| **Nhân viên**<br/>(`ACT-01`) | `[Nhân viên - 02]` | Thực hiện Check-in và lập Checklist công việc trong ngày | 1. Yêu cầu Check-in đầu ngày<br/>2. Danh sách công việc chọn tiếp tục từ ngày trước hoặc tạo mới (`description`) | 1. Kết quả Check-in tức thời, cờ `require_checklist` và gợi ý việc cũ chưa xong<br/>2. Xác nhận lưu bản ghi `daily_checklist` trong ngày |
| **Nhân viên**<br/>(`ACT-01`) | `[Nhân viên - 03]` | Thực hiện Check-out và cập nhật tiến độ công việc | 1. Yêu cầu Check-out cuối ngày<br/>2. Dữ liệu cập nhật `progress (0-100%)` và `status` (`TODO, PROGRESS, DONE, HOLD, CANCELLED`) cho từng việc | 1. Kết quả Check-out tức thời, cờ `require_checklist` và danh sách `check_list` ngày hôm nay<br/>2. Xác nhận cập nhật tiến độ thành công |
| **Nhân viên**<br/>(`ACT-01`) | `[Nhân viên - 04]` | Xem lịch sử Checklist cá nhân | Bộ lọc thời gian (từ ngày – đến ngày, tuần/tháng) | Lịch sử danh sách công việc và tiến độ % các ngày/tuần trước của bản thân |
| **Quản lý**<br/>(`ACT-02`) | `[Quản lý - 01]` | Xem danh sách công việc và tiến độ trong ngày của cá nhân/ chi nhánh | Bộ lọc ngày (`date`), mã chi nhánh (`branch_id`)/ mã nhân viên (`employee_id`) | Danh sách công việc đang làm trong ngày và tiến độ % của nhân sự/ các nhân sự trong chi nhánh |
| **Quản lý**<br/>(`ACT-02`) | `[Quản lý - 02]` | Xem báo cáo thống kê tiến độ tuần/tháng của cá nhân/ chi nhánh | Bộ lọc chu kỳ (tuần/tháng), mã chi nhánh (`branch_id`), mã nhân viên (`employee_id`) | Số liệu tổng hợp khối lượng công việc, tỷ lệ hoàn thành và tiến độ |
| **Hệ thống gốc SC24h**<br/>(`ACT-EXT-01`) | `[Hệ thống gốc - 01]` | Xử lý chấm công Check-in / Check-out tức thời | Kết quả ghi nhận thời gian Check-in / Check-out tức thời của User | Yêu cầu gọi API Check-in / Check-out tức thời ngay khi User bấm chấm công |
| **Hệ thống gốc SC24h**<br/>(`ACT-EXT-01`) | `[Hệ thống gốc - 02]` | Kiểm tra nghĩa vụ nộp báo cáo Checklist | Kết quả kiểm tra cờ bắt buộc nộp báo cáo (`require_checklist`) của User | Yêu cầu gọi API kiểm tra User có phải nộp báo cáo Checklist khi chấm công không |
| **Hệ thống gốc SC24h**<br/>(`ACT-EXT-01`) | `[Hệ thống gốc - 03]` | Kiểm tra quyền xem báo cáo Checklist | Kết quả kiểm tra quyền xem báo cáo (`can_view_report`) của User | Yêu cầu gọi API kiểm tra User có quyền Quản lý xem báo cáo không |

> **Lưu ý kiến trúc về luồng tích hợp với Hệ thống gốc SC24h (`[Hệ thống gốc - 01/02/03]`):**  
> 1. Khi Nhân viên bấm Check-in hoặc Check-out, Backend TimeKeeper gọi **ngay lập tức** sang API chấm công của Hệ thống gốc (`[Hệ thống gốc - 01]`) để ghi nhận thời gian chuẩn xác, đồng thời gọi API kiểm tra nghĩa vụ nộp báo cáo (`[Hệ thống gốc - 02]`). Việc chọn/tạo task hoặc cập nhật % tiến độ diễn ra độc lập trên TimeKeeper.  
> 2. Khi Quản lý xem báo cáo ngày/tuần/tháng, Backend TimeKeeper gọi API kiểm tra quyền xem báo cáo (`[Hệ thống gốc - 03]`).

---

## 2. Sơ đồ Ngữ cảnh DFD Level 0 (Ký hiệu Yourdon–DeMarco)

- **Chú giải màu sắc luồng dữ liệu:**
  - <span style="color:#10b981">**Mũi tên Xanh lá (`#10b981`):**</span> Luồng dữ liệu **Input** (Actor gửi vào Hệ thống).
  - <span style="color:#f97316">**Mũi tên Cam (`#f97316`):**</span> Luồng dữ liệu **Output** (Hệ thống trả về cho Actor).
  - <span style="color:#4338ca">**Khối tròn Tím (`#4338ca`):**</span> Hệ thống trung tâm (**Process 0 — Hộp đen**).

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'fontSize': '13px'
  },
  'flowchart': {
    'nodeSpacing': 50,
    'rankSpacing': 100,
    'padding': 20
  }
}}%%
flowchart LR
    ACT01["<b>ACT-01: Nhân viên</b><br/>(Employee)"]
    ACT02["<b>ACT-02: Quản lý</b><br/>(Manager)"]

    SYS(("<b>Hệ thống TimeKeeper<br/>Checklist API</b><br/>(Process 0)"))

    EXT01["<b>ACT-EXT-01: Hệ thống gốc<br/>SC24h (External API)</b>"]

    %% ACT-01 <-> SYS (Links 0, 1)
    ACT01 -->|"01. Tìm kiếm công việc cũ chưa xong<br/>02. Check-in, chọn việc cũ/tạo việc mới<br/>03. Check-out, cập nhật progress và status<br/>04. Bộ lọc lịch sử cá nhân"| SYS
    SYS -->|"01. Danh sách việc cũ chưa hoàn thành<br/>02. Kết quả Check-in, lưu Checklist ngày<br/>03. Kết quả Check-out, lưu tiến độ<br/>04. Lịch sử Checklist cá nhân"| ACT01

    %% ACT-02 <-> SYS (Links 2, 3)
    ACT02 -->|"01. Bộ lọc ngày, chi nhánh/nhân viên<br/>02. Bộ lọc tuần/tháng, chi nhánh/nhân viên"| SYS
    SYS -->|"01. Công việc, tiến độ ngày cá nhân/chi nhánh<br/>02. Thống kê tuần/tháng cá nhân/chi nhánh"| ACT02

    %% EXT-01 <-> SYS (Links 4, 5)
    EXT01 -->|"01. Kết quả chấm công tức thời<br/>02. Kết quả cờ phải nộp Checklist<br/>03. Kết quả quyền xem báo cáo"| SYS
    SYS -->|"01. Gọi API chấm công tức thời<br/>02. Gọi API kiểm tra nghĩa vụ nộp<br/>03. Gọi API kiểm tra quyền xem"| EXT01

    %% Styling Nodes
    style SYS fill:#4338ca,stroke:#312e81,stroke-width:3px,color:#ffffff
    style ACT01 fill:#f8fafc,stroke:#334155,stroke-width:2px,color:#0f172a
    style ACT02 fill:#f8fafc,stroke:#334155,stroke-width:2px,color:#0f172a
    style EXT01 fill:#eff6ff,stroke:#1d4ed8,stroke-width:2px,color:#1e3a8a

    %% Styling Edges: Green (#10b981) for Input, Orange (#f97316) for Output
    linkStyle 0,2,4 stroke:#10b981,stroke-width:2.5px,color:#065f46
    linkStyle 1,3,5 stroke:#f97316,stroke-width:2.5px,color:#9a3412
```
