# Bài Tập Thực Hành Session 17 - Lập Trình Web / Mạng Máy Tính

## Phần 1: So sánh & Đánh giá (Incognito Mode vs Dedicated Profile)

| Tiêu chí | Chế độ Ẩn danh (Incognito Mode) | Hồ sơ riêng biệt (Dedicated Profile) |
| :--- | :--- | :--- |
| **1. Cách ly dữ liệu (Data Isolation)** | Tạm thời. Xóa sạch Cookie, Cache, LocalStorage ngay khi tắt cửa sổ ẩn danh. | Tách biệt hoàn toàn tuyệt đối. Mỗi Profile có thư mục dữ liệu, Cookie, Cache riêng. |
| **2. Lưu giữ trạng thái (Persistence)** | **Rất kém.** Không lưu Bookmark, History, Extension hay tài khoản đăng nhập sau khi đóng. | **Tuyệt vời.** Lưu nguyên vẹn Bookmark, Session đăng nhập, Extension, History lâu dài. |
| **3. Hiệu suất làm việc (Productivity)** | Thấp. Phải gõ lại URL, đăng nhập lại tài khoản test từ đầu mỗi lần mở trình duyệt. | Cao. Đăng nhập 1 lần duy nhất, mở Profile là vào ngay làm việc, đồng bộ bookmark tài liệu. |
| **Ưu điểm** | Nhanh chóng cho bài test 1 lần, không dính cache cũ của phiên làm việc. | Quản lý công việc chuyên nghiệp, phân tách tài khoản cá nhân / dự án rõ ràng. |
| **Nhược điểm** | Mất hết dữ liệu, bookmark và extension hỗ trợ dev mỗi khi tắt cửa sổ. | Tốn thêm một chút dung lượng ổ đĩa cho bộ nhớ đệm riêng của từng Profile. |

---

## Phần 2: Lựa chọn & Tư vấn giải pháp cho thành viên A

### 1. Mô hình 3 Profile chuẩn Dev (Personal - Work - Sandbox)

Để giải quyết triệt để việc mất Bookmark tài liệu và phàn nàn đăng xuất/đăng nhập lại, tôi tư vấn cho thành viên A thiết lập mô hình 3 Profile độc lập trên trình duyệt (Chrome/Edge):

```text
[ Trình duyệt Web (Chrome / Edge) ]
        │
        ├──► 1. Profile 1: Personal (Cá nhân)
        │      ├── Email cá nhân, Facebook, Youtube, Giải trí
        │      └── Bookmark cá nhân
        │
        ├──► 2. Profile 2: Work (Dự án LMS - Dev)
        │      ├── Email sinh viên/công việc (PTIT), Jira, GitHub, LMS Admin
        │      ├── Bookmark: API Docs, Figma, DB Diagram, Swagger
        │      └── Cài sẵn Extension Dev: React DevTools, Redux DevTools, Postman
        │
        └──► 3. Profile 3: Sandbox / Test (Kiểm thử người dùng)
               ├── Dùng để đăng nhập tài khoản Student / Guest test tính năng
               └── Đã cài sẵn các dữ liệu mẫu để test nhanh
```

### 2. Lý do mô hình 3 Profile tối ưu hơn dùng Ẩn danh (Incognito)

1. **Lưu giữ Bookmark & Tài liệu kỹ thuật lâu dài:**
   Thành viên A có thể lưu lại tất cả link tài liệu (API spec, Figma, Note dự án) trên Profile Work. Mỗi lần bật máy lên chỉ cần mở Profile Work là có sẵn mọi thứ, không bị xóa sạch như Incognito.
2. **Không bao giờ bị xung đột Session hoặc phải đăng xuất:**
   Profile Personal giữ nguyên tài khoản cá nhân, Profile Work đăng nhập tài khoản Dev/Admin, Profile Sandbox dùng test tài khoản Học viên. Cả 3 chạy song song cùng lúc trên màn hình mà không đụng chạm Cookie của nhau.
3. **Giữ nguyên các Extension hỗ trợ Lập trình:**
   Profile Work được cài sẵn các tiện ích cần thiết (React DevTools, JSON Viewer, ColorPicker) giúp tăng tốc độ làm việc. Chế độ ẩn danh mặc định sẽ tắt toàn bộ Extension trừ khi cài đặt thủ công.
4. **Tiết kiệm thời gian & Tăng hiệu suất:**
   Loại bỏ hoàn toàn thao tác gõ lại tài khoản/mật khẩu test mỗi đầu ngày làm việc.
