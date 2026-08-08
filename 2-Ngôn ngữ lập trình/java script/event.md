-**envent** : là thứ giúp trang web có thể lắng nghe và phản hồi khi người dùng clik chuột, cuộn trang, gõ phím ....
-Như trước mình học , thì mình thườg dùng *button onclick*  .Khi mình dùng **event**
thì code sẽ clean hơn
###  **addEventListener()** -Cấp phép lắng nghe

Đây là cách hiện đại để gắn sự kiện.  Cú pháp: 
```js
phần_tử.addEventListener('tên_sự_kiện', hàm_xử_lý)
```

-Với <font color="#fac08f">tên_sự_kiện</font>: ta có các sự kiện hay dùng sau:
	- **`click`**: Xảy ra khi click chuột.
	- **`input`**: Xảy ra **ngay lập tức** mỗi khi bạn gõ thêm/xóa bớt 1 ký tự trong ô input. (Dùng làm tính năng Search Realtime trên các web phim, web mua sắm).
	- **`submit`**: Xảy ra khi bạn bấm nút Gửi hoặc nhấn phím Enter bên trong một thẻ `<form>`.
    
-với <font color="#31859b">hàm_xử_lý</font> : ta cần nhớ các hàm hay dùng sau :
	Hiểu đơn giản **e**  nó đại diện cho đối tượng HTML cần gán sự kiện.
	Người ta thường viết tắt nó là **`e`** (Event Object).
	*cứ nhìn ví dụ bên dưới là sẽ hiểu :D*
	- **`e.target`**: Chỉ đích danh phần tử HTML nào vừa bị tác động.
	- **`e.preventDefault()` (Phanh gấp):** Trình duyệt có những hành vi mặc định (như bấm submit form thì tự động F5 reload lại trang). Lệnh này giúp **chặn đứng** sự reload đó để JS tự tay xử lý ngầm.
    


>*nói chung là đọc lý thuyết có vẻ hơi khó hiểu nên ta sẽ nhìn code thực tế để hiểu luôn*

``` html
<form id="form-tim-kiem">
    <input type="text" id="o-nhap" placeholder="Tìm sản phẩm...">
    <button type="submit">Tìm</button>
</form>
<h3 id="ket-qua"></h3>

<script>
    let form = document.getElementById("form-tim-kiem");
    let oNhap = document.getElementById("o-nhap");
    let ketQua = document.getElementById("ket-qua");

    // 1. Sự kiện 'input': Kích hoạt ngay lập tức mỗi khi bạn gõ thêm 1 ký tự
    oNhap.addEventListener("input", (e) => {
        // e.target đại diện cho chính ô input. 
        // e.target.value lấy ra chữ bạn vừa gõ.
        ketQua.innerText = `Gợi ý: ${e.target.value}...`;
    });

    // 2. Sự kiện 'submit': Kích hoạt khi bấm nút hoặc nhấn Enter
    form.addEventListener("submit", (e) => {
        e.preventDefault(); // BẮT BUỘC CÓ: Chặn trình duyệt reload lại trang
        
        ketQua.innerText = `✅ Đang tải kết quả cho từ khóa: ${oNhap.value}`;
        ketQua.style.color = "green"; 
    });
</script>
```
> *-> nhìn từ đoạn script trên mình có thể thấy một điều:* 
> 	- thẻ_html.**addEventLister()** : nó sẽ tạo một sự kiện , khi cái thẻ_html đấy được click hay được nhập...thì nó sẽ kích hoạt cái hành vi(hàm) bên trong cái dấu ngoặc () của addEventListenner()
>     - nhìn vào ví dụ và xem **e.target** và **e.preventDefaut** có tác dụng gì  :D ??
>         -> ... tự trả lời mỗi lần ôn tập lại 



### 💻 Task Thực hành: Form Đăng Nhập Không Reload

``` html
<form id="form-login">
    <input type="text" id="tai-khoan" placeholder="Nhập tài khoản...">
    <button type="submit">Đăng nhập</button>
</form>
```

**Yêu cầu code JS:**

1. Tìm thẻ form và ô input lưu vào biến.
    
2. Dùng `addEventListener` gắn sự kiện `input` vào ô tài khoản. Truyền vào một hàm (có tham số `e`). Bên trong hàm, in ra console nội dung người dùng đang gõ bằng `e.target.value`.
    
3. Dùng `addEventListener` gắn sự kiện `submit` vào thẻ form.
    
4. Việc ĐẦU TIÊN trong sự kiện `submit` là dùng `e.preventDefault()` để chặn form reload lại trang.
    
5. Việc tiếp theo, in ra màn hình: `"Gửi dữ liệu lên server thành công!"`
