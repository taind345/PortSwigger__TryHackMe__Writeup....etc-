Để tự tay code được một dự án như thế này từ đầu, tư duy lập trình sẽ được chia thành **5 bước logic** đi từ dữ liệu đến giao diện. 

Dưới đây là sơ đồ tư duy chi tiết giúp bạn dễ dàng hình dung và áp dụng cho mọi dự án web sau này:

---

### Bước 1: Tư duy về Dữ liệu (Database Design)
Trước khi viết code PHP, ta cần xác định ứng dụng này quản lý những gì? Có 2 thực thể chính:
1. **Người dùng (Users):** Cần lưu thông tin tài khoản để đăng nhập.
2. **Ghi chú (Notes):** Cần lưu tiêu đề, nội dung và xem nó thuộc về ai.

* **Mối quan hệ:** 1 Người dùng có thể có Nhiều ghi chú (Mối quan hệ 1 - Nhiều).
* **Thiết kế bảng:**
  * Bảng `users`: gồm `id` (khóa chính), `username` (không trùng lặp), `password` (đã mã hóa).
  * Bảng `notes`: gồm `id` (khóa chính), `user_id` (khóa ngoại liên kết tới `users.id` để biết ghi chú này của ai), `title`, `content`.

---

### Bước 2: Thiết kế Kiến trúc dự án (Project Structure)
Vì yêu cầu code đơn giản, dễ đọc và dễ review, ta chọn mô hình **Multi-Page Application (MPA)**. Mỗi tính năng là một file riêng biệt:
* **Cấu hình & Kết nối:** [config.php](file:///d:/DESKTOP/training%20project/php+mysql/config.php) (lưu thông tin kết nối) và [db.php](file:///d:/DESKTOP/training%20project/php+mysql/db.php) (khởi tạo database/bảng tự động).
* **Xác thực:** [register.php](file:///d:/DESKTOP/training%20project/php+mysql/register.php) (Đăng ký), [login.php](file:///d:/DESKTOP/training%20project/php+mysql/login.php) (Đăng nhập), [logout.php](file:///d:/DESKTOP/training%20project/php+mysql/logout.php) (Đăng xuất).
* **Chức năng chính (CRUD):** 
  * [index.php](file:///d:/DESKTOP/training%20project/php+mysql/index.php): Trang chủ liệt kê danh sách + Tìm kiếm.
  * [create.php](file:///d:/DESKTOP/training%20project/php+mysql/create.php): Thêm mới.
  * [view.php](file:///d:/DESKTOP/training%20project/php+mysql/view.php): Xem chi tiết.
  * [edit.php](file:///d:/DESKTOP/training%20project/php+mysql/edit.php): Sửa.
  * [delete.php](file:///d:/DESKTOP/training%20project/php+mysql/delete.php): Xóa.

---

### Bước 3: Tư duy Quản lý trạng thái (Session & Authentication)
Làm sao hệ thống biết ai đang sử dụng để hiển thị đúng ghi chú của họ?
* Ta dùng **Session** của PHP (`session_start()`).
* Khi đăng nhập thành công: Lưu ID người dùng vào Session: `$_SESSION['user_id'] = $user['id']`.
* Ở đầu mỗi file chức năng (như trang chủ, thêm, sửa, xóa), ta cần kiểm tra: *Nếu Session chưa có `user_id` -> Chuyển hướng ngay về trang đăng nhập [login.php](file:///d:/DESKTOP/training%20project/php+mysql/login.php)*. Điều này giúp bảo vệ các trang riêng tư.

---

### Bước 4: Tư duy Xử lý nghiệp vụ (CRUD Logic)
Với mỗi trang, tư duy viết code luôn tuân theo quy trình 3 bước:
1. **Lấy dữ liệu:** Lấy từ form (`$_POST`) hoặc từ URL (`$_GET`).
2. **Kiểm tra và Xử lý nghiệp vụ:**
   * *Đăng ký:* Mã hóa mật khẩu bằng `password_hash()` để bảo mật.
   * *Sửa/Xóa/Xem:* Luôn phải kiểm tra `user_id` trong câu lệnh SQL để chắc chắn người dùng đang tương tác với **ghi chú của chính họ**, tránh việc người dùng A sửa được ghi chú của người dùng B bằng cách thay đổi ID trên URL.
3. **Thực thi SQL:** Dùng `PDO` với **Prepared Statements** (ví dụ `$stmt->execute([...])`) thay vì cộng chuỗi trực tiếp để chống lỗi bảo mật nghiêm trọng **SQL Injection**.

---

### Bước 5: Tối ưu hóa Trải nghiệm người dùng (UX) & Giao diện (CSS)
* Đặt tất cả style vào một file [style.css](file:///d:/DESKTOP/training%20project/php+mysql/style.css) để code HTML được sạch sẽ, dễ đọc.
* Thêm các thông báo lỗi/thành công rõ ràng (ví dụ: mật khẩu không khớp, đăng nhập sai).

---

### Tóm tắt cốt lõi
> Tư duy lập trình web cơ bản là **luồng đi của dữ liệu**:
> `Dữ liệu nhập từ Form/URL` $\rightarrow$ `PHP kiểm tra tính hợp lệ & kiểm tra Session` $\rightarrow$ `Kết nối DB thực thi SQL an toàn` $\rightarrow$ `Trả về giao diện HTML cho người dùng`.