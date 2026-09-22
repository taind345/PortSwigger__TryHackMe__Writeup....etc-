# Dấu Hiệu Nhận Biết Trong Bug Bounty (Bug Bounty Signs)

---

## 1. BỘ CÂU HỎI TƯ DUY (QUESTIONS)

### Tự đặt các câu hỏi sau cho Ứng dụng (Application):
1. **Tech Stack / Ngôn ngữ lập trình nào đang được sử dụng?**
2. **Web server nào đang chạy ứng dụng?** (Nginx, Apache, IIS, Cloudflare, v.v.)
3. **Có WAF (Tường lửa ứng dụng web) bảo vệ hay không?**
4. **Các thư viện phụ trợ nào được sử dụng?** Đã có mã khai thác (exploit) công khai cho các thư viện đó chưa? Có các thư viện JavaScript tùy chỉnh không?
5. **Cơ chế xác thực (Authentication) là gì?** (Ví dụ: OAuth qua Google/Facebook, JWT, Session Cookie, SAML, v.v.)
6. **Những đối tượng (Objects/Data Models) nào được sử dụng trong hệ thống?**
7. **Phiên làm việc (Session) được thiết lập và quản lý như thế nào?**
8. **Có các đoạn comment hữu ích nào bị sót lại trong mã nguồn/HTML/JS không?**
9. **Ứng dụng xử lý các ký tự đặc biệt như thế nào?** (Lọc bỏ, encode, hay báo lỗi?)
10. **Bạn có thể kích hoạt (trigger) các thông báo lỗi (error messages/stack traces) không?**
11. **Các tính năng phổ biến nào đang hiện diện?** (Đăng ký, quên mật khẩu, upload file, xuất báo cáo, bình luận, v.v.)
12. **Người dùng được định danh như thế nào?** (User ID số nguyên, UUID, Email, Username?)
13. **Có phân quyền nhiều cấp độ người dùng (User Roles) không?** (Admin, Manager, Member, Guest)
14. **Ứng dụng có API không?** (REST, GraphQL, SOAP, RPC)
15. **Có sử dụng hệ quản trị nội dung (CMS) nào không?** (WordPress, Drupal, Joomla, v.v.)
16. **Có triển khai CSP (Content Security Policy) không?**
17. **Cấu hình CORS (Cross-Origin Resource Sharing) được thiết lập ra sao?**
18. **Có sử dụng Captcha không?** (Ở những luồng nào: login, register, reset password?)
19. **Có sử dụng WebSockets không?**
20. **Mã nguồn ứng dụng có được công khai (publicly available) ở đâu không?** (GitHub, GitLab, npm, v.v.)

---

### Tự đặt các câu hỏi sau về Máy chủ lưu trữ ứng dụng (Hosting Server):
1. **Những cổng (ports) nào đang mở?**
2. **Những dịch vụ nào đang chạy trên các cổng đó?**
3. **Hạ tầng có được lưu trữ trên Cloud không?** (AWS, GCP, Azure, DigitalOcean)
4. **Máy chủ có chạy nhiều ứng dụng khác nhau thông qua Virtual Hosting (VHosting) không?**
5. **Hệ điều hành (OS) của máy chủ là gì?** (Linux, Windows Server)
6. **Bạn có thể lấy được phiên bản Kernel không?**

---

### Tự đặt các câu hỏi sau cho TỪNG TRANG WEB:
1. **Trang này thuộc phần nào trong CRUD?** (Create - Tạo, Read - Đọc, Update - Cập nhật, Delete - Xóa)
2. **Những phương thức HTTP nào có thể sử dụng?** (`GET`, `POST`, `PUT`, `DELETE`, `PATCH`, `OPTIONS`, v.v.)
3. **Những tham số (parameters) nào có thể truyền vào?** (Query param, Body param, Header, Path param)

---

## 2. TÌM KIẾM CƠ HỘI CHUỖI KHAI THÁC (CHAINING BUGS)

### 1. Có lỗi Open Redirect không?
- **Nếu CÓ:**
  - Có thể chuyển hướng (redirect) sang các đường dẫn (paths) khác không?
  - Có thể chuyển hướng sang các subdomain khác không?
  - Có thể chuyển hướng sang các domain ngoài hoàn toàn khác không? *(Dùng để đánh cắp OAuth token, bypass SSRF whitelist)*

### 2. Dữ liệu do người dùng kiểm soát có bị phản xạ (Reflected) trở lại không?
- **Nếu CÓ:**
  - Có bị **HTML Injection (HTMLi)** không?
  - Có bị **Cross-Site Scripting (XSS)** không?
  - Có bị **Server-Side Template Injection (SSTI)** không?

### 3. Có lỗi CSRF không?
- *Token CSRF dường như không bị vô hiệu hóa khi gửi đi hoặc khi người dùng đăng xuất.*
- **Nếu CÓ:**
  - Chúng ta có thể làm được gì với endpoint này? (Đổi email, đổi mật khẩu, xóa tài khoản?)
  - Endpoint này có bị dính Open Redirect không?

### 4. Thử thay đổi phương thức HTTP (HTTP Verb Tampering)?
- **Nếu CÓ:**
  - Endpoint có hoạt động như cũ khi đổi verb không? (Ví dụ: `POST` -> `GET`, `PUT` -> `POST`)
  - Có tham số nào bị từ chối hoặc cơ chế kiểm tra bảo mật nào bị bỏ qua không?

---

## 3. CHI TIẾT KIỂM THỬ TỪNG LOẠI LỖ HỔNG (VULN TESTING DETAILS)

| Loại lỗ hổng | Phương pháp & Công cụ kiểm thử |
| :--- | :--- |
| **Account Takeover (Chiếm đoạt tài khoản)** | Burp Suite (Thủ công / Manual) |
| **Code Injection (Chèn mã thực thi)** | Burp Suite (Quét tự động / Thủ công) |
| **HTML Injection** | Custom Script / Burp Suite (Quét / Thủ công) |
| **IDOR (Truy cập đối tượng không phân quyền)** | Burp Suite (Thủ công) |
| **Information Disclosure (Lộ lọt thông tin)** | Custom Script (`Github_brute-dork`) / Tìm kiếm thủ công |
| **Prototype Pollution** | Custom Script (`Drifting_Embers`) / Developer Tools (Thủ công) |
| **RCE (Thực thi mã từ xa)** | Nuclei (quét CVE đã biết) / Burp Suite (Thủ công) |
| **SSRF** | Burp Suite (Thủ công) |
| **XSS** | Custom Script / Burp Suite (Quét / Thủ công) |
| **SSTI** | Custom Script / Burp Suite (Quét tự động) |
| **CSRF** | Burp Suite (Thủ công) |
| **OAuth Flaws** | Burp Suite (Thủ công) |
| **Insecure Deserialization** | Burp Suite (Thủ công / Quét) / Phân tích mã nguồn |
| **HTTP Request Smuggling** | Burp Suite (HTTP Request Smuggler extension) |
| **WebSockets Flaws** | Burp Suite (Thủ công) |
| **HTTP Host Header Attacks** | Burp Suite (Thủ công) |

---

## 4. BẢNG MẪU BÁO CÁO PHÂN TÍCH LỖ HỔNG (VULN ANALYSIS TEMPLATE)

```markdown
REPORT (Đường link báo cáo gốc): 

TITLE (Tiêu đề): 

PROGRAM (Chương trình Bug Bounty): 

APP FUNCTION (Chức năng của ứng dụng): 

ENDPOINT (Điểm cuối bị ảnh hưởng): 

CRUD / EXPLOITED FUNCTION (Thao tác CRUD / Tính năng bị khai thác): 

SUMMARY (Tóm tắt lỗ hổng): 

TECHNICAL SKILL REQUIRED (Mức độ kỹ năng kỹ thuật yêu cầu): 
(Novice - Mới bắt đầu / Moderate - Trung bình / Expert - Chuyên gia)

TAKEAWAYS (Bài học / Kinh nghiệm rút ra): 
1. 
2. 
3. 

VULN DISCO AUTOMATION (Ý tưởng tự động hóa phát hiện): 
```
