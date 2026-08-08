-đaị khái là nó giống hệt như các ngôn ngữ lập trình khác thôi, khai báo trong hàm thì chỉ dùng được trong hàm đấy,khai báo
``` js
let user = "Toàn cục"; // Global

function kiemTra() {
    let matKhau = "123"; // Function Scope
    if (true) {
        let canhBao = "Lỗi!"; // Block Scope
        console.log(matKhau); // OK
    }
    // console.log(canhBao); // BÁO LỖI: canhBao is not defined
}
```
==> 
- global crope
- function scope
- block scope