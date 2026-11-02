# Bài Tập Thực Hành Session 17 - Lập Trình Web / Mạng Máy Tính

## Phần 1: Báo cáo phân tích kiến trúc đa Profile

### 1. Phân tích nguyên nhân sâu xa dẫn đến hiện tượng 'văng phiên đăng nhập' (Session Conflict)
- **Gốc rễ lưu trữ:** Khi tất cả nhân sự dùng chung 1 Profile Chrome mặc định (`Default` profile trong thư mục `User Data`), toàn bộ Cookie của mọi website sẽ được lưu chung vào một tệp cơ sở dữ liệu SQLite duy nhất tại đường dẫn:
  `C:\Users\<Username>\AppData\Local\Google\Chrome\User Data\Default\Network\Cookies`
- **Cơ chế xung đột Cookie:**
  - Trình duyệt lưu giữ Cookie phiên làm việc theo tên miền (Domain) và Tên Cookie (Name).
  - Khi Bác sĩ trực đang làm việc trên trang CarePulse, trình duyệt lưu Cookie xác thực (Session Token A).
  - Khi thực tập sinh sử dụng cùng Profile đó mở tab mới để đăng nhập tài khoản khác (hoặc trùng ứng dụng SSO), yêu cầu `Set-Cookie` mới từ Server sẽ ghi đè (Overwrite) lên Token cũ trong tệp `Cookies` duy nhất này.
  - Khi Bác sĩ quay lại thao tác ở tab CarePulse, trình duyệt gửi Token bị ghi đè lên Server. Server kiểm tra thấy Token không hợp lệ nên trả về phản hồi bắt đăng nhập lại (`Logout` giữa chừng).

---

### 2. Thiết kế Mô hình 3 Profile chuyên dụng cho máy tính trạm khám CarePulse

| Tiêu chí | Profile 1: Work (Y tế Chuyên môn) | Profile 2: Personal (Cá nhân Bác sĩ) | Profile 3: Sandbox / Test (Học tập & Thực tập) |
| :--- | :--- | :--- | :--- |
| **Tên Profile** | `CarePulse_Work` | `Doctor_Personal` | `Student_Sandbox` |
| **Mục đích** | Phục vụ trực khám, kê đơn, tra cứu bệnh án y tế CarePulse. | Bác sĩ dùng tra cứu cá nhân, xem tin tức, Facebook, giải trí lúc nghỉ. | Thực tập sinh dùng học tập PTIT, GitHub, test chức năng thử nghiệm. |
| **Theme thị giác** | **Xanh Dương (Teal/Blue)** - Chuẩn Y tế | **Xám / Tối (Dark/Grey)** - Nhã nhặn | **Cam / Vàng (Orange/Yellow)** - Cảnh báo |
| **Lưu trữ độc lập** | - Cookie kê đơn CarePulse<br>- History khám bệnh<br>- Bookmark Y tế<br>- Extension bảo mật y tế | - Cookie Shopee, Facebook<br>- History mua sắm/xem tin<br>- Bookmark tin tức cá nhân | - Cookie LMS PTIT, GitHub<br>- History học tập<br>- Extension phục vụ lập trình |

---

## Phần 2: Thiết kế hệ thống Bookmark & Quy trình an toàn

### 3. Cấu trúc cây thư mục Bookmark chuẩn (Ctrl + Shift + O)

Tree folder Bookmark gồm 3 thư mục chính nằm trên thanh dấu trang (Bookmarks Bar):

```text
📁 Bookmarks Bar
├── 📁 01_CarePulse_Clinic
│   ├── 🔗 Kê đơn thuốc trực tuyến (https://carepulse.vn/prescription)
│   ├── 🔗 Quản lý bệnh án điện tử (https://carepulse.vn/records)
│   └── 🔗 Cổng thông tin BHYT (https://baohiemxahoi.gov.vn)
├── 📁 02_Pharma_Portal
│   ├── 🔗 Tra cứu Dược thư Quốc gia (https://duocthu.gov.vn)
│   ├── 🔗 Cục Quản lý Dược - Bộ Y tế (https://dav.gov.vn)
│   └── 🔗 Hệ thống Mã ATC thuốc (https://whocc.no)
└── 📁 03_Medical_Tools
    ├── 🔗 Google Dịch Y khoa (https://translate.google.com)
    ├── 🔗 Tra cứu chỉ số BMI / GFR (https://medcalc.org)
    └── 🔗 Từ điển Y học PubMED (https://pubmed.ncbi.nlm.nih.gov)
```

---

### 4. Áp dụng mẹo Favicon-Only tiết kiệm diện tích thanh công cụ

- **Cách thực hiện:** Nhấp chuột phải vào Bookmark trên thanh công cụ -> Chọn **Edit** (Sửa) -> Xóa sạch chữ ở ô **Name** (Tên), chỉ để lại duy nhất phần **URL**.
- **Ứng dụng cho 4 trang dùng hàng ngày:**
  1. `CarePulse`: Chỉ hiện icon logo chữ thập y tế.
  2. `Cổng BHYT`: Chỉ hiện icon logo Bảo hiểm xã hội.
  3. `Dược thư`: Chỉ hiện icon quyển sách dược.
  4. `Google Dịch`: Chỉ hiện icon chữ G dịch thuật.
- **Hiệu quả:** Tiết kiệm gấp 3-4 lần diện tích thanh công cụ, giúp hiển thị được hơn 30+ bookmark quan trọng trên màn hình mà không cần mở danh sách cuộn.

---

### 5. Xử lý Bẫy an ninh: Chế độ duyệt web khi có đoàn kiểm tra & Cảnh báo Cloud Sync

#### 1. Kích hoạt chế độ duyệt web nào khi có khách / đoàn kiểm tra mượn máy 30 phút?
- **Giải pháp bắt buộc:** Kích hoạt **Guest Profile** (Chế độ Khách).
- **Phân biệt với Incognito:** 
  - *Incognito (Ẩn danh):* Vẫn truy cập được vào các Bookmark, Extension đã cài trên Profile đó và vẫn mở ra từ cửa sổ làm việc của bác sĩ.
  - *Guest Profile (Chế độ Khách):* Tạo ra một môi trường hoàn toàn trống rỗng, không hiển thị bất kỳ Bookmark, Lịch sử duyệt web hay Suggesstion nào của phòng khám. Sau khi đóng cửa sổ Khách, toàn bộ lịch sử và cookie của khách sẽ bị xóa sạch lập tức.

#### 2. Cảnh báo nguy cơ nếu bật Cloud Sync tài khoản cá nhân trên máy trạm dùng chung
- **Nguy cơ 1 (Rò rỉ dữ liệu cá nhân):** Mật khẩu Facebook, Shopee, tài khoản ngân hàng, lịch sử tìm kiếm riêng tư của Bác sĩ sẽ tự động đồng bộ xuống đĩa cứng máy trạm. Bất kỳ sinh viên thực tập hay nhân sự khác ngồi vào máy đều có thể tự động đăng nhập vào tài khoản cá nhân của bác sĩ.
- **Nguy cơ 2 (Rò rỉ dữ liệu bệnh nhân):** Nếu đồng bộ tài khoản Google cá nhân lên Cloud Sync, các file bệnh án, danh sách bệnh nhân tải về có thể bị đẩy lên Google Drive/Cloud cá nhân, vi phạm nghiêm trọng Luật An toàn Thông tin Y tế và Bảo mật dữ liệu cá nhân.
