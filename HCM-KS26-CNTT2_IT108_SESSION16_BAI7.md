# Bài Tập Thực Hành Session 17 - Lập Trình Web / Mạng Máy Tính

## Phần 1: Phân tích vấn đề

### 1. Chỉ ra 4 điểm sai lệch về mặt nguyên lý kỹ thuật của thực tập sinh

1. **Sai lệch về vai trò của hệ thống DNS (Bước 1):**
   - *Thực tập sinh phàn nàn/khẳng định:* Trình duyệt gửi trực tiếp URL `https://carepulse.vn/records/patient8821` tới Web Server mà không qua DNS.
   - *Thực tế kỹ thuật:* Trình duyệt không thể tự gửi gói tin TCP/IP trực tiếp bằng tên miền chữ (`carepulse.vn`). Bắt buộc phải qua hệ thống **DNS Lookup** (kiểm tra Cache trình duyệt, OS Cache, Router, DNS Server) để dịch tên miền thành địa chỉ **IP máy chủ** (ví dụ: `103.1.2.3`) và xác định cổng kết nối HTTPS mặc định là `443`.

2. **Sai lệch về quy trình bắt tay kết nối TCP/TLS (Bước 2):**
   - *Thực tập sinh khẳng định:* Trình duyệt tải ngay nguồn HTML mà không cần bắt tay TCP/TLS vì HTTPS tự động mở kết nối không điều kiện.
   - *Thực tế kỹ thuật:* Giao thức HTTPS yêu cầu bắt buộc hai giai đoạn khởi tạo kết nối an toàn:
     - **Bắt tay 3 bước TCP (TCP 3-way Handshake):** Gửi các gói tin `SYN` -> `SYN-ACK` -> `ACK` để thiết lập kênh truyền socket cơ bản giữa Client và Server.
     - **Bắt tay TLS/SSL (TLS Handshake):** Trao đổi chứng chỉ số SSL, xác thực độ tin cậy của máy chủ và trao đổi khóa mã hóa đối xứng trước khi gửi bất kỳ dữ liệu HTTP/HTML nào.

3. **Hiểu sai nguyên lý mã phản hồi HTTP Status Code (Bước 3):**
   - *Thực tập sinh giải thích:* Máy chủ Database sập nguồn làm Server trả về mã `404 Not Found`, và phán đoán là bệnh án bị xóa vĩnh viễn.
   - *Thực tế kỹ thuật:* 
     - Mã **404 Not Found** thuộc nhóm 4xx (Client Error): Cho biết đường dẫn URL không tồn tại trên hệ thống.
     - Khi Database bị sập/quá tải làm nổ query SQL phía backend, Server không thể truy vấn dữ liệu nên phải trả về mã **500 Internal Server Error** (thuộc nhóm 5xx - Server Error).

4. **Hiểu lầm nghiêm trọng về can thiệp DOM trên Tab Elements của DevTools (Bước 4 & Đề xuất):**
   - *Thực tập sinh tưởng rằng:* Sửa chữ "Âm tính - Bình thường" trong thẻ `<span>` trên tab Elements (F12) là đã sửa dữ liệu gốc thành công dưới Database máy chủ.
   - *Thực tế kỹ thuật:* Tab Elements chỉ thao tác trên cây cấu hình **DOM Tree** nằm tạm thời trên bộ nhớ **RAM phía Client (máy cục bộ)**. Việc sửa này hoàn toàn không gửi bất kỳ HTTP Request nào về Server và không ảnh hưởng tới dữ liệu lưu trữ trong Database.

---

### 2. Phân tích mã HTTP đúng khi Database sập & Hậu quả tâm lý bệnh nhân

- **Mã HTTP đúng khi Database bị sập phải là 500 Internal Server Error:**
  - Vì request đã đến máy chủ thành công, đường dẫn URL tồn tại. Lỗi nằm ở xử lý nội bộ backend (Database crash, unhandled exception). Do đó máy chủ phải trả về mã lỗi 500 (Server Error).
- **Hậu quả tai hại đối với bệnh nhân khi báo sai thành lỗi 404:**
  - **Tâm lý hoang mang, hoảng sợ:** Bệnh nhân tưởng hồ sơ bệnh án, lịch sử khám chữa bệnh hoặc kết quả xét nghiệm quan trọng của họ đã bị hệ thống làm mất hoặc xóa vĩnh viễn.
  - **Mất uy tín hệ thống CarePulse:** Gây hiểu nhầm rằng hệ thống y tế kém an toàn, quản lý dữ liệu yếu kém, dẫn tới khiếu nại pháp lý hoặc rời bỏ dịch vụ.

---

## Phần 2: Sửa lỗi & Hoàn thiện giải pháp kỹ thuật

### 3. Biên bản giải trình kỹ thuật chuẩn xác theo 4 bước vòng đời HTTP Request

#### Bước 1: DNS Lookup (Phân giải tên miền)
- Trình duyệt tra cứu tên miền `carepulse.vn` qua DNS Cache local (Trình duyệt, Hệ điều hành) và các Server DNS đại lý.
- DNS trả về địa chỉ IP của máy chủ CarePulse (ví dụ: `103.21.12.5`).

#### Bước 2: TCP Handshake & TLS Encryption (Thiết lập kết nối an toàn)
- **TCP 3-way Handshake:** Trình duyệt thực hiện giao tiếp `SYN` -> `SYN-ACK` -> `ACK` với IP `103.21.12.5` qua cổng `443`.
- **TLS Handshake:** Hai bên trao đổi chứng chỉ bảo mật SSL, xác thực máy chủ `carepulse.vn` và thiết lập kênh truyền mã hóa dữ liệu.

#### Bước 3: HTTP Request & Response (Gửi yêu cầu & Nhận phản hồi)
- Trình duyệt gửi gói tin `GET /records/patient8821 HTTP/1.1` kèm Cookie chứa Token xác thực.
- Máy chủ Backend tiếp nhận request nhưng dịch vụ Database SQL bị sập nên không thể truy xuất dữ liệu bệnh án `#8821`.
- Backend bắt lỗi ngoại lệ (Catch Exception) và trả về phản hồi mã trạng thái **HTTP Status 500 Internal Server Error** kèm payload JSON chứa thông báo lỗi hệ thống.

#### Bước 4: Rendering Pipeline (Dựng hình giao diện)
- Trình duyệt đọc dữ liệu phản hồi, dựng cây **DOM Tree** từ HTML và **CSSOM** từ CSS.
- Nhận được mã 500, JavaScript phía Client bắt lỗi và hiển thị giao diện thông báo "Hệ thống đang bảo trì máy chủ, vui lòng thử lại sau" thay vì bảng kết quả trống.

---

### 4. Giải thích nguyên lý bộ nhớ: Tại sao sửa trên Tab Elements không có giá trị lưu trữ

```text
[ Máy chủ CarePulse (Server-side) ]
    └── Database SQL (Lưu trữ bệnh án gốc trên Ổ Cứng/SSD)
           ▲ 
           │ (KHÔNG CÓ KẾT NỐI NÀO GỬI VỀ)
           │ 
[ Máy cục bộ bệnh nhân (Client-side) ]
    └── RAM máy tính ──► Cây DOM Tree (Chỉ sửa tạm thời trên F12) 
```

- **Nguyên lý phân tách Client - Server:**
  - **Database Server-side:** Dữ liệu bệnh án gốc được lưu trữ bền vững trên đĩa cứng của hệ thống máy chủ CarePulse. Chỉ có các lệnh HTTP POST/PUT/PATCH/DELETE có thẩm quyền mới thay đổi được Database.
  - **DOM trên DevTools:** Tab Elements chỉ cung cấp giao diện chỉnh sửa cây DOM Tree đang load tạm thời trên **bộ nhớ RAM** của trình duyệt máy khách (Client-side).
- **Hành vi khi F5 (Reload):**
  - Khi ấn F5, trình duyệt giải phóng toàn bộ cây DOM cũ trên RAM, gửi lại HTTP GET Request mới lên Server.
  - Server sẽ lấy dữ liệu gốc từ Database trả về, ghi đè toàn bộ thay đổi tạm thời mà thực tập sinh vừa gõ trên F12.
