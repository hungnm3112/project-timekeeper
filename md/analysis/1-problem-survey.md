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
Xây dựng hệ thống **Backend API (Module Checklist tích hợp với hệ thống chấm công & phân quyền hiện hữu của SC24h)** nhằm giải quyết 3 mục tiêu cốt lõi:
1. **Minh bạch hóa công việc và tiến độ hàng ngày:** Gắn kết trực tiếp việc lập danh sách công việc cần làm với sự kiện Check-in và việc cập nhật tiến độ hoàn thành với sự kiện Check-out của nhân viên (đối với nhân sự thuộc diện phải nộp báo cáo theo kết quả kiểm tra từ API hệ thống gốc).
2. **Tối ưu tốc độ khai báo công việc cho nhân viên:** Cho phép nhân viên tạo sẵn danh sách công việc mẫu cá nhân (*Template Task*) để chọn nhanh hoặc tạo mới khi Check-in.
3. **Cung cấp bức tranh tổng thể theo chu kỳ cho Ban Giám đốc & Quản lý:** Giúp Ban Giám đốc và Quản lý phòng ban nắm bắt toàn diện công việc và tiến độ của nhân sự trong ngày, trong tuần và trong tháng thông qua dữ liệu tập trung (quyền xem và phạm vi dữ liệu được xác thực qua API hệ thống gốc).

---

### Câu 2: Hiện tại quy trình đang làm như thế nào? (As-Is)
1. **Chấm công tách rời quản lý công việc:** Nhân viên thực hiện Check-in / Check-out trên hệ thống chấm công sẵn có của công ty nhưng luồng này hoàn toàn độc lập, không gắn với kế hoạch công việc trong ngày.
2. **Trao đổi công việc rời rạc:** Kế hoạch làm việc hàng ngày và tiến độ thực hiện của nhân sự phòng DEV đang được trao đổi phân tán (qua tin nhắn chat, họp miệng hoặc ghi chú cá nhân).
3. **Thiếu dữ liệu báo cáo tập trung:** Khi Ban Giám đốc cần nắm bắt nhân viên đang làm gì hoặc tổng kết công việc theo tuần/tháng thì phải hỏi thủ công và không có hệ thống dữ liệu thống kê tập trung.

---

### Câu 3: Vấn đề (Nỗi đau) hiện tại là gì?

| STT | Vấn đề hiện tại (Pain Points) | Hệ quả (Consequences) | Mục tiêu hệ thống tương ứng (Business Goals) |
|:---:|---|---|---|
| 1 | Ban Giám đốc không biết nhân viên trong phòng ban hôm nay làm gì. | Thiếu thông tin giám sát trong ngày, khó nắm bắt trọng tâm công việc của từng nhân sự. | **G1:** Khi nhân viên thực hiện Check-in (nếu API hệ thống gốc xác nhận thuộc diện phải nộp checklist), hệ thống cho phép chọn từ Template Task cá nhân hoặc tạo mới danh sách công việc cần làm trong ngày. |
| 2 | Không biết tiến độ công việc của nhân viên như thế nào. | Không kiểm soát được đầu việc nào đã hoàn thành, đang làm dở hay chưa thực hiện vào cuối ngày. | **G2:** Khi Check-out, hiển thị danh sách công việc ngày hôm nay để nhân viên cập nhật % tiến độ hoàn thành (`0% - 100%`) hoặc trạng thái chưa làm. |
| 3 | Không nắm bắt được tuần này / tháng này phòng DEV đang làm gì. | Thiếu bức tranh tổng thể để đánh giá khối lượng và hiệu quả công việc của phòng ban theo chu kỳ tuần/tháng. | **G3:** Cung cấp các API truy vấn và thống kê công việc, tiến độ theo ngày, tuần, tháng cho Ban Giám đốc & Quản lý (quyền và phạm vi xem được xác thực qua API hệ thống gốc). |

---

### Câu 4: Ai là người tham gia sử dụng?

| Nhóm đối tượng (thô) | Căn cứ trong đề bài & khảo sát | Nhu cầu chính đối với hệ thống API |
|---|---|---|
| **1. Nhân viên (Employee)** *(thuộc diện phải nộp checklist theo API hệ thống gốc, trước mắt là phòng DEV)* | *"Mỗi khi nhân viên chấm công, nếu là nhân viên của phòng Dev sẽ liệt kê các công việc cần làm trong ngày. Khi checkout thì sẽ hiển thị list công việc ngày hôm nay để nhân viên cập nhập tiến độ % công việc hoặc chưa làm."* | - Quản lý bộ Template Task cá nhân.<br>- Chọn/tạo danh sách công việc cần làm trong ngày khi Check-in.<br>- Cập nhật % tiến độ hoàn thành hoặc đánh dấu chưa làm khi Check-out.<br>- Xem lịch sử checklist cá nhân. |
| **2. Ban Giám đốc (BOD)** | *"Ban giám đốc không nắm bắt được: nhân viên hôm nay làm gì, tiến độ như thế nào, tuần này/tháng này phòng DEV đang làm gì"* | - Xem danh sách công việc và tiến độ trong ngày của nhân sự toàn công ty.<br>- Xem báo cáo thống kê công việc theo tuần, tháng toàn công ty (*Read-only*, kiểm tra quyền qua API gốc). |
| **3. Quản lý phòng ban (Department Manager)** | Nhu cầu theo dõi tiến độ nhân sự trong nội bộ phòng ban quản lý | - Xem danh sách công việc và tiến độ trong ngày của nhân sự thuộc phòng mình.<br>- Xem báo cáo thống kê tuần, tháng của phòng mình (*Read-only*, kiểm tra quyền qua API gốc). |
| **4. Hệ thống Chấm công & Phân quyền gốc SC24h (External System)** | *"Mỗi khi nhân viên chấm công... phần kiểm tra user có phải nộp báo cáo hay quyền xem báo cáo sẽ thông qua API"* | - Tiếp nhận yêu cầu Check-in / Check-out tức thời.<br>- Cung cấp API kiểm tra User có phải nộp báo cáo Checklist khi Check-in/Check-out không.<br>- Cung cấp API kiểm tra quyền và phạm vi xem báo cáo Checklist Task. |

---

### Câu 5: Kết quả mong muốn là gì? (Acceptance Criteria)

| Mục tiêu hệ thống | Tiêu chí đạt cụ thể (Kiểm chứng được qua API) |
|---|---|
| **G1: Tích hợp Chấm công tức thời & Kiểm tra điều kiện Checklist qua API** | - Khi User bấm Check-in / Check-out, Backend TimeKeeper gọi ngay API chấm công gốc để không bị lệch thời gian, đồng thời kiểm tra qua API gốc xem User có phải nộp báo cáo checklist hay không.<br>- Việc tạo/chọn task hay cập nhật tiến độ diễn ra độc lập trên TimeKeeper, không làm ảnh hưởng tới hệ thống chấm công gốc. |
| **G2: Quản lý Template Task & Luồng Check-in lập Checklist ngày** | - Nhân viên có thể tạo/sửa/xóa Template Task cá nhân từ trước.<br>- Khi Check-in (nếu thuộc diện nộp báo cáo), API trả về danh sách Template Task để nhân viên chọn hoặc tạo thêm task mới cho ngày hôm nay. |
| **G3: Luồng Check-out cập nhật tiến độ** | - Khi nhân viên thực hiện Check-out, API trả về đúng danh sách công việc đã đăng ký trong ngày hôm nay.<br>- Cho phép cập nhật kết quả từng công việc theo **% tiến độ hoàn thành (0% – 100%)** hoặc trạng thái **chưa làm**. |
| **G4: Báo cáo & Thống kê cho Ban Giám đốc / Quản lý** | - Cung cấp API truy vấn danh sách công việc và thống kê tiến độ theo **ngày hôm nay, theo tuần và theo tháng** sau khi đã xác thực quyền xem báo cáo qua API của Hệ thống gốc. |

---

## 3. Bảng Tổng hợp Các Quyết định Nghiệp vụ & Kiến trúc Đã Chốt

| Mã | Hạng mục quyết định | Nội dung đã chốt | Trạng thái |
|:---:|---|---|:---:|
| **D1** | Phạm vi triển khai dự án | Chỉ xây dựng **Backend API** (NestJS 11 + MongoDB). Phần giao diện Frontend do dev khác thực hiện. | `[Đã xác nhận]` |
| **D2** | Mô hình chấm công tức thời | Khi User chấm công, Backend TimeKeeper gọi **ngay lập tức** API chấm công gốc để chốt thời gian chuẩn; việc tạo task / cập nhật tiến độ là hành động độc lập của TimeKeeper. | `[Đã xác nhận]` |
| **D3** | Cơ chế kiểm tra nghĩa vụ nộp báo cáo & phân quyền | **Thông qua API của Hệ thống gốc SC24h** (TimeKeeper không tự lưu bảng cấu hình phòng ban hay tự xử lý phân quyền nội bộ). | `[Đã xác nhận]` |
| **D4** | Quyền hạn của Ban Giám đốc & Quản lý | Xem báo cáo, thống kê tiến độ công việc theo ngày, tuần, tháng ở chế độ **Read-only** theo phạm vi quyền trả về từ API Hệ thống gốc. | `[Đã xác nhận]` |
