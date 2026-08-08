This lab contains a DOM-based cross-site scripting vulnerability in the search blog functionality. It uses an `innerHTML` assignment, which changes the HTML contents of a `div` element, using data from `location.search`.

To solve this lab, perform a cross-site scripting attack that calls the `alert` function.


-khi nhập ô tìm kiếm thì ta thấy input đi vào qua url ?search(location.search) 
![[Pasted image 20260725093658.png]]
![[Pasted image 20260725093644.png]]
trong script thì ta cũng thấy nó lấy query=location.sear, làm tham số đầu vào cho hàm tìm kiếm . nhìn thêm thì <u>element được innerhml</u> là 1 thẻ span

-đọc thêm về thẻ span thì nó cho phép chạy script bên trong
![[Pasted image 20260725093614.png]]


>tại sao ban đầu input <script>alert(1)</script> thì nó ko chạy được?
>mà phải input là <img src=1 onerror=alert(1)> ??
>lý do là thẻ span ko cho chạy thẳng script mà ta phải chèn script vào các thẻ có *error handler* 

-kết quả như hình
![[Pasted image 20260725095234.png]]