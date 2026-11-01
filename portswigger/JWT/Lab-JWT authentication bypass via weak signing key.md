
-mô tả lab: cookie `session` là JWT ký HS256 bằng secret rất yếu. Mình brute secret rồi tự ký token admin.
https://portswigger.net/web-security/jwt#brute-forcing-secret-keys-using-hashcat
> [!NOTE]
> -test bằng `wiener:peter`. Lab: https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-weak-signing-key
![[Pasted image 20261101183348.png]]
Dưới đây là writeup ngắn gọn, đủ ý và viết ở ngôi thứ nhất cho bài lab này:

Mục tiêu: Brute-force secret key yếu, tự ký lại JWT để vào `/admin` và xóa `carlos`.
![[Pasted image 20261101184106.png]]
-> alg là mã hóa bất đối xứng -> bây giờ mình cần brute cái key này 

**Bước 1: Brute-force secret key**
- Đăng nhập `wiener:peter`, bắt request `GET /my-account` bằng Burp và copy chuỗi JWT.
- Chạy hashcat với wordlist (file `jwt.secrets.list`) để brute-force:
  `hashcat -a 0 -m 16500 <JWT> jwt.secrets.list`
- Kết quả trả về secret key là: **`secret1`**.
![[Pasted image 20261101185504.png]]

![[Pasted image 20261101185910.png]]
-> tìm được secret key
**Bước 2: Tạo Key trong Burp**
- Dùng Burp Decoder Base64 chuỗi `secret1` thành `c2VjcmV0MQ==`.
- Cài extension **JWT Editor**. Mở tab **JWT Editor Keys**, bấm **New Symmetric Key** -> **Generate**.
- Thay giá trị của thuộc tính `k` bằng chuỗi Base64 vừa tạo (`c2VjcmV0MQ==`), bấm **OK** để lưu.
![[Pasted image 20261101190443.png]]
**Bước 3: Sửa và ký lại JWT**
- Trong Burp Repeater, đổi path request thành `GET /admin`.
- Chuyển sang tab **JSON Web Token** của extension, sửa payload `"sub": "wiener"` thành `"sub": "administrator"`.
- Kéo xuống dưới bấm **Sign**, chọn key vừa tạo. Tích chọn **"Don't modify header"** rồi bấm OK.
- Bấm **Send**, request trả về `200 OK` kèm giao diện Admin.
![[Pasted image 20261101190312.png]]
**Bước 4: Xóa user**
- Đổi URL request thành `GET /admin/delete?username=carlos` và Send. Lab Solved.
![[Pasted image 20261101190340.png]]

![[Pasted image 20261101190353.png]]