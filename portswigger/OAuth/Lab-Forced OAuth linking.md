![[oauth-link.png]]
-mô tả lab: chức năng link social vào account blog thiếu `state` nên dính CSRF. Mình lừa admin click link chứa code của mình để account admin bị link vào social mình, rồi mình login social là vào admin.

> [!NOTE]
> -Lab: https://portswigger.net/web-security/oauth/lab-oauth-forced-oauth-profile-linking

### hướng giải
- [ ] mình vào trang link account (`/my-account/link`...), bấm link social, đi hết luồng, provider trả code về callback của blog
- [ ] sau đó quan sát gói tin callback này, mình thấy nó không có `state` gì cả, cứ ai cầm code hợp lệ GET vào là nó link social đó vào session hiện tại
- [ ] => mình lấy code của chính mình (chưa dùng), rồi dụ admin click URL callback gắn code đó. Admin đang login nên account admin bị link vào social của mình. Xong mình login social bằng account mình là vào admin blog.

### giải
-bước 1: mình mở luồng link social nhưng dừng ở bước provider trả code, copy code:
```
GET /oauth-callback?code=CODE_CUA_MINH
```
-mình KHÔNG gửi request này bằng session mình, mình giữ code lại.

-bước 2: mình dựng exploit cho admin click (dùng exploit-server, hoặc iframe auto):
```html
<iframe src="https://LAB-ID.web-security-academy.net/oauth-callback?code=CODE_CUA_MINH"></iframe>
```
![[oauth-link-iframe.png]]
-admin đang login blog nên request này mang session admin -> server link social-của-mình vào account admin.

-bước 3: mình bấm "Login with social" bằng account mình -> server thấy social này đã link với admin -> cho mình vào `/my-account?id=administrator` là solve.

> [!NOTE]
> -bài học của mình: luồng OAuth nào cũng cần `state` random gắn session để chống CSRF, kể cả luồng link account. Thiếu state là attacker link bừa được.
