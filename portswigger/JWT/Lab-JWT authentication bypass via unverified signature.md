https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature
![[Pasted image 20261101180736.png]]
Mục tiêu của bài này là phải vào được `/admin` và xóa user `carlos`. Lỗ hổng ở đây là do server có lỗi trong khâu triển khai: ==🟡nó giải mã JWT để lấy thông tin nhưng lại **không hề kiểm tra tính hợp lệ của phần chữ ký (signature)**.==

### 1. Trinh sát
vào xem jwt , mình thấy và bài lab nó cũng bảo là sever nó ko check phần signature
![[Pasted image 20261101182601.png]]
-> nhiệm vụ bây giờ là chỉnh jwt và vào trang /admin
### 2. Khai thác
Mình quay lại request `GET /my-account` ban đầu. Nhìn vào cookie session, mình thấy nó là một chuỗi JWT. 
Mình bấm đúp chuột vào phần **payload** (phần ở giữa) của JWT. Tab **Inspector** bên phải hiện ra bảng decode JSON, mình thấy tham số:
```json
"sub": "wiener"
```

Vì server không kiểm tra chữ ký, mình chỉ cần sửa thẳng cái `sub` này thành `administrator`:
```json
"sub": "administrator"
```
Mình bấm **Apply changes** để Burp cập nhật lại chuỗi token. 
![[Pasted image 20261101182452.png]]
*Lưu ý: Khác với bài `alg: none`, ở bài này mình **không cần** sửa header, cũng **không cần** xóa phần signature. Cứ để nguyên phần chữ ký cũ rích đó, vì server có thèm kiểm tra nó đâu!*

### 3. Hốt thành quả
Mình đổi lại path của request thành `GET /admin` và bấm **Send**. Lần này server trả về `200 OK` kèm theo giao diện Admin Panel. 

Cuộn xuống dưới cùng của response HTML, mình tìm thấy đường link xóa user:
`/admin/delete?username=carlos`

Mình đổi request thành `GET /admin/delete?username=carlos` và Send. Vậy là lab đã báo **Solved**!
![[Pasted image 20261101182714.png]]

![[Pasted image 20261101182729.png]]