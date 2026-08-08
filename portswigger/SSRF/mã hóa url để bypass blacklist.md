![[nháp]]
[[nháp]]
#ssrf 
## 1. URL Encoding (Percent-Encoding) cơ bản

Mã hóa ký tự đặc biệt thành `%HH` (hex của mã ASCII).

- `http://evil.com/` → `http://evil.com/` (không thay đổi)
    
- Chặn dấu `.` hoặc `/`: encode `.` → `%2E`, `/` → `%2F`
    
    text
    
    http://evil%2Ecom/admin
    http:%2F%2Fevil.com
    
- Nhiều backend (Apache, Nginx) sẽ decode một lần trước khi xử lý, nên nếu filter chỉ kiểm tra ký tự thô, cách này có thể bypass.
    

---

## 2. Double URL Encoding

Encode ký tự `%` thành `%25` để sau khi server decode lần đầu, kết quả vẫn còn dạng mã hóa.

- `.` → `%2E` → encode `%` thành `%25` → `%252E`
    
- `/` → `%2F` → `%252F`
    
- Ví dụ:
    
    text
    
    http://evil%252Ecom/admin
    http:%252F%252Fevil.com
    
- Nếu backend decode 2 lần (một số framework hoặc reverse proxy) thì sẽ thành `http://evil.com/admin`. Filter chỉ decode 1 lần có thể bị bỏ qua.
    

---

## 3. Unicode/UTF-8 Encoding và Normalization

Sử dụng các biểu diễn Unicode tương đương:

- Ký tự `.` (U+002E) có thể dùng các dạng fullwidth: `．` (U+FF0E), hoặc các ký tự giống dấu chấm.
    
- Dấu gạch chéo: `/` (U+002F) → `⁄` (U+2044), `∕` (U+2215).
    
- Một số backend sẽ chuẩn hóa (normalize) các ký tự này thành dạng ASCII trước khi xử lý path, giúp vượt filter chặn ký tự ASCII thô.
    
- Kỹ thuật này thường dùng trong path traversal, SSRF.
    
    text
    
    http://evil.com∕admin
    http://evil．com/admin
    

---

## 4. Biểu diễn IP thay thế (trong SSRF hoặc chặn localhost)

Nếu blacklist chặn `127.0.0.1` hoặc `localhost`, có thể dùng các cách biểu diễn khác:

|Dạng|Ví dụ|Ghi chú|
|---|---|---|
|Decimal IP|`http://2130706433/`|127.0.0.1 = 127×256³ + 0×256² + 0×256 + 1|
|Octal IP|`http://0177.0.0.1/`|0177 = 127. Mỗi octet có thể dùng octal (thêm số 0 đứng đầu)|
|Hexadecimal IP|`http://0x7f.0.0.1/`|0x7f = 127. Có thể dùng `0x7f000001` (hex toàn bộ 32-bit)|
|Kết hợp|`http://0177.0.0.0x1/`|Kết hợp octal, decimal, hex|
|IPv6|`http://[::1]/`|Loopback IPv6|
|IPv6 đầy đủ|`http://[0:0:0:0:0:0:0:1]/`||
|IPv6 với IPv4 embedded|`http://[::ffff:127.0.0.1]/`||
|DNS rebinding|Dùng tên miền trỏ về 127.0.0.1 sau khi đã qua filter|Cần domain riêng|

---

## 5. Obfuscation Protocol và Authority

- Thay đổi scheme: `http` ↔ `https`, hoặc dùng scheme ít gặp như `gopher`, `file` (nếu backend hỗ trợ, ví dụ SSRF qua cURL).
    
- Chèn thêm port không cần thiết: `http://evil.com:80/` (vẫn hoạt động nếu port 80).
    
- Chèn dấu `@` để tạo ra userinfo giả, khiến parser hiểu phần trước `@` là credentials còn host là phần sau.
    
    text
    
    http://trusted.com@evil.com/admin
    http://trusted.com%40evil.com/admin  (nếu encode @)
    
    Nhiều filter chỉ kiểm tra hostname đầu tiên mà không phân tích đúng theo RFC.
    
- Sử dụng dấu `#` (fragment) hoặc `?` (query) để che giấu:
    
    text
    
    http://evil.com#trusted.com
    http://evil.com?url=http://trusted.com
    

---

## 6. Path Obfuscation (che giấu đường dẫn)

- Sử dụng path traversal: `../`, `..%2F`, `..%252F`, `....//`, `....\/` để thoát khỏi thư mục và truy cập file hoặc endpoint bị cấm.
    
- Dùng dấu `;` hoặc `%3B` cho path parameter (theo chuẩn cũ của Java/Tomcat):
    
    text
    
    /admin;/login
    /admin%3B/login
    
    Một số server coi `;` là phân tách path param và bỏ qua nó, trong khi filter thấy `/admin` và chặn.
    
- Chèn thêm dấu `/` dư thừa: `//admin`, `/./admin`, `/admin/.` – server thường chuẩn hóa bỏ qua.
    

---

## 7. Sử dụng URL Shortener hoặc Redirect

- Gửi link rút gọn ([bit.ly](https://bit.ly/), tinyurl) để vượt blacklist kiểm tra domain. Backend sẽ tự động resolve redirect nếu không kiểm tra đích đến cuối cùng.
    
- Dùng open redirect nội bộ: `http://trusted.com/redirect?url=http://evil.com` – nếu backend cho phép redirect và không validate tham số.
    

---

## 8. Case Variation

- Thay đổi chữ hoa/thường trong hostname hoặc path vì hostname không phân biệt hoa thường nhưng path có thể (tùy hệ điều hành). Blacklist thường dùng so khớp chính xác, không phân biệt hoa thường:
    
    text
    
    http://EVIL.COM/admin
    http://Evil.Com/Admin
    
- Đối với protocol: `HTTP://`, `hTTp://` – filter thường chỉ kiểm tra lowercase.
    

---

## 9. NULL Byte Injection (%00, \0)

- Đặt `%00` vào giữa chuỗi, một số ngôn ngữ (C, PHP cũ) cắt chuỗi tại NULL byte, khiến filter chỉ thấy phần trước đó, còn server xử lý tiếp phần sau.
    
    text
    
    /admin%00/index.html  => filter thấy "/admin" và chặn, server cắt thành "/index.html"
    
- Ngày nay ít hiệu quả trên nền tảng hiện đại nhưng vẫn đáng thử trên hệ thống cũ.
    

---

## 10. Sử dụng Bộ mã hóa khác nhau (Charset Mismatch)

- Gửi request với mã hóa không phải UTF-8, ví dụ UTF-7, UTF-16, Shift-JIS có thể khiến các ký tự được diễn giải khác nhau giữa filter và backend.
    
- Ví dụ: trong UTF-7, `<` được mã hóa thành `+ADw-`. Ít áp dụng với URL nhưng quan trọng với body hoặc header.
    

---

## 11. Kết hợp nhiều kỹ thuật

- Để vượt qua filter phức tạp, kết hợp double encoding + case mix + obfuscation IP:
    
    text
    
    http://0177.0.0.1%252Fadmin
    
    Filter giải mã 1 lần → `http://0177.0.0.1%2Fadmin`, IP octet octal chưa bị chặn, path vẫn encoded. Server decode lần nữa và truy cập `127.0.0.1/admin`.
    

---

## Lưu ý phòng thủ (cho người xây dựng filter)

Để chống lại các bypass trên, backend filter nên:

- Decode URL đầy đủ (và lặp cho đến khi không còn encoded) trước khi kiểm tra.
    
- Chuẩn hóa Unicode (NFC/NFD) và path (loại bỏ `..`, `.`, `/` thừa).
    
- Parse URL nghiêm ngặt theo RFC, tách riêng host, path, query.
    
- Không chỉ dùng regex đơn giản; sử dụng thư viện parse URL chuẩn.
    
- Kiểm tra IP sau khi phân giải DNS, không chỉ domain.
    
- Giới hạn scheme cho phép (chỉ http/https), chặn redirect đến internal IP.
    

---

Các kỹ thuật trên giúp hiểu rõ cách attacker vượt blacklist; hãy sử dụng có trách nhiệm. Nếu cần ví dụ cụ thể cho một tình huống SSRF, LFI hay SQLi nào, tôi có thể cung cấp thêm chi tiết.