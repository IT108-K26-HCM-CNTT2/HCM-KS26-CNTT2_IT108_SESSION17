# Bài Tập Thực Hành Session 17 - Lập Trình Web / Mạng Máy Tính

## Phần 1: Phân tích lỗ hổng & nguyên nhân sự cố

### 1. Nguyên nhân trình duyệt hiển thị dữ liệu cũ (Cơ chế Browser Cache)
- **Hiện tượng:** Khi bác sĩ bấm F5, các file `medicine_catalog.css` và `pricing_data.json` hiển thị kích thước là `(from disk cache)` hoặc `(from memory cache)` với thời gian tải cực nhanh là 2ms.
- **Giải thích cơ chế:**
  - Trình duyệt web đã lưu trữ (cache) các file tài nguyên tĩnh này vào đĩa cứng (Disk) hoặc RAM của máy tính cục bộ ở những lần truy cập trước.
  - Khi bấm `F5` thông thường, trình duyệt kiểm tra thấy các tệp này vẫn còn hiệu lực đệm (Cache Validation) nên **không gửi HTTP Request về Server** mà nạp trực tiếp bản sao từ bộ nhớ máy tính.
  - Vì vậy, dù Server đã tải lên bảng giá và danh mục thuốc mới từ tối qua, trình duyệt của bác sĩ vẫn giữ nguyên và render dữ liệu cũ của tuần trước.

### 2. Phân tích lỗ hổng bảo mật & Hiểm họa Session Hijacking

- **Câu lệnh Hacker dùng để đánh cắp Token Cookie:**
  Nếu trang web dính lỗi Cross-Site Scripting (XSS), mã độc JavaScript của hacker có thể dễ dàng truy xuất giá trị Token Cookie phiên làm việc qua câu lệnh:
  ```javascript
  const stolenToken = document.cookie;
  // Hoặc trích xuất riêng token bác sĩ:
  console.log(document.cookie.match(/carepulse_doctor_session=([^;]+)/)[1]);
  ```
- **Tác hại của việc thiếu cờ HttpOnly và Secure:**
  - **Thiếu cờ `HttpOnly`:** Cho phép mã JavaScript phía Client có quyền đọc/ghi Cookie thông qua `document.cookie`. Khi bị dính XSS, hacker có thể đánh cắp chuỗi Token `TOKEN_XYZ883921983` và gửi về máy chủ của kẻ tấn công.
  - **Thiếu cờ `Secure`:** Trình duyệt sẵn sàng gửi Cookie phiên qua kết nối HTTP thường (chưa mã hóa). Hacker nằm trong cùng mạng Wi-Fi/LAN phòng khám có thể dùng công cụ bắt gói tin (Wireshark, Ettercap) nghe lén và lấy trọn vẹn Cookie Token.
  - **Thiếu/Đặt `SameSite=None` không an toàn:** Dễ bị tấn công giả mạo yêu cầu chéo trang (CSRF).
- **Hiểm họa đối với hồ sơ bệnh án (Session Hijacking):**
  - Hacker có được Cookie phiên sẽ giả mạo hoàn toàn danh tính của Bác sĩ mà không cần biết mật khẩu.
  - Hacker có quyền truy cập, chỉnh sửa, giả mạo kết quả xét nghiệm, xóa hồ sơ bệnh án hoặc đánh cắp thông tin cá nhân và dữ liệu y tế nhạy cảm của toàn bộ bệnh nhân trong phòng khám CarePulse.

---

## Phần 2: Hoàn thiện giải pháp & Thao tác DevTools

### 3. Hướng dẫn bác sĩ thực hiện Hard Reload (Làm mới cưỡng bức)
Để cập nhật danh mục thuốc mới ngay lập tức mà không cần xóa toàn bộ lịch sử duyệt web:
- **Trên hệ điều hành Windows:** Nhấn tổ hợp phím `Ctrl + F5` hoặc `Ctrl + Shift + R`.
- **Trên hệ điều hành Mac:** Nhấn tổ hợp phím `Cmd + Shift + R`.
- **Tác dụng:** Cưỡng chế trình duyệt bỏ qua toàn bộ bản lưu `(from disk cache)` / `(from memory cache)`, phát lệnh tải lại toàn bộ file CSS, JSON từ Server.

---

### 4. Viết lại dòng lệnh phản hồi Set-Cookie chuẩn an ninh y tế

```http
Set-Cookie: carepulse_doctor_session=TOKEN_XYZ883921983; Domain=carepulse.vn; Path=/; Expires=Wed, 21 Oct 2026 07:00:00 GMT; HttpOnly; Secure; SameSite=Strict
```

**Giải thích các cờ bổ sung:**
- `HttpOnly`: Cấm JavaScript (`document.cookie`) truy cập Cookie -> Chống đánh cắp Cookie qua XSS.
- `Secure`: Bắt buộc chỉ gửi Cookie qua kết nối mã hóa HTTPS -> Chống nghe lén trên đường truyền.
- `SameSite=Strict`: Ngăn chặn trình duyệt tự động gửi Cookie trong các request đến từ website bên ngoài -> Chống tấn công CSRF.

---

### 5. Quy trình 3 bước kiểm tra Cookie và vô hiệu hóa Cache trên Chrome DevTools (F12)

1. **Bước 1: Mở Chrome DevTools và kiểm tra Cookie (Tab Application)**
   - Bấm `F12` (hoặc `Ctrl + Shift + I`), chọn tab **Application**.
   - Tại menu bên trái, tìm mục **Storage** -> mở rộng **Cookies** -> click vào domain `https://carepulse.vn`.
   - Kiểm tra cột **Name** (`carepulse_doctor_session`), kiểm tra xem cột **HttpOnly** và **Secure** đã có dấu tích xanh hay chưa.

2. **Bước 2: Bật tính năng vô hiệu hóa Cache khi Dev (Tab Network)**
   - Chuyển sang tab **Network** trong giao diện DevTools.
   - Tích chọn vào ô **Disable cache** (ngay dưới danh sách các tab công cụ).
   - *Lưu ý:* Tính năng này có hiệu lực vô hiệu hóa cache khi cửa sổ F12 đang mở, phục vụ công tác kiểm thử và lập trình.

3. **Bước 3: Thực hiện Hard Reload trực tiếp qua nút Reload của trình duyệt**
   - Khi DevTools F12 đang mở, nhấp **chuột phải vào nút Reload (xoay tròn)** trên thanh địa chỉ của Chrome.
   - Chọn dòng menu: **Empty Cache and Hard Reload** (Xóa bộ nhớ đệm và Tải lại cứng) để đảm bảo trình duyệt kéo toàn bộ bảng giá và danh mục thuốc mới nhất từ Server.
