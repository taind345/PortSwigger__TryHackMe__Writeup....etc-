[[DOM-]]
### định nghĩa
-**DOM-Document object model** là cách javascript thao tác với HTMl.Giúp <u>đổi màu chữ </u>, <u>tạo hiệu ứng giỏ hàng</u> ,....
-Do java script là ngôn ngữ client-side nên nó phải có khả năng thao tác với HTML.Do đó DOM sinh ra để làm việc đó 
-*??* Tư duy học DOM là **tìm phần tử**-->**đổi nội dung**--> **thêm/xóa phần tử** *(1)*
-**document** : là toàn bộ nội dung HTML hiển thị trên trang web.
mọi thao tác DOM đều bắt đầu bằng chữ *document* 
-**window** toàn bộ cửa sổ trình duyệt....

-tại *(1)* ta có :
 - <u>Tìm kiếm phần tử</u> : Lấy thẻ HTML để xử lý 
	 - *muốn sơn nhà phải chỉ ra phần nào của ngôi nhà mà mình muốn sơn*
	 - **`getElementById('id_của_thẻ')`:** Tìm chính xác 1 thẻ thông qua `id`. (Tốc độ chạy nhanh nhất).
	- **`querySelector('.class' hoặc '#id')`:** Tìm thẻ y hệt như cách bạn viết CSS. Rất linh hoạt, được dùng nhiều nhất hiện nay.
	
- Sau khi đã chỉ ra được phần nào của HTML mà mình muốn thao tác.Bây giờ ta sẽ <u>thao tác</u> với cái phần đã được chọn đó-*bắt đầu sơn nhà* 
	- **`innerText` / `textContent`:** *Đổi* nội dung chữ bên trong thẻ.
	- **`innerHTML`:** *Đổi* chữ VÀ có thể *chèn* luôn thẻ HTML mới vào.
	- **`value`:** *Lấy* hoặc *đổi* dữ liệu mà người dùng gõ vào ô `<input>`.
	- **`setAttribute('thuộc_tính', 'giá_trị')`:** *Đổi* id, class, src (link ảnh), href (link web)...
*--> những cái trên là thuộc tính của *
- <u>Thêm & Xóa phần tử</u> (Nâng cao)
	- **`createElement('tên_thẻ')`:** Tạo ra một thẻ HTML mới, nhưng nó mới chỉ nằm trong RAM máy tính, **chưa hiện ra màn hình**.
	- **`appendChild(thẻ_con)`:** Bơm cái thẻ vừa tạo ở trên vào bên trong một thẻ cha có sẵn trên màn hình.
	- **`remove()`:** Xóa sổ hoàn toàn một thẻ khỏi màn hình.
--> *những cái này là phương thức của documents*


-Nhìn sơ qua có vẻ khó hiểu nên mình sẽ học qua ví dụ :
**dom.js**
```js
function themViec() {
	let oNhap = document.getElementById("o-nhap");
	let khungDanhSach = document.getElementById("danh-sach");
	let chuNguoiDungGo = oNhap.value;
	if (chuNguoiDungGo === "") return;
	let theMoi = document.createElement("li");
	// Dùng Template Literal và innerHTML để chèn cả chữ và nút xóa
	theMoi.innerHTML = `${chuNguoiDungGo} <button onclick="xoaViec(this)">delete task</button>`;
	khungDanhSach.appendChild(theMoi);
	oNhap.value = "";
}

function xoaViec(nutXoa) {
	// 1. Từ cái nút bấm, tìm ngược lên cái thẻ <li> chứa nó (thẻ cha)
	let theLiCanXoa = nutXoa.parentElement;
	// 2. Xóa thẻ <li> đó
	theLiCanXoa.remove();
}
```

**index.html**
``` html
<!-- Ô nhập việc cần làm -->
<!DOCTYPE html>
<html lang="vi">
	<head>
		<meta charset="UTF-8">
		<title>To-Do List</title>
	</head>
	<body>
		<input type="text" id="o-nhap" placeholder="Nhập việc cần làm...">
		<button onclick="themViec()">Thêm</button>
		<ul id="danh-sach"></ul>
		<!-- Dòng này dùng để NHÚNG file JS bên ngoài vào -->
		<script src="dom.js"></script>
	</body>
</html>
<!-- các id của các object ở đây là "o-nhap", "danh-sach", và hàm themViec() dùng DOM để thao tác với HTML-->
  
<!-- luồng hoạt động:
nhâpj text vào "o-nhap"
=>click button Thêm : Chạy hàm themViec()
=>DOM lấy text tu "o-nhap" cho ra the moi <li></li>
=>chen <li> </li> vào <ul id="danh-sach"
-->
```



### tong ket
Bạn đã nắm được "Tam giác vàng" thao tác với giao diện:
1. **Tìm kiếm:** `getElementById`, `querySelector`
    
2. **Biến đổi:** `innerText`, `innerHTML`, `value`, `setAttribute`
    
3. **Thêm/Xóa:** `createElement`, `appendChild`, `remove`
    
Chỉ với bấy nhiêu công cụ, kết hợp cùng Object/Array và Loop, bạn đã đủ sức làm được 80% các tính năng tương tác cơ bản trên web.