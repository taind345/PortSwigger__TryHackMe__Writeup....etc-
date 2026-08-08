[[các cách để bypass ssrf]]
#ssrf 
```Text
Vượt qua phòng thủ SSRF (Blacklist-based Filters)
│
├── Khái quát vấn đề
│   ├── Ứng dụng có lỗ hổng SSRF nhưng được trang bị bộ lọc bảo vệ.
│   └── Bộ lọc sử dụng "Danh sách đen" (Blacklist) để chặn các chuỗi như:
│       ├── 127.0.0.1
│       ├── localhost
│       └── /admin
│
└── Các kỹ thuật vượt qua (Circumvention Techniques)
    │
    ├── 1. Biểu diễn IP dưới dạng thay thế (Alternative IP representation)
    │   ├── Máy chủ backend vẫn hiểu đây là 127.0.0.1 nhưng bypass được bộ lọc chuỗi.
    │   ├── Dạng thập phân (Decimal): 2130706433
    │   ├── Dạng bát phân (Octal): 017700000001
    │   └── Dạng rút gọn: 127.1
    │
    ├── 2. Sử dụng tên miền tự tạo (Custom Domain Name)
    │   ├── Đăng ký một tên miền (domain) và cấu hình DNS để nó trỏ về IP 127.0.0.1.
    │   ├── Khi ứng dụng kiểm tra, tên miền không nằm trong blacklist.
    │   └── Ví dụ được cung cấp sẵn: spoofed.burpcollaborator.net
    │
    ├── 3. Che giấu chuỗi bị chặn (Obfuscation)
    │   ├── Mã hóa URL (URL Encoding): Chuyển đổi ký tự sang định dạng %xx (VD: %61dmin thay vì admin).
    │   └── Thay đổi kiểu chữ (Case variation): Viết hoa/viết thường xen kẽ (VD: lOcAlHoSt, aDmIn).
    │
    └── 4. Kỹ thuật chuyển hướng (Redirection)
        ├── Cung cấp cho ứng dụng một URL hợp lệ do bạn kiểm soát (không bị chặn).
        ├── Cấu hình URL của bạn để nó tự động chuyển hướng (redirect) máy chủ mục tiêu về 127.0.0.1.
        ├── Thử nghiệm các mã trạng thái chuyển hướng khác nhau (VD: 301, 302, 307).
        └── Thử nghiệm thay đổi giao thức trong quá trình chuyển hướng (VD: Từ http:// chuyển sang https://) để đánh lừa bộ lọc.
```
[[mã hóa url để bypass blacklist]]
