### function
``` js
function tinhTien(Gia, SoLuong) { // Gia, SoLuong là parameters
        let tong = Gia * SoLuong;
        return tong;                  // Trả kết quả ra ngoài
    }
let bill = tinhTien(50, 2);       // bill = 100
```

### Callback (Giới thiệu):
-Là một function được truyền vào một function khác dưới dạng tham số. Tư duy: "Làm xong việc A đi, rồi lấy kết quả gọi hàm B".
*--> đại khái là để gọi được hàm bên trong một hàm khác ta phải truyền tham số*
```js
// Định nghĩa sẵn hàm callback
function diNgu() {
    console.log("Xong việc rồi, đi ngủ thôi!");
}

function anMi(hanhDongSauDo) {
    console.log("Đang ăn mì...");
    hanhDongSauDo();
}
// Khi truyền vào, CHỈ TRUYỀN TÊN HÀM (không có dấu ngoặc tròn diNgu() )
anMi(diNgu);
```

-nếu mà không muốn sử dụng hàm cũ, thì ta có thể viết thằng hàm trong lúc truyền
``` js
function anMi(hanhDongSauDo) {
    console.log("Đang ăn mì...");
    hanhDongSauDo();
}
// Viết thẳng hàm vào ngay lúc truyền
anMi( () => { 
    console.log("Xong việc rồi, cày rank tiếp!"); 
});
```
