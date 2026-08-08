#portswinger 
**Path Traversal (hay Directory Traversal)** là lỗ hổng bảo mật web cho phép kẻ tấn công đọc các tệp tin hệ thống bị ẩn trên máy chủ (mã nguồn, cấu hình, thông tin hệ thống).

**Cơ chế hoạt động:** Lợi dụng việc ứng dụng không kiểm tra kỹ đầu vào, hacker chèn các ký tự lùi thư mục (như `../` trên Linux hoặc `..\` trên Windows) để thoát khỏi thư mục gốc của web và truy cập hệ thống.

**Ví dụ khai thác:**

- **Bình thường:** `[website.com/get-file?file=avatar.png](https://website.com/get-file?file=avatar.png)`
    
- **Khai thác:** `[website.com/get-file?file=../../../../etc/passwd](https://website.com/get-file?file=../../../../etc/passwd)` -> Đọc được file chứa thông tin người dùng trên Linux.
    

**Hậu quả:**

- Lộ thông tin nhạy cảm (mật khẩu database, file cấu hình, mã nguồn).
    
- Đọc log hệ thống (hữu ích cho việc trinh sát).
    
- Có thể dẫn đến RCE (Thực thi mã từ xa) nếu lỗ hổng cho phép ghi đè file hệ thống.
    

**Cách phòng chống (Dành cho Dev & SOC):**

1. **Tuyệt đối không dùng trực tiếp:** Không dùng input của người dùng để ghép trực tiếp vào đường dẫn file. (Nên dùng ID hoặc Hash map với file name).
    
2. **Filter/Sanitize Input:** Loại bỏ các ký tự nguy hiểm như `../`, `..\`, hoặc null byte `%00`.
    
3. **Whitelist:** Chỉ cho phép truy cập các tệp có định dạng (ví dụ: chỉ `.png`, `.jpg`) hoặc tên nằm trong danh sách cho phép.
    
4. **Giới hạn quyền (Sandboxing):** Cấu hình quyền hệ điều hành (như `chroot` trên Linux) để ứng dụng web chỉ có thể đọc/ghi trong đúng thư mục của nó.