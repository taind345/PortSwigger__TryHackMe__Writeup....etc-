![[auth-encryption-oracle.png]]
-mô tả lab: bài này khoai nhất phần auth. Cookie `stay-logged-in` được mã hóa bằng cùng key với chức năng comment. Mình sẽ dùng comment như 1 cái máy mã hóa để tự tạo cookie admin.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/authentication/other-mechanisms/lab-authentication-bypass-via-encryption-oracle
> -bài này mình phải hiểu sơ về AES-CBC + IV, nhưng cách làm thì đơn giản: copy block.

### hướng giải
- [ ] mình login account thường, sau đó quan sát cookie này, mình thấy có 2 cookie: `session` và `stay-logged-in=ENCRYPTED_BLOB` rất dài
- [ ] mình vào chức năng post comment ở blog, mình nhập email `test@test.com`, sau đó quan sát gói tin này, mình thấy comment lưu lại email của mình dưới dạng mã hóa giống hệt format của cookie
- [ ] mình thử nhập email dài hơn, mình thấy blob mã hóa dài ra theo block 16 byte
- [ ] => nên mình đoán là server dùng AES-CBC với cùng key để mã hóa cả cookie và email comment. Vậy mình có thể dùng comment để mã hóa chuỗi `administrator:xxx` rồi copy thành cookie

### giải
-bước 1: mình tìm hiểu cấu trúc cookie bằng cách giải mã base64:
```
stay-logged-in = IV (16 byte đầu) + ciphertext
plaintext = username:password_hash ?
```
-portswigger gợi ý là plaintext có dạng `administrator:156...` gì đó. Mình không cần biết chi tiết, mình chỉ cần tạo được block chứa `administrator`.

-bước 2: mình dùng kỹ thuật prefix + padding để ép chữ `administrator` nằm gọn trong 1 block riêng. Ví dụ mình nhập email là:
```
AAAAAAAAAAAAAAAAadministrator
```
-sau đó mình xem blob comment trả về, mình cắt lấy block chứa `administrator` đã mã hóa.

-bước 3: mình ghép block đó với IV + các block còn lại để tạo cookie mới:
```python
# cách của mình: dùng Burp + script của portswigger solution, hoặc làm tay
# 1. post comment với email = "A"*X + "administrator"
# 2. lấy ciphertext block thứ N
# 3. thay block tương ứng trong cookie stay-logged-in của mình bằng block đó
# 4. gửi GET /my-account với cookie mới
```
![[auth-oracle-blocks.png]]
-sau khi quan sát gói tin trả về này, mình thấy nó hiện `My Account | administrator`. Là mình đã giả được admin.

-bước 4: mình vào `/admin/delete?username=carlos` hoặc click delete là solve.

> [!NOTE]
> -vì sao làm được? Vì AES-CBC mỗi block mã hóa độc lập (XOR với block trước). Nếu mình kiểm soát được plaintext của 1 block thì mình sẽ có ciphertext của block đó, rồi mình ghép vào chỗ khác vẫn giải mã ra đúng.
> -bài học của mình: không bao giờ dùng cùng key mã hóa cho 2 chức năng khác nhau, nhất là khi 1 chức năng cho user kiểm soát input (comment) còn 1 cái là cookie phân quyền.
> -lab này mình làm theo write-up mẫu của portswigger là dễ hiểu nhất, mình chỉ cần hiểu ý tưởng oracle là được.
