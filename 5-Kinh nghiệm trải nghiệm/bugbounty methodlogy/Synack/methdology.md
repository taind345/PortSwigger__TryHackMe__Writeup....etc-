# Phương Pháp Săn Bug Bounty Trên Synack (Synack Methodology)

---

## 1. QUY TRÌNH TỰ ĐỘNG HÓA (AUTOMATION)

1. **Tìm lỗ hổng:** Dùng `nuclei` hoặc các công cụ quét tự động quét diện rộng.
2. **Kiểm tra thống kê (Analytics):** Đối chiếu xem danh mục (category) hoặc endpoint đó đã có ai báo cáo trước đó hay chưa (tránh bị trùng - Duplicate).
3. **Khởi tạo bản báo cáo lỗ hổng qua API:** Dùng phương thức `POST` tiêu chuẩn lên Synack API.
4. **Tự động chụp ảnh màn hình bằng chứng (PoC):** Dùng `imagemagick`, `aquatone` hoặc công cụ tương tự.
5. **Đính kèm ảnh bằng chứng:** Sử dụng phương thức `PUT` qua API.
6. **Nộp báo cáo:** Sử dụng phương thức `POST` để hoàn tất submit.

---

## 2. PHÂN LOẠI MỤC TIÊU (TYPES OF TARGETS)

### Mục tiêu phạm vi rộng có Subdomain (Wide-Scope w/ Subdomains)
- Chờ đợi cơ hội xuất hiện (thiết lập giám sát liên tục).
- Hành động **cực kỳ nhanh** khi mục tiêu có tài sản Mới (New) hoặc vừa Cập nhật (Updated).

### Ứng dụng lớn có cấp tài khoản (Large App w/ Creds)
- Chủ động đào sâu và tìm kiếm các lỗi logic / kiểm soát truy cập.
- Phương pháp: Chậm rãi, bài bản, kiểm thử thủ công (manual).

---

## 3. THU THẬP THÔNG TIN & LIỆT KÊ (ENUMERATION)

### 1. Endpoint Quản Trị Bị Lộ (Exposed Admin Endpoint)
- Chức năng Admin dạng Ghi/Sửa (Admin Functions Write) -- **$800**
- Chức năng Admin dạng Đọc (Admin Functions Read) -- **$600**

### 2. Lộ Lọt Thông Tin (Information Disclosure)
- Lộ nội dung thư mục (Directory Contents Disclosed) -- **$150**
- Dò quét cấu trúc thư mục (Directory Structure Enumeration) -- **$170**
- Lộ sơ đồ cấu trúc mạng (Identity of Network Topology) -- **$150**
- Lộ kiến trúc phần mềm / framework (Identity of Software Architecture) -- **$150**
- Rò rỉ thông tin đăng nhập (username/password):
  - Tài khoản đặc quyền cao (High Privilege) -- **$700**
  - Tài khoản đặc quyền thấp (Low Privilege) -- **$250**
- Rò rỉ API Keys -- **$500**
- Thông tin khách hàng nhạy cảm (Sensitive Client Information - Tuân thủ / Quyền riêng tư) -- **$300**
- Nội dung tập tin / thư mục nhạy cảm (Sensitive Directory/File Contents) -- **$300**
- Mã nguồn nhạy cảm bị lộ (Sensitive Source Code) -- **$150**

---

## 4. QUÉT & TẤN CÔNG BẰNG CÔNG CỤ (SCANNING)

### Các cuộc tấn công phía Client (Client-Side Attacks)
1. **Dữ liệu phản xạ (Reflected Input):**
   - Giả mạo nội dung HTML (Spoof HTML Content) -- **$200**
   - Cross-Site Scripting (XSS):
     - Reflected XSS -- **$330**
     - Kết hợp với Web Cache Poisoning -- **$$$ (Tiền thưởng cực lớn)**
   - CSS Injection -- **$330**
2. **DOM-Based XSS** -- **$775**
3. **Cross-Site Request Forgery (CSRF):**
   - CSRF mức độ Cao (High Impact) -- **$500**
   - CSRF mức độ Thấp (Low Impact) -- **$400**

### Các cuộc tấn công phía Server (Server-Side Attacks)
1. **Tương tác dịch vụ bên ngoài (External Service Interaction / SSRF):**
   - **SSRF Toàn phần (SSRF Full):** Bắt buộc phải tương tác toàn diện với ứng dụng/dịch vụ nội bộ -- **$1,500**
   - **SSRF Giới hạn (SSRF Limited):** Phải chứng minh được tương tác với IP/Port nội bộ -- **$500**
2. **Xác thực dữ liệu đầu vào không hợp lệ (Improper Input Validation):**
   - Vượt qua cơ chế kiểm tra phía client (Bypass Client-Side Validations - Phải có tính lưu vết/persistent) -- **$150**
3. **Tải lên tập tin (File Upload):**
   - Tải file không giới hạn (Unrestricted File Upload) -- **$180**
4. **SQL Injection (SQLi):**
   - *Quy tắc: Chạy SQLMap và kiểm tra qua Burp Suite trên MỌI tham số.*
   - **SQLi Toàn phần (Full):** Phải trích xuất được dữ liệu trong cơ sở dữ liệu -- **$3,000**
   - **SQLi Một phần (Partial / Blind):** -- **$1,500**
5. **Local/Remote File Inclusion & Path Traversal (Đọc/Chèn file trái phép):** -- **$850**
6. **CRLF Injection:** -- **$300**
7. **HTTP Host Header Injection:**
   - Open Mail Relay (Phải gửi được email đến hòm thư bên ngoài bất kỳ) -- **$400**
8. **Blind XSS:** -- **$880**
9. **XML External Entity (XXE):**
   - XXE Giới hạn (Limited) -- **$500**
   - XXE Toàn phần (Full - Đọc file/SSRF) -- **$1,500**

---

## 5. CÁC HẠNG MỤC KIỂM THỬ THỦ CÔNG ĐỊNH KỲ (MANUAL ROUTINE)

1. **Liệt kê tài khoản người dùng (Account Enumeration):** -- **$150**
   - Mật khẩu mặc định tài khoản Admin (Default Credentials Admin) -- **$700**
   - Mật khẩu mặc định tài khoản thường (Default Credentials Non-Admin) -- **$250**
2. **Session Fixation (Cố định phiên làm việc):** -- **$100**
3. **Vượt qua CAPTCHA (Captcha Bypass):** -- **$200**
4. **Cấu hình sai trên AWS (AWS Misconfigurations):**
   - Khai thác / chiếm đoạt dữ liệu AWS S3 Bucket -- **$450**
5. **Dependency Confusion (Xung đột gói thư viện phụ thuộc):** -- **$750**

---

## 6. CÁC HẠNG MỤC SÁNG TẠO THỦ CÔNG (MANUAL CREATIVE)

1. **Vi phạm kiểm soát truy cập (Access Control Violation / BOLA / BFLA):**
   - Chức năng Admin -- Đọc/Ghi hoặc Chỉ Ghi (Admin Functions Read/Write or Write Only) -- **$800**
   - Chức năng Admin -- Chỉ Đọc (Admin Functions Read Only) -- **$600**
   - Chức năng Non-Admin -- Đọc/Ghi hoặc Chỉ Ghi (Phải xóa hoặc sửa đổi được dữ liệu từ thư mục cá nhân của người dùng khác) -- **$450**
   - Chức năng Non-Admin -- Chỉ Đọc (Phải đọc được dữ liệu cá nhân của người dùng khác) -- **$300**
2. **Tham chiếu đối tượng trực tiếp không an toàn (IDOR):**
   - IDOR Chỉ Đọc (IDOR Read Only) -- **$500**
   - IDOR Cả Đọc và Ghi (IDOR Read and Write) -- **$600**
3. **Vượt qua xác thực đăng nhập (Login Authentication Bypass):** -- **$850**
4. **Vượt qua xác thực 2FA / MFA (2FA/MFA Authentication Bypass):** -- **$500**
5. **Vượt qua xác thực SSO (SSO Authentication Bypass):** -- **$750**
