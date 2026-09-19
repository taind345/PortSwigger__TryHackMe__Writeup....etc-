-ok , đầu tiên là cần có tree kiến thức về CSRS đã, sau đó đá sang các lab về CSRS
CSRS có thể là bạn gõ nhầm **CSRF** (Cross-Site Request Forgery). Dưới đây là tree keyword chi tiết về CSRF để bạn học nhanh.

```
CSRF (Cross-Site Request Forgery)
│
├── 1. Khái niệm
│   ├── Định nghĩa
│   │   └── Lợi dụng browser tự động gửi request kèm credentials của nạn nhân
│   ├── Điều kiện để có CSRF
│   │   ├── Nạn nhân đã đăng nhập (có session/cookie hợp lệ)
│   │   ├── Browser tự động gửi cookie/session theo request
│   │   ├── Request có thể đoán trước (không có tham số ngẫu nhiên bí mật)
│   │   └── Server không phân biệt được request hợp lệ và request giả mạo
│   ├── Impact
│   │   ├── Thay đổi thông tin tài khoản
│   │   ├── Chuyển tiền, mua hàng
│   │   ├── Xóa dữ liệu
│   │   ├── Thay đổi email/mật khẩu
│   │   └── Leo quyền nếu chức năng nguy hiểm
│   └── Phân biệt
│       ├── CSRF: lợi dụng quyền của nạn nhân
│       ├── XSS: chạy JavaScript trong origin nạn nhân
│       └── SSRF: server-side, lợi dụng server gửi request
│
├── 2. Cơ chế hoạt động
│   ├── Same-Origin Policy (SOP)
│   │   ├── Ngăn đọc response từ origin khác
│   │   └── Không ngăn gửi request từ origin khác
│   ├── Browser tự động gửi cookie
│   │   ├── Cookie session
│   │   ├── Cookie xác thực
│   │   └── Cookie SameSite ảnh hưởng hành vi
│   └── Attack flow
│       ├── Nạn nhân đăng nhập vào ứng dụng
│       ├── Attacker dụ nạn nhân truy cập trang độc
│       ├── Trang độc tự động gửi request tới ứng dụng
│       └── Server xử lý request như của nạn nhân
│
├── 3. Phân loại CSRF
│   ├── GET-based CSRF
│   │   └── Dùng thẻ img, iframe, link để gửi GET request
│   ├── POST-based CSRF
│   │   └── Dùng form auto-submit
│   ├── Login CSRF
│   │   └── Ép nạn nhân đăng nhập vào tài khoản của attacker
│   ├── Logout CSRF
│   │   └── Ép nạn nhân đăng xuất
│   ├── Stored CSRF
│   │   └── Payload lưu trên server, kích hoạt khi user xem
│   └── Blind CSRF
│       └── Không thấy response, chỉ cần request được thực thi
│
├── 4. Kỹ thuật khai thác
│   ├── HTML form auto-submit
│   ├── Thẻ img
│   │   └── <img src="https://target/action?param=value">
│   ├── iframe
│   ├── fetch/XHR
│   │   └── Cần CORS misconfig hoặc same-origin
│   ├── Method override
│   │   └── Dùng _method=POST hoặc header X-HTTP-Method-Override
│   ├── Chaining với XSS
│   │   └── XSS lấy token, gửi request hợp lệ
│   ├── Chaining với Clickjacking
│   │   └── UI redressing để lừa click
│   └── Chaining với CORS misconfig
│       └── Đọc response trái phép
│
├── 5. Bypass phòng thủ
│   ├── Token bypass
│   │   ├── Xóa token
│   │   ├── Để trống token
│   │   ├── Dùng token cũ
│   │   ├── Token không gắn với session
│   │   ├── Token yếu, đoán được
│   │   ├── Token leak qua Referer
│   │   └── Dùng XSS lấy token
│   ├── SameSite bypass
│   │   ├── SameSite=Lax + POST top-level navigation
│   │   ├── SameSite=Lax + GET
│   │   ├── SameSite=Strict bypass qua sibling domain
│   │   ├── SameSite=Strict bypass qua client-side redirect
│   │   └── SameSite=None không Secure
│   ├── Referer/Origin validation bypass
│   │   ├── Thiếu header
│   │   ├── Origin: null
│   │   ├── Subdomain bypass
│   │   ├── Regex bypass
│   │   └── Open redirect
│   ├── Custom header bypass
│   │   └── Nếu server không bắt buộc
│   └── CORS misconfig
│       └── Cho phép origin attacker đọc response
│
├── 6. Phòng chống
│   ├── CSRF Token
│   │   ├── Synchronizer Token Pattern
│   │   ├── Double Submit Cookie
│   │   ├── Per-request token
│   │   ├── Token gắn với session
│   │   ├── Token ngẫu nhiên đủ mạnh
│   │   └── Không leak token qua URL/Referer
│   ├── SameSite Cookie
│   │   ├── Strict
│   │   ├── Lax
│   │   ├── None + Secure
│   │   └── Hiểu behavior của browser
│   ├── Origin/Referer validation
│   │   ├── Kiểm tra Origin header
│   │   ├── Kiểm tra Referer header
│   │   └── Chỉ cho phép origin tin cậy
│   ├── Custom header
│   │   └── X-Requested-With, X-CSRF-Token
│   ├── Re-authentication
│   │   └── Yêu cầu mật khẩu cho hành động nhạy cảm
│   ├── CAPTCHA
│   ├── Không dùng GET cho state-changing
│   ├── Logout bằng POST
│   ├── Content-Type validation
│   │   └── application/json + custom header
│   └── Framework protections
│       ├── Django CSRF
│       ├── Rails protect_from_forgery
│       ├── Spring Security CSRF
│       └── Express csurf
│
├── 7. Phát hiện và kiểm thử
│   ├── Burp Suite
│   │   ├── Generate CSRF PoC
│   │   ├── Repeater
│   │   └── Scanner
│   ├── OWASP ZAP
│   ├── CSRFTester
│   ├── Kiểm tra token
│   │   ├── Có token không?
│   │   ├── Token có ngẫu nhiên không?
│   │   ├── Token có gắn session không?
│   │   └── Token có bị leak không?
│   └── Kiểm tra SameSite
│       └── Dùng browser devtools xem cookie attributes
│
├── 8. Lab thực hành
│   ├── PortSwigger Web Security Academy
│   │   ├── CSRF vulnerability with no defenses
│   │   ├── CSRF where token validation depends on request method
│   │   ├── CSRF where token validation depends on token being present
│   │   ├── CSRF where token is not tied to user session
│   │   ├── CSRF where token is tied to non-session cookie
│   │   ├── CSRF where token is duplicated in cookie
│   │   ├── SameSite Lax bypass via method override
│   │   ├── SameSite Strict bypass via client-side redirect
│   │   ├── SameSite Strict bypass via sibling domain
│   │   ├── SameSite Lax bypass via cookie refresh
│   │   ├── CSRF where Referer validation depends on header presence
│   │   └── CSRF with broken Referer validation
│   └── TryHackMe
│       └── CSRF room (nếu có)
│
├── 9. Liên quan
│   ├── XSS
│   │   └── XSS có thể bypass CSRF token
│   ├── SSRF
│   │   └── Server-side, khác hoàn toàn
│   ├── Clickjacking
│   │   └── UI redressing, thường kết hợp
│   ├── CORS
│   │   └── Misconfig có thể dẫn tới CSRF/đọc dữ liệu
│   └── SOP
│       └── Same-Origin Policy
│
└── 10. Thuật ngữ quan trọng
    ├── Synchronizer Token Pattern
    ├── Double Submit Cookie
    ├── SameSite
    ├── Origin
    ├── Referer
    ├── CORS
    ├── SOP
    ├── State-changing request
    ├── Idempotent methods
    ├── Anti-CSRF token
    ├── CSRF PoC
    ├── Login CSRF
    └── Logout CSRF
```


### note

> [!NOTE]
> can this request actually from user?

> [!NOTE] Title
> Cốt lõi vẫn là giả danh request hợp lệ.Cái website mà hacker dựng lên sẽ giả danh request hợp lệ--> bắt trình duyệt của victim gửi request đó 


> [!NOTE] csrf token
> với các request nhạy cảm , sửa thông tin, để tránh bị giả mạo reuest khi victtim bị csrf thì người ta thêm một cái csrf token vào các request nhạy cảm đó
> 

> [!NOTE] Title
> GET và POST liên quan gì đến CSRF?
> - 

### LAB
1.Lab đầu tiên là lab THM về thực hành tấn công CSRF qua 1 cái web.
đại khái là nó sẽ bắt mình viết 1 trang web mà sẽ gửi cái POST request giả danh cái POST request đăng nhập của trang đó==> sau đó email của nạn nhân đã được đổi thành email của attacker
[[THM_LAB_staffthub]]

2-> lab 2 này nó sẽ host 1 cái trang web==> người dùng click vào nó sẽ gửi request+token vơi role user
=> mấu chốt là cái token này quá dễ đoán(chỉ mã hóa role bằng base 64)
==> do đó có thể dễ dàng giả danh request
[[THM_LAB_weak_csrftoken]]
