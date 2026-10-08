# SRS – Module Login (Perfex CRM | Anh Tester Demo)

| Mục | Nội dung |
|---|---|
| Hệ thống | Perfex CRM – Anh Tester Demo |
| URL | https://crm.anhtester.com/admin/authentication |
| Module | Authentication – Login (khu vực Admin) |
| Phiên bản tài liệu | 1.0 (Draft) |
| Ngày | 2026-10-08 |
| Nguồn | Khảo sát giao diện trang Login (reverse-engineering từ UI) |

> **Lưu ý:** Tài liệu được suy ra từ giao diện quan sát được. Các mục đánh dấu **(Giả định)** là hành vi thông thường của Perfex CRM chưa được xác minh trên hệ thống thật (chưa thực hiện đăng nhập). Cần BA/Dev xác nhận.

---

## 1. Giới thiệu

### 1.1 Mục đích
Mô tả yêu cầu chức năng và phi chức năng của module Login, cho phép người dùng nội bộ (staff/admin) xác thực để truy cập khu vực quản trị của CRM.

### 1.2 Phạm vi
- Trong phạm vi: màn hình Login, xác thực email/mật khẩu, "Remember me", liên kết "Forgot Password?", điều hướng sau đăng nhập, thông báo lỗi.
- Ngoài phạm vi: luồng Forgot/Reset Password chi tiết, đăng nhập khách hàng (Client portal), quản lý người dùng/phân quyền.

### 1.3 Đối tượng sử dụng
| Vai trò | Mô tả |
|---|---|
| Admin | Quản trị viên hệ thống |
| Staff | Nhân viên có tài khoản trong CRM |

### 1.4 Thuật ngữ
| Thuật ngữ | Ý nghĩa |
|---|---|
| CSRF token | Token ẩn chống tấn công giả mạo request (hidden field trong form) |
| Remember me | Ghi nhớ phiên đăng nhập lâu hơn |

---

## 2. Mô tả tổng quan

### 2.1 Bối cảnh
Login là cổng vào duy nhất của khu vực `/admin`. Người dùng chưa xác thực truy cập bất kỳ trang admin nào đều bị chuyển về trang này **(Giả định)**.

### 2.2 Giao diện màn hình Login
Bố cục: logo "Anh Tester – Automation Testing" ở đầu, tiêu đề **Login**, form đặt giữa trang.

| # | Thành phần | Loại | Ghi chú |
|---|---|---|---|
| 1 | Logo | Image / Link | Link về `https://crm.anhtester.com/` |
| 2 | Tiêu đề "Login" | Heading | |
| 3 | CSRF token | Hidden input | Ẩn, do server sinh |
| 4 | Email Address | Textbox `type=email` | Bắt buộc |
| 5 | Password | Textbox `type=password` | Bắt buộc, che ký tự |
| 6 | Remember me | Checkbox | Mặc định bỏ chọn |
| 7 | Login | Button `type=submit` | Gửi form |
| 8 | Forgot Password? | Link | → `/admin/authentication/forgot_password` |

### 2.3 Giả định & ràng buộc
- Hệ thống là web app, truy cập qua trình duyệt, kết nối HTTPS.
- Tài khoản do Admin tạo; không có chức năng tự đăng ký trên màn hình này.
- Dữ liệu demo: tài khoản test `admin@example.com` (mật khẩu cung cấp riêng, không ghi trong tài liệu).

---

## 3. Yêu cầu chức năng

### FR-LOGIN-01 – Hiển thị trang Login
- Truy cập `/admin/authentication` hiển thị đủ các thành phần ở mục 2.2.
- Title trang: `Perfex CRM | Anh Tester Demo - Login`.
- Focus ban đầu có thể đặt ở ô Email **(Giả định)**.

### FR-LOGIN-02 – Nhập Email
- Trường bắt buộc, kiểu `email`.
- Hợp lệ khi đúng định dạng `local@domain`.
- Email sai định dạng bị chặn bởi validate trình duyệt/server.

### FR-LOGIN-03 – Nhập Password
- Trường bắt buộc; ký tự được che.
- Phân biệt chữ hoa/thường.
- Không hiển thị giá trị đã nhập sau khi đăng nhập thất bại.

### FR-LOGIN-04 – Đăng nhập
1. User nhập email + mật khẩu hợp lệ, nhấn **Login** (hoặc Enter).
2. Server kiểm tra CSRF token, thông tin đăng nhập và trạng thái tài khoản.
3. Thành công → tạo phiên, chuyển tới Dashboard (`/admin`) hoặc trang yêu cầu trước đó **(Giả định)**.
4. Thất bại → ở lại trang Login, hiển thị thông báo lỗi (FR-LOGIN-07).

### FR-LOGIN-05 – Remember me
- Chọn: phiên đăng nhập được duy trì lâu hơn sau khi đóng trình duyệt **(Giả định)**.
- Không chọn: phiên kết thúc theo thời gian hết hạn session / khi đóng trình duyệt.

### FR-LOGIN-06 – Forgot Password
- Click **Forgot Password?** chuyển tới `/admin/authentication/forgot_password`.

### FR-LOGIN-07 – Thông báo lỗi
| Tình huống | Thông báo / hành vi mong đợi |
|---|---|
| Để trống Email hoặc Password | Thông báo trường bắt buộc, không đăng nhập |
| Email sai định dạng | Báo định dạng email không hợp lệ |
| Sai email hoặc mật khẩu | Thông báo chung "Invalid email or password" **(Giả định)**, không tiết lộ trường nào sai |
| Tài khoản bị vô hiệu hóa (inactive) | Từ chối đăng nhập kèm thông báo **(Giả định)** |

### FR-LOGIN-08 – Người dùng đã đăng nhập
- Truy cập lại trang Login khi đã có phiên hợp lệ → tự chuyển về Dashboard **(Giả định)**.

### FR-LOGIN-09 – Phiên & điều hướng khi chưa đăng nhập
- Truy cập trực tiếp URL admin khi chưa đăng nhập → chuyển về trang Login.

---

## 4. Yêu cầu phi chức năng

| Mã | Loại | Yêu cầu |
|---|---|---|
| NFR-01 | Bảo mật | Truyền dữ liệu qua HTTPS; mật khẩu không hiển thị plain text, không nằm trong URL |
| NFR-02 | Bảo mật | Form có CSRF token; chống SQL Injection, XSS ở các trường nhập |
| NFR-03 | Bảo mật | Thông báo lỗi không tiết lộ email có tồn tại hay không |
| NFR-04 | Bảo mật | Có cơ chế giới hạn thử sai nhiều lần (rate limit/khóa tạm) **(Giả định)** |
| NFR-05 | Hiệu năng | Trang Login tải ≤ 3 giây; xử lý đăng nhập ≤ 3 giây trong điều kiện bình thường |
| NFR-06 | Tương thích | Chrome, Firefox, Edge, Safari phiên bản mới |
| NFR-07 | Responsive | Hiển thị đúng trên desktop, tablet, mobile |
| NFR-08 | Khả dụng | Hỗ trợ Tab/Enter; label gắn đúng với input |
| NFR-09 | Ngôn ngữ | Giao diện tiếng Anh |

---

## 5. Quy tắc nghiệp vụ

| Mã | Quy tắc |
|---|---|
| BR-01 | Email là định danh đăng nhập duy nhất của staff |
| BR-02 | Chỉ tài khoản active mới được đăng nhập |
| BR-03 | Sau đăng nhập, quyền truy cập menu/chức năng phụ thuộc vai trò (permission) của user |

---

## 6. Luồng nghiệp vụ

### 6.1 Luồng chính – Đăng nhập thành công
```
User mở /admin/authentication
 → nhập Email + Password
 → (tuỳ chọn) tick Remember me
 → click Login
 → hệ thống xác thực thành công
 → chuyển tới Dashboard
```

### 6.2 Luồng thay thế
- A1: Click **Forgot Password?** → trang khôi phục mật khẩu.
- A2: Đã có phiên hợp lệ → tự chuyển Dashboard.

### 6.3 Luồng ngoại lệ
- E1: Trường trống / sai định dạng → báo lỗi validate.
- E2: Sai thông tin → báo lỗi, giữ lại ở trang Login.
- E3: Tài khoản inactive → từ chối.

---

## 7. Gợi ý các tình huống kiểm thử (đầu vào cho Test Case)

| ID | Tình huống | Kết quả mong đợi |
|---|---|---|
| TC-01 | Đăng nhập với tài khoản hợp lệ | Vào Dashboard |
| TC-02 | Đăng nhập, bật Remember me | Vào Dashboard, phiên được ghi nhớ |
| TC-03 | Email đúng, password sai | Báo lỗi, không đăng nhập |
| TC-04 | Email không tồn tại | Báo lỗi chung |
| TC-05 | Để trống cả hai trường | Báo bắt buộc nhập |
| TC-06 | Để trống Email | Báo bắt buộc |
| TC-07 | Để trống Password | Báo bắt buộc |
| TC-08 | Email sai định dạng (`abc`, `a@`, `a@b`) | Báo sai định dạng |
| TC-09 | Password sai hoa/thường | Báo lỗi |
| TC-10 | Khoảng trắng đầu/cuối ở Email | Xử lý nhất quán (trim hoặc báo lỗi) |
| TC-11 | SQL Injection (`' OR 1=1 --`) | Không đăng nhập được |
| TC-12 | XSS (`<script>`) trong Email | Không thực thi script |
| TC-13 | Nhấn Enter để submit | Tương đương click Login |
| TC-14 | Click Forgot Password? | Chuyển trang quên mật khẩu |
| TC-15 | Truy cập URL admin khi chưa đăng nhập | Chuyển về Login |
| TC-16 | Mở Login khi đã đăng nhập | Chuyển Dashboard |
| TC-17 | Nút Back sau khi đăng nhập/đăng xuất | Không lộ dữ liệu phiên cũ |
| TC-18 | Password bị che, không copy được plain text | Đạt |
| TC-19 | Nhập sai nhiều lần liên tiếp | Có giới hạn/khóa tạm (nếu hỗ trợ) |
| TC-20 | Hiển thị trên mobile/tablet | Bố cục không vỡ |
| TC-21 | Tab order: Email → Password → Remember me → Login → Forgot | Đúng thứ tự |

---

## 8. Câu hỏi mở (cần làm rõ)

1. Thông báo lỗi chính xác cho từng trường hợp?
2. Có giới hạn số lần đăng nhập sai / CAPTCHA không?
3. Thời gian hết hạn session và thời hạn của Remember me?
4. Có 2FA không?
5. Chính sách mật khẩu (độ dài, độ phức tạp)?
6. Trang đích sau đăng nhập có phụ thuộc vai trò không?
