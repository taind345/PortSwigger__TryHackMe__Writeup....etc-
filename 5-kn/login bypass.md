Vượt qua cổng đăng nhập (Login Bypass)
│
├── 1. Thu thập thông tin (Recon)
│   ├── Xác định công nghệ
│   │   ├── Ngôn ngữ backend (PHP, ASP.NET, Node.js...)
│   │   ├── Framework (Django, Laravel, Spring...)
│   │   └── Database (MySQL, PostgreSQL, MSSQL...)
│   ├── Xem source code
│   │   ├── Comment lộ thông tin
│   │   ├── Endpoint ẩn trong JS
│   │   └── API routes
│   ├── Kiểm tra robots.txt, sitemap.xml
│   ├── Fuzz đường dẫn
│   │   ├── /admin, /dashboard, /panel
│   │   ├── /backup, /config, /.git
│   │   └── /api/v1, /api/v2
│   └── Xem response header
│       ├── Server, X-Powered-By
│       └── Cookie, Set-Cookie
│
├── 2. Tấn công SQL Injection
│   ├── Auth bypass cơ bản
│   │   ├── ' OR '1'='1
│   │   ├── ' OR 1=1 --
│   │   ├── admin' --
│   │   └── ' OR 'x'='x
│   ├── Union-based
│   │   └── ' UNION SELECT ... --
│   ├── Blind SQLi
│   │   ├── Boolean-based
│   │   └── Time-based
│   ├── NoSQL Injection
│   │   ├── {"username": {"$ne": null}}
│   │   ├── {"password": {"$ne": null}}
│   │   └── {"$where": "sleep(5000)"}
│   └── Bypass filter
│       ├── Comment: `/**/`, --, #
│       ├── Case: Or, oR, OR
│       └── Encoding: URL, hex
│
├── 3. Tấn công Credential
│   ├── Default credentials
│   │   ├── admin/admin
│   │   ├── admin/password
│   │   ├── root/root
│   │   └── Tra DefaultCreds-Cheat-Sheet
│   ├── Brute-force
│   │   ├── Hydra
│   │   ├── Medusa
│   │   ├── Burp Intruder
│   │   └── Wordlist: rockyou.txt, SecLists
│   ├── Password spraying
│   │   └── 1 password, nhiều username
│   ├── Credential stuffing
│   │   └── Dùng leak từ breach khác
│   └── Phân tích wordlist
│       ├── Tên công ty, sản phẩm
│       └── Pattern từ leak cũ
│
├── 4. Khai thác logic nghiệp vụ
│   ├── Response manipulation
│   │   ├── Sửa "success:false" → "success:true"
│   │   ├── Sửa status code 401 → 200
│   │   └── Sửa role trong response
│   ├── Parameter tampering
│   │   ├── Thêm "isAdmin=true"
│   │   ├── Sửa "user_id=1"
│   │   └── Hidden field trong form
│   ├── Array injection
│   │   └── username[]=admin&username[]=x
│   ├── Type juggling
│   │   ├── "0e..." == "0e..." (PHP magic hash)
│   │   └── true == "any string"
│   └── Race condition
│       └── Gửi nhiều request đồng thời
│
├── 5. Tấn công Session / Token
│   ├── Cookie manipulation
│   │   ├── Sửa role trong cookie
│   │   ├── Sửa user_id
│   │   └── Decode base64 cookie
│   ├── JWT attacks
│   │   ├── alg:none
│   │   ├── Weak secret (brute-force)
│   │   ├── Algorithm confusion (RS256 → HS256)
│   │   └── kid injection
│   ├── Session fixation
│   ├── Session prediction
│   └── Cookie forgery
│
├── 6. Lỗi logic Authentication
│   ├── Username enumeration
│   │   ├── So sánh thông báo lỗi
│   │   ├── So sánh thời gian response
│   │   └── So sánh status code
│   ├── Bypass 2FA
│   │   ├── Response manipulation
│   │   ├── Brute-force mã OTP
│   │   ├── Reuse OTP cũ
│   │   └── Skip bước xác thực
│   ├── Password reset flaw
│   │   ├── Token yếu, đoán được
│   │   ├── Host header injection
│   │   ├── Token leak qua Referer
│   │   └── Reset không cần xác thực email
│   ├── Remember me bypass
│   └── Insecure direct object reference
│
├── 7. Tấn công phía client
│   ├── Client-side validation bypass
│   │   ├── Sửa JS
│   │   ├── Disable JS
│   │   └── Gửi request trực tiếp
│   ├── XSS để đánh cắp cookie
│   ├── CSRF trên form login
│   ├── Login CSRF
│   └── Clickjacking
│
├── 8. Khai thác dịch vụ khác
│   ├── Database service (3306, 5432)
│   │   ├── Yếu mật khẩu
│   │   └── Anonymous access
│   ├── SSH (22)
│   │   ├── Key leak
│   │   └── Yếu mật khẩu
│   ├── SMB (445)
│   │   ├── Null session
│   │   └── Guest access
│   ├── Redis (6379)
│   │   └── Không auth
│   ├── API endpoint
│   │   ├── Không auth
│   │   └── Auth yếu
│   └── Backup file
│       ├── .bak, .old, .zip
│       └── Config lộ credential
│
├── 9. Lateral Movement
│   ├── Reuse password
│   │   └── Password dùng nhiều nơi
│   ├── SSH key leak
│   ├── Pivot qua máy khác
│   └── Leo quyền sau khi vào
│
├── 10. Công cụ hỗ trợ
│   ├── Burp Suite
│   │   ├── Intruder (brute-force)
│   │   ├── Repeater (test thủ công)
│   │   ├── Scanner (tự động)
│   │   └── Extension (JWT, Authz)
│   ├── Hydra
│   ├── SQLMap
│   ├── ffuf / Gobuster
│   ├── nmap
│   └── Metasploit
│
└── 11. Checklist thực hành
    ├── [ ] Xem source, header, robots.txt
    ├── [ ] Thử default credentials
    ├── [ ] Thử SQL injection cơ bản
    ├── [ ] Kiểm tra username enumeration
    ├── [ ] Test password reset flow
    ├── [ ] Phân tích cookie, JWT
    ├── [ ] Test logic (response manipulation)
    ├── [ ] Brute-force nếu cho phép
    ├── [ ] Tìm endpoint ẩn
    └── [ ] Kiểm tra dịch vụ khác trên host