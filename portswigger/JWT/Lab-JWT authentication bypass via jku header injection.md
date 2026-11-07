![[Pasted image 20261102181525.png|626]]
https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jku-header-injection

Mục tiêu: Lợi dụng tham số `jku` (JWK Set URL) để ==🟢lừa server tải Public Key do mình tự tạo==, từ đó giả mạo token admin để vào `/admin` và xóa `carlos`.

### 1. Chuẩn bị "vũ khí" (Tạo khóa và Host JWK Set)
-đầu tiên ta phải biết cấu trúc của jwt 
![[Pasted image 20261102182047.png|480]]
Đầu tiên, mình đăng nhập bằng `wiener:peter`, bắt request `GET /my-account` sang Repeater, thử đổi thành `/admin` thì bị chặn. Token dùng thuật toán `RS256`.

- Mình mở tab **JWT Editor Keys** -> Bấm **New RSA Key** -> Bấm **Generate** để tạo một cặp khóa mới -> Bấm **OK**.
- Chuột phải vào cái key vừa tạo, chọn **Copy Public Key as JWK**. Lưu lại mã `kid` của khóa này.
- Lên **Exploit Server**, ở phần Body, mình tạo một cấu trúc JWK Set trống, rồi dán cái JWK vừa copy vào trong mảng `keys`:
```json
{
    "keys": [
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "mã_kid_của_bạn",
            "n": "chuỗi_n_của_bạn..."
        }
    ]
}
```
Bấm **Store** để lưu lại. Lúc này mình đã có một Public Key "hợp pháp" do chính mình host trên Exploit Server.
![[Pasted image 20261102182439.png|427]]
### 2. Chế tạo Token và tấn công
- Quay lại Burp Repeater, chuyển sang tab **JSON Web Token**.
- **Sửa Header:** 
  - Thay đổi giá trị `kid` hiện tại thành mã `kid` của cái khóa mình vừa tạo.
  - Thêm một tham số mới là `jku`. Giá trị của nó chính là URL của Exploit Server (ví dụ: `https://exploit-xxx.exploit-server.net/exploit`).
  *Header lúc này sẽ trông như sau:*
  ```json
  {
    "kid": "mã_kid_của_bạn",
    "jku": "https://exploit-xxx.exploit-server.net/exploit",
    "alg": "RS256"
  }
  ```
- **Sửa Payload:** Đổi `"sub": "wiener"` thành `"sub": "administrator"`.
- **Ký lại Token:** Kéo xuống dưới cùng, bấm **Sign**, chọn cái **RSA Key** mình đã tạo ở Bước 1. Tích chọn **"Don't modify header"** rồi bấm OK.
![[Pasted image 20261102183144.png|593]]
-> vào được admin thành công, là do ở đây ta đã đánh lừa sever lấy cái public key nằm trên sever của mình để ký và giải mã header
### 3. Hốt thành quả
- Bấm **Send** lại request `GET /admin`. Server sẽ đọc tham số `jku`, tự động gửi request đến Exploit Server của mình để lấy Public Key, và dùng khóa đó để xác thực chữ ký của token. Vì mình ký bằng Private Key tương ứng, chữ ký hoàn toàn khớp -> Server trả về `200 OK` kèm giao diện Admin.
- Mình đổi request thành `GET /admin/delete?username=carlos` và Send. Lab Solved!
![[Pasted image 20261102183402.png]]

-Hiểu thêm về vai trò của public key và private key trong jwt
![[Pasted image 20261102183711.png]]
-> Trong bài này , sever đáng lẽ ra phải vừa private để ký, và dùng pub để ký, lý do tại sao cần rườm rà như vậy, tận 2 khóa lận? là do sever có nhiều máy chủ, và người ta muốn cái private key-chức năng ký dành cho một vài người thôi-> do đó mới sinh ra cái trò pub-pri trong jwt
-> còn về ở trường hợp này, chúng ta đã giả danh sver đề ký luôn, và lừa sever tới 1 cái public key ta tạo ra, để nó xác minh-> solve