# BƯỚC 1: KHẢO SÁT & HIỂU BÀI TOÁN (PROBLEM SURVEY)
> **Dự án:** Project TimeKeeper (Module Quản lý Checklist Công việc tích hợp Chấm công)  
> **Đơn vị áp dụng:** Công ty SC24h  
> **Phạm vi kỹ thuật:** Xây dựng hệ thống Backend API (NestJS 11 + MongoDB). Phần Frontend do dev khác đảm nhiệm.

---

## 1. Mô hình Chuyển hóa Yêu cầu Hệ thống

$$\text{Pain Points (Nỗi đau thực tại)} \longrightarrow \text{Business Goals (Mục tiêu hệ thống)} \longrightarrow \text{System Boundary (Ranh giới hệ thống)} \longrightarrow \text{Acceptance Criteria (Tiêu chí nghiệm thu)}$$

---

## 2. Năm Câu hỏi Khảo sát & Hiểu Bài toán Gốc

### Câu 1: Tại sao cần xây dựng hệ thống này?
Xây dựng hệ thống **Backend API (Module Checklist ghi chú công việc hàng ngày tích hợp với hệ thống chấm công & phân quyền hiện hữu của SC24h)** nhằm giải quyết 3 mục tiêu cốt lõi:
1. **Minh bạch hóa công việc và tiến độ hàng ngày:** Gắn kết trực tiếp việc lập danh sách công việc cần làm với sự kiện Check-in và việc cập nhật tiến độ hoàn thành với sự kiện Check-out của nhân viên (đối với nhân sự thuộc diện phải nộp báo cáo theo kết quả kiểm tra từ API hệ thống gốc).
2. **Kế thừa nhanh công việc chưa hoàn thành:** Cho phép nhân viên khi Check-in có thể tìm kiếm / chọn lại các công việc chưa xong từ những ngày trước để tiếp tục làm (tạo bản ghi mới cho ngày hôm nay) mà không phải gõ lại từ đầu.
3. **Cung cấp bức tranh tổng thể theo chu kỳ cho Quản lý:** Giúp cấp Quản lý nắm bắt toàn diện công việc và tiến độ của nhân sự trong ngày, trong tuần và trong tháng thông qua dữ liệu tập trung (quyền xem báo cáo được xác thực qua API hệ thống gốc).

---

### Câu 2: Hiện tại quy trình đang làm như thế nào? (As-Is)
1. **Chấm công tách rời ghi chú công việc ngày:** Nhân viên thực hiện Check-in / Check-out trên hệ thống chấm công sẵn có của công ty nhưng luồng này hoàn toàn độc lập, không gắn với kế hoạch công việc trong ngày.
2. **Trao đổi công việc hàng ngày rời rạc:** Các việc làm hàng ngày của nhân sự phòng DEV đang được trao đổi phân tán (qua tin nhắn chat, họp miệng hoặc ghi chú cá nhân).
3. **Thiếu dữ liệu báo cáo tập trung:** Khi cấp Quản lý cần nắm bắt nhân viên hôm nay làm gì hoặc tổng kết công việc theo tuần/tháng thì phải hỏi thủ công và không có hệ thống dữ liệu thống kê tập trung.

---

### Câu 3: Vấn đề (Nỗi đau) hiện tại là gì?

| STT | Vấn đề hiện tại (Pain Points) | Hệ quả (Consequences) | Mục tiêu hệ thống tương ứng (Business Goals) |
|:---:|---|---|---|
| 1 | Cấp Quản lý không biết nhân viên trong phòng ban hôm nay làm gì. | Thiếu thông tin giám sát trong ngày, khó nắm bắt trọng tâm công việc của từng nhân sự. | **G1:** Khi nhân viên thực hiện Check-in (nếu API hệ thống gốc xác nhận thuộc diện phải nộp checklist), hệ thống gợi ý các công việc chưa xong từ ngày trước để chọn làm tiếp hoặc cho phép tạo mới danh sách công việc cần làm trong ngày. |
| 2 | Không biết tiến độ công việc của nhân viên như thế nào. | Không kiểm soát được đầu việc nào đã hoàn thành, đang làm dở hay chưa thực hiện vào cuối ngày. | **G2:** Khi Check-out, hiển thị danh sách công việc ngày hôm nay để nhân viên cập nhật % tiến độ hoàn thành (`0% - 100%`) và trạng thái công việc. |
| 3 | Không nắm bắt được tuần này / tháng này phòng ban đang làm gì. | Thiếu bức tranh tổng thể để đánh giá khối lượng và hiệu quả công việc của nhân sự theo chu kỳ tuần/tháng. | **G3:** Cung cấp các API truy vấn và thống kê công việc, tiến độ theo ngày, tuần, tháng cho cấp Quản lý (quyền xem được xác thực qua API hệ thống gốc). |

---

### Câu 4: Ai là người tham gia sử dụng?

| Nhóm đối tượng (thô) | Căn cứ trong đề bài & khảo sát | Nhu cầu chính khi sử dụng hệ thống |
|---|---|---|
| **1. Nhân viên (Employee)** *(thuộc diện phải nộp checklist theo API hệ thống gốc, trước mắt là phòng DEV)* | *"Mỗi khi nhân viên chấm công, nếu là nhân viên của phòng Dev sẽ liệt kê các công việc cần làm trong ngày. Khi checkout thì sẽ hiển thị list công việc ngày hôm nay để nhân viên cập nhập tiến độ % công việc hoặc chưa làm."* | - Xem gợi ý các công việc chưa xong trước đó để chọn làm tiếp hoặc nhập công việc mới khi Check-in.<br>- Cập nhật % tiến độ hoàn thành (`0% - 100%`) và trạng thái khi Check-out.<br>- Xem lịch sử checklist cá nhân. |
| **2. Quản lý (Manager)** *(gộp chung Ban Giám đốc & Quản lý các cấp)* | *"Không nắm bắt được: nhân viên trong phòng ban hôm nay làm gì, tiến độ công việc như thế nào, tuần này/tháng này đang làm gì"* | - Xem danh sách công việc và tiến độ trong ngày của cá nhân / phòng ban.<br>- Xem báo cáo thống kê tổng hợp công việc theo tuần, tháng (*Read-only*). |

---

### Câu 5: Kết quả mong muốn là gì? (Acceptance Criteria)

| Mục tiêu hệ thống | Tiêu chí đạt cụ thể (Kiểm chứng được qua API) |
|---|---|
| **G1: Tích hợp Chấm công tức thời & Kiểm tra điều kiện Checklist qua API** | - Khi User bấm Check-in / Check-out, Backend TimeKeeper gọi ngay API chấm công gốc để không bị lệch thời gian, đồng thời kiểm tra qua API gốc xem User có phải nộp báo cáo checklist hay không.<br>- Việc tạo/chọn task hay cập nhật tiến độ diễn ra độc lập trên TimeKeeper, không làm ảnh hưởng tới hệ thống chấm công gốc. |
| **G2: Gợi ý việc chưa xong & Luồng Check-in lập Checklist ngày** | - Khi Check-in (nếu thuộc diện nộp báo cáo), nhân viên có thể tìm kiếm / chọn các công việc chưa xong trước đó (tạo bản sao mới cho ngày hôm nay) hoặc thêm công việc mới vào `daily_checklist`. |
| **G3: Luồng Check-out cập nhật tiến độ** | - Khi nhân viên thực hiện Check-out, API trả về đúng danh sách công việc đã đăng ký trong ngày hôm nay.<br>- Cho phép cập nhật kết quả từng công việc theo **% tiến độ hoàn thành (0% – 100%)** và trạng thái (`TODO`, `PROGRESS`, `DONE`, `HOLD`, `CANCELLED`). |
| **G4: Báo cáo & Thống kê cho Quản lý** | - Cung cấp API truy vấn danh sách công việc và thống kê tiến độ theo **ngày hôm nay, theo tuần và theo tháng** của cá nhân / phòng ban sau khi đã xác thực quyền xem báo cáo (`can_view_report = true`) qua API của Hệ thống gốc. |

---

## 3. Bảng Tổng hợp Các Quyết định Nghiệp vụ & Kiến trúc Đã Chốt

| Mã | Hạng mục quyết định | Nội dung đã chốt | Trạng thái |
|:---:|---|---|:---:|
| **D1** | Phạm vi triển khai dự án | Chỉ xây dựng **Backend API** (NestJS 11 + MongoDB) với **01 Collection duy nhất (`daily_checklist`)** đóng vai trò ghi chú công việc hàng ngày. | `[Đã xác nhận]` |
| **D2** | Mô hình chấm công tức thời | Khi User chấm công, Backend TimeKeeper gọi **ngay lập tức** API chấm công gốc để chốt thời gian chuẩn; việc tạo task / cập nhật tiến độ là hành động độc lập của TimeKeeper. | `[Đã xác nhận]` |
| **D3** | Cơ chế kiểm tra nghĩa vụ nộp báo cáo & phân quyền | **Thông qua API của Hệ thống gốc SC24h** (TimeKeeper không tự lưu bảng cấu hình phòng ban hay tự xử lý phân quyền nội bộ). | `[Đã xác nhận]` |
| **D4** | Quyền hạn của Quản lý | Gộp chung Ban Giám đốc & Quản lý phòng ban thành đối tượng **Quản lý (Manager)**; xem báo cáo, thống kê tiến độ công việc theo ngày, tuần, tháng ở chế độ **Read-only** khi API Hệ thống gốc xác nhận có quyền xem. | `[Đã xác nhận]` |
