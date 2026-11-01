![[Pasted image 20261101231249.png]]
-mô tả lab: server ==🟡chịu nhúng `jwk` (public key) từ header token vào để verify==. Mình tự tạo cặp key, nhét public key mình vào token, ký bằng private key mình.

> [!NOTE]
> -test bằng `wiener:peter`. Lab: https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jwk-header-injection

Mục tiêu: Lợi dụng ==🔴lỗi== ==🟢server chấp nhận khóa `jwk` nhúng trong header,== mình sẽ tự tạo một cặp khóa, giả mạo token để vào `/admin` và xóa `carlos`.

### 1. Trinh sát
Mình đăng nhập bằng `wiener:peter`, bắt request `GET /my-account` đưa sang Burp Repeater. 
Mình thử đổi path thành `/admin` và Send, đương nhiên là bị chặn. Nhìn vào cookie session, mình thấy nó là một chuỗi JWT. ==🟡Đề bài cho biết server hỗ trợ tham số `jwk` trên header, nghĩa là mình có thể tự nạp khóa của mình vào== để server dùng nó xác thực chữ ký.
![[Pasted image 20261101232351.png]]

-> từ ảnh ta có thể thấy sever dùng kid để khai báo public key
-Lý thuyết bên dưới như này , cách để khai báo public key cho server
![[Pasted image 20261101232336.png]]
### 2. Chuẩn bị "vũ khí"
Mình cần một cặp khóa RSA (Public/Private Key) để tự ký token.
- Mình mở tab **JWT Editor Keys** (do extension JWT Editor tạo ra).
- Bấm **New RSA Key** -> Bấm **Generate** để nó tự sinh ra một cặp khóa -> Bấm **OK** để lưu.
![[Pasted image 20261101233259.png]]
### 3. Tiến hành tấn công
- Mình quay lại tab **Repeater** với request `GET /admin`.
- Chuyển sang tab **JSON Web Token** của extension.
- Ở phần Payload, mình đổi `"sub": "wiener"` thành `"sub": "administrator"`.
- Kéo xuống dưới cùng, mình bấm nút **Attack** -> chọn **Embedded JWK**.
- Một bảng hiện ra, mình chọn cái **RSA Key** vừa tạo ở Bước 2 rồi bấm OK.
![[Pasted image 20261101233732.png]]
*Lúc này, extension đã tự động làm 2 việc: Nhúng Public Key của mình vào header JWT (tạo ra tham số `jwk`) và dùng Private Key để ký lại toàn bộ token.*

-> thành quả nhận được
![[Pasted image 20261101233806.png]]
### 4. Hốt thành quả
Mình bấm **Send** lại request `GET /admin`. Lần này server trả về `200 OK` kèm giao diện Admin. 
Cuộn xuống dưới, mình lấy đường link `/admin/delete?username=carlos`, đổi method request và Send. Lab báo **Solved**!

![[Pasted image 20261101234337.png]]

![[Pasted image 20261101234408.png]]
![[Pasted image 20261101234556.png]]
**Chút suy nghĩ cá nhân:**
Lỗi này cực kỳ nguy hiểm. Lập trình viên thường nghĩ: *"Token có chữ ký hợp lệ, mà chữ ký này khớp với cái `jwk` (khóa) đi kèm trong token, nên token này là thật"*. 
Họ quên mất một điều kiện tiên quyết: Cái khóa đó phải là của Server, chứ không phải khóa do Hacker tự bịa ra rồi nhét vào! Vì thế, mình chỉ cần tự tạo khóa, tự ký, tự nhúng vào là qua mặt được server. 

*(Mẹo nhỏ: Nếu không dùng tính năng Attack tự động của Burp, bạn có thể tự copy cái `jwk` dán vào header, nhưng nhớ phải sửa tham số `kid` (Key ID) trên header cho khớp với `kid` của cái khóa vừa nhúng thì mới chạy được).*