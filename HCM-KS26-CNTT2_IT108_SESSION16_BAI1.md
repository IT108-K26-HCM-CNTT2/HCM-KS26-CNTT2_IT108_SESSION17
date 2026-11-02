# Bài Tập Thực Hành Session 17 - Lập Trình Web / Mạng Máy Tính

## Phần 1: Phân tích lỗi sai trong lập luận của Nam

### 1. Phân tích lỗi sai về DNS Lookup
- **Lập luận của Nam:** Nam cho rằng DNS Lookup không tìm thấy địa chỉ IP nên máy mới báo lỗi `500 Internal Server Error`.
- **Thực tế:** 
  - Nếu DNS Lookup thất bại (không phân giải được tên miền `lms.ptit.edu.vn` thành địa chỉ IP), trình duyệt sẽ dừng lại ngay từ đầu và hiển thị các lỗi phía Client như `ERR_NAME_NOT_RESOLVED` hoặc `Server IP address could not be found`.
  - Việc trình duyệt nhận được mã phản hồi **500** chứng tỏ quá trình DNS Lookup **đã thành công 100%**. Trình duyệt đã lấy được IP của máy chủ PTIT, đã kết nối thành công và gửi được gói tin HTTP Request tới máy chủ.

### 2. Phân tích bản chất mã lỗi 500 Internal Server Error
- **Mã lỗi 500 là gì?** Đây là mã trạng thái thuộc nhóm **5xx (Server Error)**. Nó cho biết yêu cầu từ phía người dùng đã gửi tới máy chủ thành công, nhưng máy chủ (Web Server / Database / App Server) gặp sự cố nội bộ trong lúc xử lý nên không thể trả về kết quả bình thường.
- **Đối chiếu mã trạng thái:**
  - `200 OK`: Truy cập và xử lý thành công.
  - `404 Not Found`: Lỗi phía Client (4xx) - gõ sai đường dẫn hoặc trang không tồn tại.
  - `500 Internal Server Error`: Lỗi hoàn toàn thuộc về **phía Server** (lỗi code backend, lỗi kết nối Database, máy chủ bị quá tải CPU/RAM,...).
- **Kết luận:** Đường truyền mạng nhà Nam và DNS 8.8.8.8 hoàn toàn bình thường. Việc đổi DNS sẽ **không thể khắc phục được lỗi 500** này vì nguyên nhân nằm ở hệ thống máy chủ của LMS PTIT.

---

## Phần 2: Sơ đồ luồng dữ liệu gói tin (HTTP Request Lifecycle)

Luồng đi của gói tin từ khi Nam nhấn Enter đến khi nhận phản hồi mã lỗi 500:

```text
[ Trình duyệt (Nam) ] 
       │
       ├─► 1. DNS Lookup ──► Phân giải lms.ptit.edu.vn ──► Trả về IP Server (Thành công)
       │
       ├─► 2. TCP Handshake (3-way) & TLS ──► Kết nối IP Server thành công
       │
       ├─► 3. Gửi HTTP Request (GET / HTTP/1.1) ──► Gửi tới Web Server
       │
       │                                            [ Server PTIT ]
       │                                                   │
       │                                                   ├─► Web Server nhận request và chuyển qua App Server
       │                                                   │
       │                                                   └─► App Server bị crash / Lỗi Code / Mất kết nối CSDL
       │                                                                      │
       │                                                                      ▼
       │                                                        Sinh lỗi nội bộ (Fatal Error)
       │                                                                      │
       └◄─ 4. Trả về HTTP Response (Status 500) ◄─────────────────────────────┘
```

### Chi tiết các bước diễn ra:
1. **DNS Lookup:** Trình duyệt tra cứu tên miền `lms.ptit.edu.vn` và nhận về IP của máy chủ.
2. **Khởi tạo kết nối:** Trình duyệt bắt tay 3 bước TCP (TCP 3-way handshake) và thiết lập kênh truyền mã hóa SSL/TLS với Server.
3. **Gửi HTTP Request:** Trình duyệt gửi gói tin HTTP GET Request để xin nội dung trang web.
4. **Xử lý tại Server & Phát sinh lỗi 500:** 
   - Web Server tiếp nhận request và gọi chương trình Backend xử lý.
   - Ứng dụng Backend phát sinh lỗi nghiêm trọng (ví dụ: tràn kết nối Database, crash nguồn code, lỗi tài nguyên server).
   - Máy chủ đóng gói và trả về cho Client mã trạng thái **HTTP 500 Internal Server Error**.
5. **Hiển thị lỗi:** Trình duyệt nhận HTTP 500 và hiển thị màn hình báo lỗi máy chủ cho Nam.
