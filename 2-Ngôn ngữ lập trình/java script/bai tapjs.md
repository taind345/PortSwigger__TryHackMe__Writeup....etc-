taidtaitaidtdifvcv

### Bài tập: Tính tiền và thông báo giao hàng
**Yêu cầu code:**
1. **(Scope):** Khai báo một biến toàn cục (Global Scope) `phiShip = 30000`.
2. **(Function & Scope):** Viết một hàm tên là `tinhTongTien(giaDonHang, hanhDongSauKhiTinh)`.
    - Bên trong hàm, khai báo một biến `tongTien` bằng tổng của `giaDonHang` và `phiShip` (Function Scope).
    - Gọi hàm callback `hanhDongSauKhiTinh` và truyền `tongTien` vào nó
3. **(Arrow Function & Callback):** Tiến hành gọi hàm `tinhTongTien`, truyền vào:
    - Mức giá đơn hàng: `100000`.
    - Một **Arrow Function** thực hiện việc in ra console: `"Thanh toán thành công! Tổng số tiền là: [số tiền] VNĐ"`.
        
```js
let phiShip=30000 ;
function tinhTongTien(giaDonHang , hanhDongSauKhiTinh){
// callback and function
let tongTien =giaDonHang+phiShip ;
hanhDongSauKhiTinh(tongTien) ;
}

tinhTongTien(100000,(x)=>{
console.log("Thanh toan thanh cong ! Tong so tien la " +x +" vnd");
})
```


### bài tập object và arrays 
Khi làm việc với API thực tế, dữ liệu trả về thường là một **Array chứa nhiều Object**. Hãy thử sức với bài toán thực tế sau:
**Đề bài:**
1. Tạo một mảng tên là `danhSachSanPham` chứa 3 Object. Mỗi Object đại diện cho một sản phẩm có 2 properties: `ten` (chuỗi) và `gia` (số). _(Gợi ý: Cho giá trị tùy ý, ví dụ 30000, 70000, 100000)_
    
2. Sử dụng hàm `filter()` để tạo ra một mảng mới tên là `sanPhamCaoCap`, chỉ chứa những sản phẩm có giá **lớn hơn 50000**.
    
3. Sử dụng `forEach()` trên mảng `sanPhamCaoCap` vừa tạo để in ra console tên của từng sản phẩm.
```js
let danhSachSanPham=[{ten:"coca" , gia :30000}, {ten:"pepsi" , gia: 70000}, {ten:"monster", gia:100000}] ;
let sanphamCaoCap=danhSachSanPham.filter(ham => ham.gia > 50000);
sanphamCaoCap.forEach(ham=>console.log('ten cua sp la : '+ ham.ten))
```
*==> hoặc ta có thể dùng class- cái khuôn , còn object là để chỉ hẳn 1 vật cụ thể*
``` js
// 1. Tạo "Cái khuôn" bằng từ khóa class
class SanPham {
    // Hàm khởi tạo bắt buộc tên là constructor
    constructor(name, price) {
        this.ten = name;   // Gán đúng bằng từ khóa this
        this.gia = price;
    }
}

let danhSachSanPham = [];

// 2. Dùng từ khóa "new" để đúc ra sản phẩm mới
let sp1 = new SanPham("coca", 30000);
let sp2 = new SanPham("pepsi", 70000);
let sp3 = new SanPham("monster", 100000);

danhSachSanPham.push(sp1, sp2, sp3); // Đẩy 1 lúc nhiều SP vào mảng cho gọn
console.log(danhSachSanPham);
```

### bài tập xử lý xâu 
 💻 Task Thực hành (Bảo mật thông tin khách hàng)
Trong thực tế, khi hiển thị lịch sử giao dịch, bạn thường phải che đi một phần số điện thoại của khách hàng. Hãy làm bài tập sau:
**Đề bài:** Bạn nhận được một chuỗi dữ liệu thô từ hệ thống:
```js
let data = "nguyen van a - 0987654321";
```
**Yêu cầu viết code:**
1. Dùng `split()` để tách chuỗi `data` trên thành một mảng gồm 2 phần (Tên và Số điện thoại). Gán nó vào 1 biến.
2. Từ mảng vừa tạo, lấy ra Số điện thoại. Dùng `slice()` để cắt lấy **3 số cuối cùng** của số điện thoại.
3. Dùng `replace()` để thay thế 3 số cuối đó trong chuỗi số điện thoại gốc thành `"***"`.
4. Dùng `Template literal` để in ra kết quả cuối cùng: `"Khách hàng nguyen van a có số điện thoại là 0987654***"`
```js
let data = "nguyen van a - 0987654321";
let arr=data.split("-");
let sdt=arr[1].slice(0,-3)+"***";
console.log(`khach hang ten ${arr[0]} co sdt la ${sdt}`);
```


