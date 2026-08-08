### đề bài
This lab contains a DOM-based cross-site scripting vulnerability in the search query tracking functionality. It uses the JavaScript `document.write` function, which writes data out to the page. The `document.write` function is called with data from `location.search`, which you can control using the website URL.
To solve this lab, perform a cross-site scripting attack that calls the `alert` function.

-đầu tiên ta nhập input vào trong thanh tìm kiếm
![[Pasted image 20260724151558.png]]

-nhìn mã js ứng với trang kết quả trả về, ta thấy được , tham số được truyền qua search , qua url parametter, ta dự đoán nó sẽ lấy từ locaton.search
![[Pasted image 20260724152654.png]]

-tiếp tục nhìn vào mã js, có thể thấy biến *query* sẽ là biến đi vào *document.write(mã js)*, 
![[Pasted image 20260724152855.png]]


ok bây giờ mình sẽ bypass thằng này để bỏ cái thẻ img kia đi, mình sẽ dùng payload là:
```
query = "><script>alert(1)</script>
```
thì kết quả có thể trở thành:
``` js
<img src="/resources/images/tracker.gif?searchTerms=">
<script>alert(1)</script>
">
```
![[Pasted image 20260724154812.png]]