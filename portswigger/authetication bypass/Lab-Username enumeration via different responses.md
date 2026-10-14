![[Pasted image 20261002171532.png]]
-mô tả lab: lab này có 1 account với username/password predict được trong list mẫu của portswigger. Mình cần enumerate username đúng rồi brute password.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-different-responses
> -wordlist: [[lys thuyet authentication]] phần username enumeration

### hướng giải
- [ ] mình login thử bừa 1 cặp `test:test`, sau đó quan sát gói tin trả về, mình thấy nó báo `Invalid username or password.`
- [ ] mình login thử với username dài ngoằng không tồn tại, mình vẫn thấy câu đó
- [ ] => giả thuyết của mình là: khi username đúng mà password sai thì câu báo lỗi sẽ khác
- [ ] nên mình sẽ brute username trước, lọc response nào khác với bình thường

### giải
-bước 1: mình bắt request login bằng Burp rồi lưu thành file `login.txt` như này:
```
POST /login HTTP/2
Host: LAB-ID.web-security-academy.net
Cookie: session=SESSION_CUA_MINH
Content-Type: application/x-www-form-urlencoded

username=USER&password=test
```
-mình để password là `test` cố định, chỉ fuzz `USER` thôi.

-bước 2: mình chạy ffuf để enumerate username:
```bash
ffuf -request login.txt -request-proto https -w usernames.txt:USER -fr "Invalid username or password"
```
-sau khi quan sát kết quả này, mình thấy chỉ có 1 username trả về response khác hẳn, không chứa câu `Invalid username or password` kia. Đó là `carlos`.
![[auth-enum-different-responses-ffuf.png]]
-> nên mình chốt username đúng là `carlos`.

-bước 3: có username rồi, mình đổi file `login.txt` thành:
```
username=carlos&password=PASS
```
-rồi mình chạy:
```bash
ffuf -request login.txt -request-proto https -w passwords.txt:PASS -fr "Incorrect password" -fc 200
# mình lọc ngược: chỉ giữ response nào KHÔNG chứa "Incorrect password"
```
-sau khi quan sát gói tin này, mình thấy khi password đúng thì server trả về `302` redirect sang `/my-account`, còn sai thì vẫn `200` + câu `Incorrect password`. Nên mình lọc theo status cũng được:
```bash
ffuf -request login.txt -request-proto https -w passwords.txt:PASS -mc 302
```

-kết quả mình tìm được là `carlos:montoya` (hoặc 1 password trong list, tùy instance lab).
![[auth-enum-different-login-success.png]]

-mình login tay lại bằng `carlos:montoya` là solve lab.
