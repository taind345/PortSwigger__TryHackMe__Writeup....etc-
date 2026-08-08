This lab contains a DOM-based cross-site scripting vulnerability in the stock checker functionality. It uses the JavaScript `document.write` function, which writes data out to the page. The `document.write` function is called with data from `location.search` which you can control using the website URL. The data is enclosed within a select element.
>ở đây đề bài nói là data đi từ searrch url --> biến đổi --> đi vào document.write() để in lên màn hình 

To solve this lab, perform a cross-site scripting attack that breaks out of the select element and calls the `alert` function.



-phương thức POTS này giúp truyền tham số của storeID và poductID để kiểm tra giỏ hàng
![[Pasted image 20260725082621.png|500]]

-mã js giúp xử lý việc kiểm tra giỏ hàng.Lấy tham số *storeid* từ url qua hàm *locatio.search*, tiếp đó dùng thàm số đó để truy vấn, và ghi ra màn hình qua *document.write* 
![[Pasted image 20260725083147.png]]
-có thể thấy bên trên url , mã js lấy document.search dựa trên url
![[Pasted image 20260725083034.png|400]]
-payload ta dùng để bypass cái thẻ option
```html
<option selected>
-- bat dau payload
</option>
	<script>
		alert(document.domain) ;
	</script>
<option selected>
-- ket thuc payload 
</option>
```
-ta truyền vào url tham số storeid=
![[Pasted image 20260725091506.png]]
