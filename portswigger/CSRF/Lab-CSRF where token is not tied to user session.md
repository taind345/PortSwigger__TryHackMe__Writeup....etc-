![[Pasted image 20260927002848.png|700]]
![[Pasted image 20260927143010.png]]
-> csrf token có ở trong request nhưng ko được lưu trên sesion để xác thực , đại khái nó để cho có
-thử xóa hay thay đổi csrs token cũng ko được=> có lẽ là do nó vẫn check csrf token, nhưng nó ko phân biệt csrf của user này với user khác bằng sessions
=> [1] ko được, dùng token của wreiner cho carlos để đổi email thì nó 400 bad request

-đăng nhập bằng carlos , nhưng gửi cái reuqest change email cũ của wreiner vẫn ok==> token csrf vẫn duy trì lâu , vậy mấu chốt ở đâu?

![[Pasted image 20260928111154.png]]==> mấu chốt ở đây ở cái [1] trên kia mình ko được là do , lúc mình gửi mình đã đăng xuất wreiner, nên caí session token của carlos nó đóng rồi, nên mình mới ko dùng được==> bây giờ check lại xem bước [1] khi cả 2 session vẫn đang mở [2]
=> vẫn ko được
=> mình phát hiện ra là cái token này là single-use, khi mình cố gửi lại cái request change email cho carlos ![[Pasted image 20260928111806.png]]

=> **do đó**
- [ ] token dùng 1 lần
- [ ] server ko check token-user session có khớp nhau ko?

![[Pasted image 20260928112206.png]]
-> đúng là như vậy , mỗi lần đổi email là 1 token khác nhau


-vấn đề đã rõ, 
- [ ] cần đưa token của wreiner vào trong request của carlos
	- [ ] mà ko được trigger cái token này
		- [ ] ==> đơn giản là inspect cái nút submit để xem token .Phần này đã học lý thuyết ở phần [[csrf token]]
		- [ ] ![[Pasted image 20260928112924.png|516]]
		- [ ] đưa cái token này vào request của carlos thôi 
- [ ] <u>-> ok được rồi, token của wreiner , đã được dùng cho request của carlos thành công</u>![[Pasted image 20260928112830.png]]


-ok bây giờ host cái exploit website thôi, đơn giản là lấy cái token chưa được sử dụng , gắn vô cái request change-email
``` html
<html>
  <body>
    <form action="https://0a8f00070335e06d82d8b066003b0001.web-security-academy.net/my-account/change-email" method="POST">
        <input type="hidden" name="email" value="hacker@evil.com">
        <input type="hidden" name="csrf" value="ybW4cs62Cv4Sns6pWwpxbT0kV27PDpo8">
    </form>
    <script>
        document.forms[0].subSmit();
    </script>
  </body>
</html>
```
![[Pasted image 20260928113323.png]]