# Lý thuyết JWT - mình tổng hợp lại

-sau khi học phần này, mình hiểu là: JWT là 3 cục base64 nối bằng dấu chấm `header.payload.signature`. Server không lưu session mà mỗi request tự verify lại token bằng cách ký lại rồi so sánh.
-chi tiết dài mình từng note ở [[../authetication bypass/lys thuyet authentication#7. JWT - mình note ngắn (lab JWT nằm topic riêng)]], file này mình viết riêng cho topic JWT + lab.

## 1. Cấu trúc JWT, mình decode thử 1 cái là hiểu ngay
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9  <- header:  {"alg":"HS256","typ":"JWT"}
eyJzdWIiOiJ3aWVuZXIiLCJleHAiOjE3MzU2ODk2MDB9 <- payload: {"sub":"wiener","exp":...}
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJVadQssw <- signature
```
- [ ] header: khai báo thuật toán ký (`alg`) + loại token. **Server mà tin mù quáng vào `alg` này là toang.**
- [ ] payload: claims (`sub` là username, `exp` hạn, `iss` ai cấp, `aud` cho ai). Base64 decode là đọc được, **đừng để secret ở đây**.
- [ ] signature: `HMACSHA256(base64(header) + "." + base64(payload), secret)`. Đây là lớp bảo vệ duy nhất.

## 2. Luồng server verify (mình tự vẽ lại cho nhớ)
```
nhận token -> tách 3 phần -> decode header/payload
-> đọc alg (+ kid/jwk nếu có) -> lấy key tương ứng
-> ký lại header.payload -> so sánh chữ ký
-> check exp/iss/aud -> tin payload (sub=...) -> cấp quyền
```
- [ ] lỗi nằm ở 2 chỗ: **tin header** (alg/kid/jwk) và **key yếu**.

## 3. Các đòn mình đã làm qua
- [ ] secret yếu -> `hashcat -m 16500` brute rồi tự ký token admin. -> [[Lab-JWT weak signing key]]
- [ ] `alg:none` / bỏ signature -> server ẩu vẫn nhận. -> [[Lab-JWT flawed signature verification]]
- [ ] RS256 -> HS256: lấy public key ở `/jwks.json` làm HMAC secret để ký. -> [[Lab-JWT algorithm confusion]]
- [ ] nhúng `jwk` vào header: tự tạo cặp key, nhét public key mình vào token, ký bằng private key mình. -> [[Lab-JWT jwk injection]]
- [ ] `kid` là đường dẫn file -> path traversal về `/dev/null` (secret rỗng). -> [[Lab-JWT kid path traversal]]
- [ ] `kid` ghép vào SQL -> UNION SELECT ra secret do mình chọn. -> [[Lab-JWT kid SQL injection]]

## 4. Tool mình dùng
- [ ] Burp extension **JWT Editor Keys**: decode/sửa/ký lại + có sẵn nút attack (none, confusion, embedded jwk, kid...). Lab JWT mà không có nó thì cực.
- [ ] `hashcat -m 16500 jwt.txt rockyou.txt` để brute secret HS256.
- [ ] `jwt.io` để decode nhanh + test ký tay.
- [ ] python `PyJWT` khi cần script hàng loạt.

## 5. Phòng chống - mình tự nhắc
- [ ] whitelist `algorithms` cứng ở server (chỉ nhận đúng 1 alg), không bao giờ nhận `none`.
- [ ] secret HS256 dài random; RS256 thì tách bạch key verify (public) và key ký (private), đừng dùng chung 1 biến.
- [ ] `kid` whitelist, không ghép path/SQL. Không hỗ trợ `jwk` nhúng từ client.
- [ ] check `exp/iss/aud` đầy đủ.

> [!NOTE]
> -thứ tự lab mình đi ở file: [[0-index JWT]]
