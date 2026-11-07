
-mô tả lab: server ký RS256 (có public/private key) nhưng verify ẩu theo `alg` trong header. Mình đổi sang HS256 rồi ký bằng chính public key của server.
![[Pasted image 20261101235750.png]]
> [!NOTE]
> -test bằng `wiener:peter`. Lab: https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-algorithm-confusion

-đầu tiên phải hiểu PEM

![[Pasted image 20261102012516.png]]

![[Pasted image 20261102012533.png]]
Mục tiêu: Đánh lừa server bằng kỹ thuật "Algorithm Confusion", lấy quyền admin để vào `/admin` và xóa `carlos`.

### 1. Trinh sát
Mình đăng nhập bằng `wiener:peter`, gửi request `GET /my-account` sang Burp Repeater. Thử đổi path thành `/admin` thì bị chặn. Nhìn vào JWT, mình thấy header báo `"alg": "RS256"` (sử dụng thuật toán bất đối xứng RSA). 

Đề bài gợi ý ==🟢server lưu public key dạng X.509 PEM và để lộ nó ở một endpoint chuẩn.== Mình mở trình duyệt, truy cập đường dẫn `/jwks.json` thì thấy server trả về một JWK Set chứa public key. Mình copy nguyên cái object JWK bên trong mảng `keys` (không copy dấu ngoặc vuông `[]`).
![[Pasted image 20261102013108.png]]
-> có thể thấy cái khóa public key này 
![[Pasted image 20261102010410.png|503]]

> [!NOTE] 
> Thường thì server lưu public key ở 1 enpoint chuẩn 


### 2. Chuẩn bị "vũ khí" (Tạo Key)
Lỗ hổng nằm ở chỗ: Server không kiểm tra chặt chẽ thuật toán. ==🔴Nếu mình đổi `alg` từ `RS256` (RSA) sang `HS256` (HMAC), server vẫn chấp nhận.== ==🟡Trớ trêu thay, khi dùng HS256, server sẽ lấy **Public Key** của nó làm **Secret Key** để kiểm tra chữ ký==. Mình lợi dụng điều này để tự ký token.

**Các bước mình làm trên Burp:**
1.  Mở tab **JWT Editor Keys** -> Bấm **New RSA Key**.
2.  Chọn tab **JWK**, dán cái JWK vừa copy trên trình duyệt vào -> Bấm **OK**.
3.  Chuột phải vào key vừa tạo, chọn **Copy Public Key as PEM**.
 ![[Pasted image 20261102013623.png|542]]
 -> mục đích là để lưu cái jwk public kia về burp để chỉnh sửa...
 ![[Pasted image 20261102014531.png|354]]
 ->copy cái public key ở dạng pem -> encode cái pubkey này ở base 64 rồi đưa vào trường k của khóa đối xứng để làm phôi
      
4.  Quay lại **JWT Editor Keys** -> Bấm **New Symmetric Key** -> Bấm **Generate**.
![[Pasted image 20261102014954.png|495]]
->Tìm thuộc tính `k` trong cái key vừa sinh ra, xóa giá trị cũ và dán chuỗi Base64 vừa tạo ở bước 4 vào. Bấm **OK** để lưu. 
*(Lúc này mình đã có một Symmetric Key, nhưng thực chất ruột của nó chính là Public Key của server).*

> [!NOTE]
> ok có thể hiểu như này, tạo 1 khóa đối xứng(symetric key), và cho cái khóa công khai vào phôi cái khóa đối xứng này để tạo symetric key, sau đó đổi thành admin và ký lại signature bằng cái khóa này thôi.
> -> hiểu dơn giản là cho cái thằng public key thành 1 cái private key thôi
> -> sau đó đổi trường agl thành mã hóa đối xứng để lừa server thôi

> [!question] Vậy tại sao phải tạo rsa key làm gì, mà ko cho thẳng public key làm phôi cho khóa đối xứng luôn?
> ![[Pasted image 20261102012133.png]]
> 


### 3. Chế tạo Token và tấn công
1.  Quay lại tab **Repeater** với request `GET /admin`.
2.  Chuyển sang tab **JSON Web Token** của extension.
3.  Ở phần Header, mình sửa `"alg": "RS256"` thành `"alg": "HS256"`.
4.  Ở phần Payload, mình sửa `"sub": "wiener"` thành `"sub": "administrator"`.
5.  Kéo xuống dưới cùng, bấm **Sign**, chọn cái **Symmetric Key** vừa tạo ở Bước 2. Tích chọn **"Don't modify header"** rồi bấm OK.
6.  Bấm **Send**.
![[Pasted image 20261102015157.png|631]]
Kết quả: Server trả về `200 OK` kèm giao diện Admin Panel. Mình đổi request thành `/admin/delete?username=carlos` và Send. Lab Solved!


![[Pasted image 20261102023638.png|610]]