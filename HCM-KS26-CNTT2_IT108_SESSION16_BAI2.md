# Bài Tập Thực Hành Session 17 - Lập Trình Web / Mạng Máy Tính

## Phần 1: Phân tích nguyên nhân sự cố

### 1. Tại sao nút "Thanh toán" vẫn hiện màu xanh cũ (Lỗi CSS không cập nhật)
- **Nguyên nhân:** Do trình duyệt đã lưu trữ file CSS cũ vào **Browser Cache** (Bộ nhớ đệm trình duyệt).
- **Cơ chế:** Khi ấn `F5` thông thường, trình duyệt vẫn ưu tiên tải lại các tài nguyên tĩnh (Static Assets như CSS, JS, hình ảnh) từ bộ nhớ đệm cục bộ trên máy tính để tăng tốc độ load trang. Vì URL của file CSS giữ nguyên (`style.css`), trình duyệt nhận diện file chưa đổi nên tiếp tục dùng bản CSS cũ.

### 2. Tại sao mở lại trình duyệt bị văng ra (Mất phiên đăng nhập)
- **Nguyên nhân:** Do Cookie xác thực đăng nhập hiện tại là **Session Cookie** (Cookie phiên).
- **Phân biệt Session Cookie vs Persistent Cookie:**
  - **Session Cookie:** Không được cài đặt thuộc tính `Max-Age` hay `Expires`. Cookie này chỉ sống tạm thời trong RAM của trình duyệt. Ngay khi đóng trình duyệt (Close Window/Tab), cookie bị xóa tự động khiến Session ID mất và người dùng phải đăng nhập lại.
  - **Persistent Cookie:** Được cài đặt thuộc tính `Max-Age` (thời gian sống tính bằng giây) hoặc `Expires` (ngày giờ hết hạn). Cookie này được lưu dưới ổ đĩa đệm và duy trì trạng thái đăng nhập kể cả khi đóng và mở lại trình duyệt.

---

## Phần 2: Đề xuất 2 hành động cụ thể để sửa lỗi

### 1. Cập nhật giao diện CSS mới ngay lập tức
- **Thao tác nhanh cho An (Hard Reload):** Sử dụng tổ hợp phím cưỡng chế trình duyệt bỏ qua đệm và tải lại tài nguyên mới từ Server:
  - Windows: `Ctrl + F5` hoặc `Ctrl + Shift + R`.
  - Mac: `Cmd + Shift + R`.
- **Giải pháp triệt để trong Code (Cache Busting):** Đổi đường dẫn file CSS trong HTML bằng cách thêm query string phiên bản:
  ```html
  <!-- Trước khi sửa -->
  <link rel="stylesheet" href="style.css">

  <!-- Sau khi sửa (Cache Busting) -->
  <link rel="stylesheet" href="style.css?v=1.1">
  ```
  Khi URL thay đổi (`?v=1.1`), trình duyệt tự động hiểu đây là tài nguyên mới và sẽ nạp lại CSS mới ngay lập tức mà không cần xóa lịch sử web.

### 2. Duy trì trạng thái đăng nhập lâu dài (Keep Login State)
- **Cấu hình Cookie phía Server:** Khi trả về response đăng nhập thành công (`Set-Cookie`), An bổ sung thuộc tính `Max-Age` hoặc `Expires` để chuyển thành **Persistent Cookie**.
- **Cấu hình ví dụ (`Set-Cookie` Header):**
  - Sử dụng `Max-Age` (Ví dụ giữ đăng nhập 7 ngày = 604800 giây):
    ```http
    Set-Cookie: session_id=xyz123abc; Max-Age=604800; Path=/; Secure; HttpOnly
    ```
  - Hoặc sử dụng `Expires` (Cài ngày giờ cụ thể):
    ```http
    Set-Cookie: session_id=xyz123abc; Expires=Wed, 10 Nov 2026 00:00:00 GMT; Path=/; Secure; HttpOnly
    ```
