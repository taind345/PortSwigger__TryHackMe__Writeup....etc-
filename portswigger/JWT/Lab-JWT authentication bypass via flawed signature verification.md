https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-flawed-signature-verification

![[Pasted image 20261101171127.png]]
Bài lab này (mức Apprentice) yêu cầu mình phải vào được trang `/admin` và xóa user `carlos`. ==🟡Lỗi nằm ở chỗ server tin tưởng tuyệt đối vào thuật toán (algorithm) do client gửi lên==, và sẵn sàng chấp nhận thuật toán `none` (không cần chữ ký).

Dưới đây là các bước mình đã làm:

> [!info] 
> đơn giản là thay trường algrithm trong header và ký lại jwt 

### 1. Trinh sát (Recon)
Đầu tiên, mình đăng nhập vào lab bằng tài khoản được cấp sẵn là `wiener:peter`. 
Mình bật Burp Suite lên, vào tab **Proxy > HTTP history**, tìm cái request `GET /my-account` ngay sau khi đăng nhập. Mình thấy cookie session của mình là một chuỗi JWT. 

Mình gửi request này sang tab **Repeater**, rồi bấm đúp chuột vào phần payload (phần ở giữa) của cái token. Tab **Inspector** bên phải hiện ra bảng decode JSON, mình thấy nội dung đại khái như sau:

![[Pasted image 20261101173831.png]]
```json
{
  "iss": "portswigger",
  "sub": "wiener",
  "exp": 1712345678
}
```
Tham số `sub` đang là `wiener`. Mình thử đổi path của request thành `/admin` và Send, đương nhiên là bị chặn (trả về lỗi) vì `wiener` không phải là admin.

### 2. Khai thác lỗi "alg: none"
Mục tiêu của mình là ==🟡làm giả một token để server nghĩ mình là== `administrator`. Bình thường JWT phải có chữ ký (signature) để chống giả mạo, nhưng server này cấu hình lỏng lẻo, nó chấp nhận token không có chữ ký nếu header ghi `alg: none`.

**Cách mình thao tác trên Burp:**
1. Trong tab Repeater, mình bấm vào phần **payload** của JWT.
2. Ở bảng Inspector bên phải, mình sửa giá trị của `sub` từ `wiener` thành `administrator`. Bấm **Apply changes**.
3. Mình bấm tiếp vào phần **header** của JWT. Sửa tham số `alg` (algorithm) từ `HS256` thành `none`. Bấm **Apply changes**.
4. **Điểm mấu chốt (Rất dễ sai):** JWT có cấu trúc gồm 3 phần cách nhau bằng dấu chấm: `Header.Payload.Signature`. Vì mình dùng `alg: none` nên phần Signature sẽ bị bỏ trống. Mình **xóa sạch phần chữ ký (Signature)** ở cuối token, nhưng **phải nhớ giữ lại dấu chấm (.)** cuối cùng. 
   * *Token ban đầu:* `eyJhbG... .eyJzdWI... .chu_ky_bi_xoa`
   * *Token sau khi sửa:* `eyJhbG... .eyJzdWI... .` (có dấu chấm ở cuối).

> [!NOTE] 
> Có 2 việc mình đã làm ở đây
> -1 là sửa sub thành admin
> -2 là sửa alg thành none và ký lại
> -> như thế server sẽ bỏ qua phần signature và chấp nhận cái jwt token là admin
![[Pasted image 20261101174144.png]]
### 3. Hốt thành quả
Mình bấm **Send** lại request `GET /admin` với cái token vừa sửa. Lần này server trả về `200 OK` kèm giao diện Admin Panel. 

Cuộn xuống dưới cùng của response HTML, mình thấy một đường link dùng để xóa user:
`/admin/delete?username=carlos`

Mình đổi request hiện tại thành `GET /admin/delete?username=carlos` và bấm Send. Server trả về 200 là lab đã báo **Solved**.
![[Pasted image 20261101173937.png]]

![[Pasted image 20261101174014.png]]

![[Pasted image 20261101174215.png]]