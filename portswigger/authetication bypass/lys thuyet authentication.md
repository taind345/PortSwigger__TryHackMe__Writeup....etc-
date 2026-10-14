# Lý thuyết authentication - mình tổng hợp lại

-sau khi học hết phần này, mình hiểu là: authentication là chứng minh "mình là ai" (username/password), còn authorization là "mình được làm gì" sau khi đã login.
- auth luôn đứng trước authz. Bypass được auth là coi như xong phim, vào được hết.

## 1. Username enumeration - 3 cách mình hay dùng
- [ ] cách 1: khác câu báo lỗi. User sai báo kiểu A, user đúng + pass sai báo kiểu B. -> [[Lab-Username enumeration via different responses]]
- [ ] cách 2: khác tinh vi 1 ký tự (thừa/thiếu dấu `.`). Nhìn mắt thường không ra, phải soi raw response hoặc diff length. Trước mình còn tưởng lab lỗi, hóa ra là mình nhìn không kỹ. -> [[Lab-Username enumeration via subtly different responses]]
- [ ] cách 3: khác thời gian phản hồi. User đúng thì server phải hash password nên chậm hơn. Mình dùng password dài 100 ký tự `aaa...` cho delay nó rõ, rồi lọc bằng `ffuf -ft`. -> [[Lab-Username enumeration via response timing]]
- [ ] cách 4: qua account lock. User đúng thì sai n lần là bị lock + báo `too many incorrect attempts`, user sai thì báo mãi 1 câu. Mình dùng chính cái lock này để enumerate. -> [[Lab-Username enumeration via account lock]]

## 2. Bypass bảo vệ brute-force - mấy trò mình gặp
- [ ] reset counter bằng login đúng xen kẽ. Pattern: `wiener:peter` -> `carlos:guess` -> `wiener:peter`... đi 1 luồng. -> [[Lab-Broken brute-force protection, IP block]]
- [ ] nhét nhiều password vào 1 request (trường `password` là JSON array). Đếm theo request thì chết với trò này. -> [[Lab-Broken brute-force protection, multiple credentials per request]]
- [ ] đổi IP qua header `X-Forwarded-For` (chỉ đúng ở lab timing, lab IP block thì KHÔNG dùng trò này). -> [[Lab-Username enumeration via response timing]]
- [ ] brute ở endpoint khác không bị limit, vd `/my-account/change-password`. -> [[Lab-Password brute-force via password change]]

> [!NOTE]
> -bài học của mình: không bao giờ tin `X-Forwarded-For` để rate-limit; đếm thì phải đếm theo số lần thử password, không đếm theo request; chỗ nào check password thì chỗ đó phải limit như nhau.

## 3. 2FA
- [ ] bypass thẳng: login xong nó đá sang `/login2` nhưng mình vào thẳng `/my-account` thì nó cho vào luôn, không thèm check đã nhập 2FA chưa. -> [[Lab-2FA simple bypass]]
- [ ] brute mã 4 số (0000-9999): server chỉ logout sau vài lần sai chứ không khóa cứng, nên mình dùng macro/script auto-login lại rồi brute tiếp. -> [[Lab-2FA bypass using a brute-force attack]]

## 4. Cookie remember-me / stay-logged-in
- [ ] công thức ngớ ngẩn `base64(username:md5(password))`. Mình decode ra là thấy ngay, rồi brute md5 của carlos là xong. md5 không salt thì tra rainbow 1 phát là ra. -> [[Lab-Brute-forcing a stay-logged-in cookie]]
- [ ] mã hóa cookie chung key với chức năng khác (comment mã hóa email). Mình dùng comment như cái máy mã hóa (oracle), cắt block AES-CBC chứa `administrator` rồi ghép thành cookie admin. -> [[Lab-Authentication bypass via encryption oracle]]

## 5. Reset password
- [ ] không check token khớp user: submit form reset của mình nhưng đổi `username=carlos` là đổi được password carlos. Token phải gắn với đúng user, lúc submit phải check lại cả 2. -> [[Lab-Password reset broken logic]]
- [ ] tin `X-Forwarded-Host`: server lấy host từ header mình gửi để tạo link reset, nên mình đổi host thành exploit-server là cướp được token của carlos. Không bao giờ lấy host từ header client để tạo link quan trọng. -> [[Lab-Password reset poisoning via X-Forwarded-Host]]

## 6. Bypass bằng lộ thông tin
- [ ] method `TRACE` echo lại request làm lộ header nội bộ `X-Custom-IP-Authorization`. Mình copy header đó + giá trị `127.0.0.1` là vào được `/admin`. Nên tắt `TRACE` trên production. -> [[Lab-Authentication bypass via information disclosure]]

## 7. JWT - mình note ngắn (lab JWT nằm topic riêng)
-sau khi học phần này, mình hiểu luồng server verify là:
```
nhận token -> tách 3 phần header.payload.signature -> decode base64
-> đọc alg -> ký lại header.payload bằng secret -> so sánh chữ ký
-> check claims (exp, iss, aud) -> cho vào
```
- [ ] `alg:none`: sửa header thành `{"alg":"none"}`, xóa signature. Server nào không whitelist alg là dính.
- [ ] RS256 -> HS256: đổi `alg` thành HS256 rồi ký bằng **public key** của server (lấy ở `/jwks.json`) như thể nó là HMAC secret.
- [ ] secret yếu (`secret1`...): `hashcat -m 16500` brute rồi tự ký token admin.
- [ ] `kid` injection: `kid` là đường dẫn file (`../../../../dev/null` = secret rỗng) hoặc SQLi/command injection.
- [ ] vì sao hệ thống lớn dùng RS256: nhiều server verify bằng public key, chỉ 1 server giữ private key để ký. Mình thấy hợp lý.

## 8. OAuth - mình note ngắn (lab OAuth riêng)
- [ ] `redirect_uri` bypass: whitelist bằng regex ẩu (`app.com.evil.com`, `app.com@evil.com`) -> code/token bay về evil.com.
- [ ] thiếu `state`: attacker lấy code của mình rồi dụ admin click `/callback?code=...` để link tài khoản admin vào OAuth của attacker.
- [ ] leak code qua Referer (trang callback load tài nguyên ngoài) hoặc open redirect.

## 9. Phòng chống - mình tự nhắc
- [ ] rate-limit theo cả account + IP, CAPTCHA sau vài lần sai, không tin header client.
- [ ] token reset random, có hạn, gắn đúng user, check lại lúc submit.
- [ ] JWT: whitelist `algorithms`, secret mạnh, validate `kid`, check `exp/iss/aud`, không bao giờ nhận `none`.
- [ ] 2FA check ở server mọi luồng (kể cả OAuth/OIDC), mã ngắn thì phải lock cứng.
- [ ] tắt `TRACE`, không dùng 1 key mã hóa cho 2 chức năng, cookie `HttpOnly + Secure + SameSite`.
- [ ] query DB dùng prepared statements.

## 10. Mấy case thực tế mình thấy hay
- [ ] Microsoft Titan: JWT không thèm verify signature, sửa payload thành admin là vào, đọc được 25k account.
- [ ] Flowise (CVE-2025-58434): API reset trả luôn `tempToken` trong response, khỏi cần email.
- [ ] GitHub Enterprise (CVE-2026-4296): regex check `redirect_uri` ẩu -> leak JWT.
- [ ] Vikunja (CVE-2026-34727): luồng OIDC cấp JWT mà quên check TOTP -> bypass 2FA.
- [ ] Fortinet FortiWeb (CVE-2025-64446): path traversal + tin header `CGIINFO` -> tạo admin.
- [ ] M365: botnet 130k máy password spraying qua basic auth (không có MFA) -> tắt basic auth đi.
- [ ] BigBear 2.0 AiTM: phishing đứng giữa đánh cắp session sau MFA -> MFA thường không chống được, phải FIDO2/passkey.

> [!NOTE]
> -wordlist mẫu portswigger: usernames + passwords ở trang authentication (mục auth-lab-usernames / auth-lab-passwords).
> -thứ tự lab mình đi ở file index: [[0-authentication]]
