# Index server-side - mình tổng hợp lại

-phần này gồm mấy topic server-side mình hay gặp nhất. Mỗi topic mình có 1 file lý thuyết + lab đi kèm, thứ tự từ dễ tới khó.

## 1. Path traversal
-lý thuyết: [[lys thuyet path traversal]] (chi tiết cũ: [[path traversal]])
- [ ] mở bát: [[Lab-Path traversal simple]] => `filename=../../../../etc/passwd`
- [ ] tiếp: [[Lab-Path traversal absolute bypass]] => `filename=/etc/passwd`
- [ ] tiếp: [[Lab-Path traversal non-recursive strip]] => `....//....//` vì server strip 1 lần
- [ ] tiếp: [[Lab-Path traversal encoded]] => `%2e%2e%2f` / double encode

## 2. OS command injection
-lý thuyết: [[lys thuyet OS command injection]] (chi tiết cũ: [[injection command]])
- [ ] mở bát: [[Lab-OS command simple]] => `;whoami;` đọc output trên trang
- [ ] blind time: [[Lab-OS command blind time]] => `||ping -c 10||` đo delay
- [ ] blind redirect: [[Lab-OS command blind output redirect]] => `whoami > file` rồi GET file đọc

## 3. SSTI
-lý thuyết: [[lys thuyet SSTI]] (chi tiết cũ: [[server side template injection]])
- [ ] mở bát: [[Lab-SSTI basic]] => `{{7*7}}` ra 49 là dính
- [ ] đọc secret: [[Lab-SSTI info disclosure via objects]] => `{{settings.SECRET_KEY}}`
- [ ] engine riêng: [[Lab-SSTI engine-specific]] => Tornado/Mako payload RCE

## 4. File upload
-lý thuyết: [[lys thuyet file upload]] (chi tiết cũ: [[file upload]])
- [ ] mở bát: [[Lab-File upload unrestricted]] => up thẳng shell.php
- [ ] bypass content-type: [[Lab-File upload content-type bypass]] => đổi `image/jpeg`
- [ ] bypass extension: [[Lab-File upload extension bypass]] => `.php5` / `.htaccess`

## 5. Access control / Auth phía server
- [ ] [[access control]], [[Authentication vulnerabilities-note]]: 2 file này mình để đây để tra, còn lab IDOR/Auth chi tiết mình để ở folder [[../IDOR/0-index IDOR]] và [[../authetication bypass/0-authentication]].

> [!NOTE]
> -bài học chung của mình cho cả 4 topic: đừng bao giờ ghép input user vào câu lệnh/đường dẫn/template/upload mà không validate + canonicalize + whitelist.
