# Bài Tập Thực Hành Session 17 - Lập Trình Web / Mạng Máy Tính

## Phần 1: Báo cáo phân tích (I/O) & Sơ đồ Box Model hiện tại

### 1. Phân tích hiện trạng chồng lấn giao diện (DevTools Tab Computed)
Khi sử dụng F12 soi vào 2 phần tử "Nội dung bài học" (`.course-content`) và "Danh sách video" (`.video-list`), nguyên nhân đè lên nhau xuất phát từ cơ chế tính toán kích thước mặc định của trình duyệt:

- **Cơ chế mặc định (`box-sizing: content-box`):**
  Chiều rộng thực tế = width + padding-left/right + border-left/right + margin-left/right
- **Nguyên nhân gây lỗi:**
  - Chiều rộng `width` được khai báo dạng % (ví dụ: 70% và 30%).
  - Khi thêm `padding` hoặc `border` dạng pixel (`px`), tổng chiều rộng thực tế bị phình to vượt quá 100% chiều rộng của thẻ cha, dẫn đến đè lấn hoặc đẩy vỡ khung.
  - Sử dụng `margin` âm hoặc định vị `position: absolute` thủ công mà không dọn dẹp (clear) luồng hiển thị làm hai thẻ đè trực tiếp lên nhau.

### 2. Sơ đồ 4 lớp Box Model của phần tử bị lỗi

```text
+-----------------------------------------------------------------------+
| MARGIN (Lề ngoài - Khoảng cách với phần tử xung quanh)                |
|   +---------------------------------------------------------------+   |
|   | BORDER (Viền phần tử)                                         |   |
|   |   +-------------------------------------------------------+   |   |
|   |   | PADDING (Đệm trong - Khoảng cách từ Border đến Content)|  |   |
|   |   |   +-----------------------------------------------+   |   |   |
|   |   |   | CONTENT (Nội dung chính: chữ, video, ảnh)    |   |   |   |
|   |   |   | - .course-content: width: 700px, height: 500px|   |   |   |
|   |   |   +-----------------------------------------------+   |   |   |
|   |   +-------------------------------------------------------+   |   |
|   +---------------------------------------------------------------+   |
+-----------------------------------------------------------------------+
```

### 3. Bẫy dữ liệu: Điều gì xảy ra nếu chỉ thay đổi Margin mà không kiểm tra Padding?
- Nếu chỉ thay đổi `margin` mà giữ `box-sizing: content-box`:
  - Kích thước tổng thể phần tử vẫn bị phình to do `padding` và `border`.
  - Việc tăng/giảm `margin` không khắc phục triệt để gốc rễ vấn đề, dễ làm đẩy vỡ khung của các phần tử lân cận hoặc khiến nội dung bị đè khuất khi co giãn màn hình.

---

## Phần 2: Thiết kế giải pháp sửa lỗi trong Styles

### 1. Chuẩn hóa cơ chế Box Model (Áp dụng Border-Box)
Chuyển toàn bộ phần tử sang `box-sizing: border-box` để `padding` và `border` được tính nằm gọn bên trong chiều rộng `width` đã khai báo:

```css
/* Reset CSS và chuẩn hóa Box Model cho toàn bộ trang */
* {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}
```

### 2. Cấu hình lại mã CSS cho hai khung nội dung

```css
/* Khung chứa tổng (Container) */
.main-container {
    display: flex;
    gap: 20px; /* Khoảng cách an toàn giữa 2 khung */
    width: 100%;
}

/* Khung 1: Nội dung bài học */
.course-content {
    box-sizing: border-box;
    flex: 1; /* Tự động co giãn chiếm phần không gian còn lại */
    padding: 16px;
    border: 1px solid #e0e0e0;
    margin: 0;
}

/* Khung 2: Danh sách video */
.video-list {
    box-sizing: border-box;
    width: 320px; /* Chiều rộng cố định cho sidebar video */
    padding: 16px;
    border: 1px solid #e0e0e0;
    margin: 0;
}
```

### Kết quả thu được:
- Hai khung tách rời độc lập, nằm cạnh nhau chuẩn xác.
- Không lo bị vỡ bố cục khi thay đổi `padding` hay `border` về sau.
