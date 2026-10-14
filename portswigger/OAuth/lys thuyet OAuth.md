# Lý thuyết OAuth - mình tổng hợp lại

-sau khi học phần này, mình hiểu là: OAuth cho phép login bằng tài khoản bên thứ 3 (Google/Facebook/PortSwigger social). Luồng chuẩn: client -> OAuth provider (lấy `code`) -> client đổi `code` lấy `token` -> lấy info user -> login.
-lỗi toàn nằm ở: `redirect_uri` ẩu, thiếu `state`, leak `code/token`.

## 1. Luồng mình vẽ lại cho nhớ
```
1. mình bấm "Login with Google" -> client redirect mình sang:
   /authorize?client_id=...&redirect_uri=https://client/callback&response_type=code&scope=email&state=RANDOM
2. mình login ở Google, Google redirect về:
   https://client/callback?code=ABC&state=RANDOM
3. client mang code + secret đổi lấy token ở Google, rồi GET info (/me?access_token=...)
4. client thấy email mình là ai thì cho login vào account đó
```
- [ ] bước 2 và 4 là 2 chỗ chết: code về sai địa chỉ, hoặc client tin email không verify.

## 2. Các đòn mình đã làm qua
- [ ] implicit flow tin email client gửi: POST `/authenticate` với `email` mình tự ghi, đổi thành `carlos@...` là vào account carlos. -> [[Lab-OAuth implicit flow]]
- [ ] thiếu `state` -> CSRF link tài khoản: mình lấy code của mình rồi dụ admin click `/oauth-link?code=CODE_MINH`, account admin bị link vào social của mình, mình login social là vào admin. -> [[Lab-Forced OAuth linking]]
- [ ] `redirect_uri` ẩu: sửa thành domain mình để cướp `code` của admin, rồi dùng code đó login vào admin. -> [[Lab-OAuth redirect_uri hijack]]
- [ ] open redirect ở client + redirect_uri: đẩy `token` về server mình qua redirect trung gian. -> [[Lab-OAuth open redirect steal]]
- [ ] OpenID dynamic registration SSRF: endpoint đăng ký client nhận `logo_uri` URL, server tự fetch -> SSRF vào nội bộ. -> [[Lab-OpenID dynamic registration SSRF]]

## 3. Phòng chống - mình tự nhắc
- [ ] whitelist `redirect_uri` tuyệt đối (scheme+host+path), không regex ẩu.
- [ ] bắt buộc `state` random gắn session + check lại ở callback.
- [ ] `code` dùng 1 lần + hạn ngắn; không bỏ token/code vào URL log được (Referer).
- [ ] verify `token` ở server bằng server-to-server, đừng tin field client gửi (email).

> [!NOTE]
> -thứ tự lab mình đi ở file: [[0-index OAuth]]
