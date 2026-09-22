# 1-intro
This room introduces several protocols commonly used across networks:
	- http
	- pop3
	- ftp
	- smtp
	- imap
These protocols form the foundation of how you browse the web, transfer files, and send and receive email. <u>Understanding how they work at a low level is essential for anyone pursuing a career in cybersecurity,</u> whether in penetration testing, network defence, or security engineering.

Each protocol task is designed to show what happens beneath the surface, which is usually hidden by a (Graphical User Interface). You will "talk" using the above protocols with a simple Telnet client to understand what your client is doing under the hood. <u>The purpose is not to memorise the protocol commands but rather to observe the protocol while it is working</u>.

## Why Learn These Protocols Today?

You might wonder why you are studying protocols that are decades old. There are several important reasons.

First, these protocols are still in use. While most public-facing services now use encrypted versions (HTTPS, SFTP, IMAPS), the underlying protocol mechanics remain the same. The commands you send over HTTPS are the same commands, wrapped in encryption.

Second, you will encounter cleartext protocols during penetration tests and security assessments. Legacy systems, internal networks, devices, and misconfigured services often still use unencrypted protocols. Recognising and exploiting these weaknesses is a core penetration testing skill.

Third, understanding the protocol helps you understand the attacks. When you know how works, you understand why email spoofing is possible. When you understand , web application vulnerabilities make more sense. This foundational knowledge makes you a better security professional.

This room also discusses some of the insecurities inherent in these protocols. In particular, the focus is on passwords sent in cleartext. When credentials are transmitted without encryption, anyone with access to the network traffic can capture them. This was acceptable when these protocols were designed for trusted academic networks, but it is a serious vulnerability on modern networks.

In the follow-up room, [Protocols and Servers 2](https://tryhackme.com/room/protocolsandservers2), you will explore how these protocols can be secured using encryption, and examine attacks such as sniffing, man-in-the-middle, and password attacks in more detail.

## Prerequisites

This room assumes a working understanding of /IP, ports, and the client-server model. You should also be comfortable with the terminal. If you need a refresher on any of these topics, complete the following first:
- [Networking](https://tryhackme.com/module/networking)
- [linux Fundamentals](https://tryhackme.com/module/linux-fundamentals)

# 2-Telnet
Giao thức Telnet là giao thức tầng ứng dụng (application-layer protocol) được dùng để kết nối tới terminal ảo của một máy tính khác. Sử dụng Telnet, người dùng có thể đăng nhập vào máy tính từ xa và truy cập terminal (console) để chạy chương trình, thực thi các tiến trình hàng loạt (batch process) và thực hiện các tác vụ quản trị hệ thống từ xa.

Giao thức Telnet tương đối đơn giản. Khi người dùng kết nối, hệ thống sẽ yêu cầu cung cấp username và password. Sau khi xác thực đúng, người dùng sẽ có quyền truy cập vào terminal của hệ thống từ xa. Tuy nhiên, toàn bộ dữ liệu giao tiếp giữa Telnet client và Telnet server đều không được mã hóa, biến nó thành mục tiêu dễ bị tấn công.

### Telnet ngày nay

Telnet từng được sử dụng rộng rãi để quản trị từ xa trong thời kỳ đầu của mạng máy tính. Tuy nhiên, nó gần như đã bị thay thế hoàn toàn bởi SSH (Secure Shell) cho các kết nối tương tác từ xa. Bạn hầu như sẽ không thấy Telnet được bật trên các hệ thống hiện đại được cấu hình chuẩn. Dù vậy, bạn vẫn có thể bắt gặp nó trong:

* Các hệ thống cũ (legacy) và thiết bị mạng đời cũ (router, switch, bộ điều khiển công nghiệp).
* Các thiết bị nhúng và thiết bị bị hạn chế về tài nguyên phần cứng.
* Các mạng nội bộ nơi tính bảo mật chưa từng được ưu tiên.
* Các hệ thống bị cấu hình sai khi Telnet được bật nhưng không bao giờ bị tắt đi.

Trong quá trình kiểm thử xâm nhập (pentest), <u>việc tìm thấy cổng Telnet (port 23) đang mở thường là một phát hiện quan trọng, vì nó báo hiệu một hệ thống cũ hoặc một lỗi cấu hình bảo mật.</u>

### Telnet Client như một công cụ kiểm thử

Mặc dù Telnet server rất hiếm gặp, Telnet client vẫn hữu ích như một công cụ đơn giản để kết nối tới bất kỳ cổng TCP nào và tương tác thủ công với các giao thức dạng văn bản (text-based protocol). Ví dụ, bạn có thể dùng lệnh `telnet target 80` để kết nối tới web server và nhập các lệnh HTTP thủ công. Cách dùng này được trình bày trong bài Active Reconnaissance.

### Cách Telnet hoạt động

Telnet server lắng nghe các kết nối đến trên port 23. Phần hiển thị terminal dưới đây chỉ mang tính minh họa và được trích xuất từ một môi trường riêng biệt. Cổng Telnet không mở trên máy ảo mục tiêu của bài học này, nhưng nó thể hiện rõ luồng xác thực:

1. Người dùng được yêu cầu nhập tên đăng nhập (username). Trong ví dụ này, người dùng nhập `frank`.
2. Hệ thống yêu cầu nhập mật khẩu (`D2xc9CgD`). Mật khẩu không hiển thị trên màn hình thực tế, nhưng được hiển thị ở đây nhằm mục đích minh họa.
3. Khi hệ thống xác thực thông tin đăng nhập, người dùng sẽ nhận được thông báo chào mừng.
4. Server từ xa cấp dấu nhắc lệnh `frank@bento:~$`. Ký tự `$` cho biết đây không phải là terminal có quyền root.

```tex
pentester@TryHackMe$ telnet MACHINE_IP
Trying MACHINE_IP...
Connected to MACHINE_IP.
Escape character is '^]'.
Ubuntu 20.04.3 LTS
bento login: frank
Password: D2xc9CgD
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-84-generic x86_64)
 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
  System information as of Fri 01 Oct 2021 12:24:56 PM UTC
  System load:  0.05              Processes:              243
  Usage of /:   45.7% of 6.53GB   Users logged in:        1
  Memory usage: 15%               IPv4 address for ens33: MACHINE_IP
  Swap usage:   0%
 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.
   https://ubuntu.com/blog/microk8s-memory-optimisation
0 updates can be applied immediately.
*** System restart required ***
Last login: Fri Oct  1 12:17:25 UTC 2021 from meiyo on pts/3
You have mail.
frank@bento:~$

```

Dù Telnet cấp quyền truy cập terminal từ xa rất nhanh chóng, nó không phải là giao thức đáng tin cậy cho việc quản trị vì toàn bộ dữ liệu được gửi dưới dạng văn bản rõ (cleartext). Khi bắt lại các gói tin Telnet, việc tìm ra mật khẩu là rất dễ dàng. Dữ liệu mã ASCII trao đổi cho thấy phần chữ màu đỏ là dữ liệu client gửi đi, còn màu xanh dương là phản hồi từ hệ thống từ xa. Username được hiển thị lại trên terminal, nhưng password thì không. Tuy nhiên, việc giấu password trên màn hình là vô nghĩa vì nó vẫn được truyền qua mạng ở dạng văn bản không mã hóa.
![[Pasted image 20260815103332.png]]
### Tại sao Telnet không an toàn

Telnet không còn là một lựa chọn an toàn. Bất kỳ ai bắt được lưu lượng mạng của bạn đều có thể đọc được username và password để chiếm quyền truy cập hệ thống từ xa. Những đối tượng này bao gồm:

* Kẻ tấn công nằm trong cùng phân đoạn mạng (network segment).
* Bất kỳ ai kiểm soát được router hoặc switch trên đường truyền.
* Kẻ nội bộ độc hại có quyền truy cập vào mạng.
* Bất kỳ ai thực hiện thành công cuộc tấn công Man-in-the-Middle (MitM).

Giải pháp thay thế an toàn là SSH (Secure Shell), giao thức mã hóa toàn bộ lưu lượng dữ liệu bao gồm cả thông tin xác thực. SSH đã trở thành tiêu chuẩn quản trị dòng lệnh từ xa trong hơn hai thập kỷ qua.

# 3-HTTP
Giao thức truyền tải siêu văn bản (HTTP - Hypertext Transfer Protocol) là giao thức dùng để truyền tải các trang web. Trình duyệt web kết nối tới web server và dùng HTTP để yêu cầu các trang HTML, hình ảnh và các file khác. Nó cũng gửi dữ liệu biểu mẫu và tải lên các file. Mỗi khi duyệt World Wide Web (WWW), bạn đều đang sử dụng giao thức HTTP.

Hình ảnh bên dưới minh họa việc client yêu cầu trang HTML `index.html` do web server cung cấp. Sau đó client yêu cầu file ảnh `logo.jpg` và web server gửi file đó về.
![[Pasted image 20260815103944.png]]
### HTTP so với HTTPS

HTTP gửi và nhận dữ liệu dưới dạng văn bản thuần (cleartext, không mã hóa). Điều này có nghĩa là bất kỳ ai có quyền truy cập vào lưu lượng mạng đều có thể đọc nội dung được truyền tải, bao gồm các thông tin nhạy cảm như thông tin đăng nhập và dữ liệu cá nhân.*=> không mã hóa*

Ngày nay, phần lớn các website sử dụng HTTPS (HTTP Secure), tức là bọc HTTP bên trong lớp mã hóa TLS. Các trình duyệt hiện đại sẽ đánh dấu các trang web chạy HTTP thuần là "Không an toàn" (Not Secure), và một số tính năng (như vị trí địa lý và quyền truy cập camera) bị chặn hoàn toàn trên các trang không dùng HTTPS. Tuy nhiên, việc hiểu cách thức hoạt động của HTTP vẫn rất cần thiết vì:

* Các câu lệnh và cấu trúc HTTP hoàn toàn giống nhau dù sử dụng HTTP hay HTTPS.
* Bạn sẽ gặp HTTP trong các bài kiểm thử xâm nhập nội bộ (internal pentest) và trên các hệ thống cũ (legacy).
* Hiểu rõ giao thức này giúp bạn phát hiện và khai thác các lỗ hổng web.
* Các công cụ như **Burp Suite** *giải mã lưu lượng HTTPS để phân tích, hiển thị cho bạn dữ liệu HTTP thô.*

Trong phần minh họa này, giao thức HTTP thông thường được sử dụng để bạn có thể thấy chính xác những gì đang được truyền đi.

### Gửi yêu cầu HTTP thủ công

Vì HTTP là giao thức dạng văn bản thuần, bạn có thể dùng một công cụ đơn giản như Telnet (hoặc Netcat) để giao tiếp với web server và đóng vai trò như một "trình duyệt web". <u>Điểm khác biệt chính là bạn cần tự nhập các câu lệnh HTTP</u> thay vì để trình duyệt làm điều đó cho bạn.*=> http là giao thức dạng văn bản thuần*

Trong ví dụ sau, bạn sẽ thấy cách yêu cầu một trang từ web server và khám phá phiên bản của web server. Telnet client được sử dụng vì Telnet là giao thức đơn giản truyền dữ liệu dạng văn bản rõ. Các bước thực hiện như sau:

1. Kết nối tới cổng 80 bằng lệnh `telnet MACHINE_IP 80`.
2. Nhập `GET /index.html HTTP/1.1` để lấy trang index.html, hoặc `GET / HTTP/1.1` để lấy trang mặc định.
3. Cung cấp giá trị cho host header, ví dụ `host: telnet`, rồi nhấn phím Enter 2 lần.

Ở đầu ra terminal bên dưới, trang được yêu cầu đã được trả về cùng với các thông tin mà trình duyệt web thường không hiển thị. Nếu trang yêu cầu không tồn tại, server sẽ trả về lỗi 404.

```tex
pentester@TryHackMe$ telnet MACHINE_IP 80
Trying MACHINE_IP...
Connected to MACHINE_IP.
Escape character is '^]'.
GET /index.html HTTP/1.1
host: telnet

HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Wed, 15 Sep 2021 08:56:20 GMT
Content-Type: text/html
Content-Length: 234
Last-Modified: Wed, 15 Sep 2021 08:53:59 GMT
Connection: keep-alive
ETag: "6141b4a7-ea"
Accept-Ranges: bytes

<!DOCTYPE html>
<html lang="en">
<head>
  <title>Welcome to my Web Server</title>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
</head>
<body>
  <h1>Coming Soon</h1>
</body>
</html>

```

Nội dung nhập quan trọng từ người dùng gồm hai dòng: `GET /index.html HTTP/1.1` theo sau bởi `host: telnet`.

### Thông tin được tiết lộ trong HTTP Headers

Hãy chú ý đến các response header trong ví dụ trên. Header `Server: nginx/1.18.0 (Ubuntu)` tiết lộ cả phần mềm web server cùng phiên bản, cũng như hệ điều hành đang chạy. Thông tin này rất có giá trị trong giai đoạn trinh sát (reconnaissance) vì:

* Các phiên bản cụ thể có thể tồn tại lỗ hổng đã biết để bạn nghiên cứu khai thác.
* Thông tin hệ điều hành giúp tinh chỉnh các cuộc tấn công tiếp theo.
* Việc biết loại phần mềm web server giúp thu hẹp các hướng tấn công tiềm năng.

Các quản trị viên chú trọng bảo mật thường cấu hình server để ẩn hoặc làm mờ thông tin này. Trong một bài kiểm thử xâm nhập, việc phát hiện thông tin phiên bản chi tiết trong header là điều rất đáng ghi nhận.

### Web Servers và Clients

Cần có một HTTP server (web server) và một HTTP client (trình duyệt web) để sử dụng giao thức HTTP. Web server phục vụ một tập hợp các file cụ thể cho trình duyệt web gửi yêu cầu.

<u>Các phần mềm HTTP server phổ biến gồm:</u>
* **Nginx**: Web server được sử dụng rộng rãi nhất trên internet, nổi tiếng về hiệu năng và khả năng xử lý đồng thời nhiều kết nối. Miễn phí và mã nguồn mở.
* **Apache**: Vận hành phần lớn các trang web, khả năng tùy biến cao với hệ sinh thái module khổng lồ. Miễn phí và mã nguồn mở.
* **IIS** (Internet Information Services): Web server của Microsoft, thường gặp trong môi trường doanh nghiệp chạy Windows, yêu cầu bản quyền Windows Server.
* Các web server đáng chú ý khác: LiteSpeed, Caddy (tích hợp sẵn HTTPS tự động), và Node.js cho các ứng dụng JavaScript.

Các trình duyệt web phổ biến nhất hiện nay:

* Chrome của Google (chiếm thị phần áp đảo)
* Safari của Apple (mặc định trên macOS và iOS)
* Edge của Microsoft (dựa trên nhân Chromium, thay thế cho Internet Explorer)
* Firefox của Mozilla (mã nguồn mở, tập trung vào quyền riêng tư)

Các trình duyệt web thường miễn phí cài đặt và sử dụng. <u>Đối với kiểm thử xâm nhập và nghiên cứu bảo mật, Firefox thường được ưu tiên nhờ bộ công cụ lập trình viên toàn diện và hệ sinh thái tiện ích mở rộng phong phú.</u>

### Các phiên bản giao thức HTTP
Ví dụ trên sử dụng HTTP/1.1, phiên bản nền tảng của web trong nhiều thập kỷ qua. Tuy nhiên, <u>các phiên bản mới hơn đã xuất hiện:</u>
* **HTTP/2** giới thiệu tính năng ghép kênh (multiplexing - nhiều yêu cầu trên một kết nối duy nhất), nén header và server push. Nó sử dụng định dạng nhị phân (binary) thay vì văn bản, khiến việc tương tác thủ công bằng Telnet trở nên khó khăn hơn.
* **HTTP/3** sử dụng QUIC (xây dựng trên UDP) thay vì TCP, mang lại hiệu năng cao hơn, đặc biệt trên các mạng không ổn định. Nó đang ngày càng phổ biến trên các website lớn.
Đối với mục đích học tập và kiểm thử thủ công, HTTP/1.1 vẫn là phiên bản dễ tiếp cận nhất nhờ định dạng văn bản đọc được trực tiếp.
![[Pasted image 20260815105740.png]]
# 4-FTP
Giao thức truyền tập tin (FTP - File Transfer Protocol) được phát triển nhằm giúp việc trao đổi file giữa các hệ thống máy tính khác nhau diễn ra hiệu quả. Đây là một trong những giao thức mạng xuất hiện sớm nhất và hiện vẫn còn được sử dụng, dù phần lớn đã được thay thế bằng các giải pháp an toàn hơn.
>phần này phần lớn đã học ở room [[Networking core Protocols#5-FTP]]
### FTP hiện đại
FTP gửi thông tin đăng nhập và dữ liệu hoàn toàn dưới dạng văn bản rõ (không mã hóa), do đó không an toàn khi truyền tải dữ liệu nhạy cảm. Trong hầu hết môi trường hiện nay, FTP đã được thay thế bởi:

* SFTP (SSH File Transfer Protocol): Chạy trên nền SSH qua port 22 và mã hóa toàn bộ lưu lượng mạng. Đây là giải pháp thay thế phổ biến nhất.
* FTPS (FTP Secure): Bổ sung lớp mã hóa TLS vào giao thức FTP, hoạt động trên port 990 hoặc dùng STARTTLS trên port 21.
* SCP (Secure Copy Protocol): Cũng hoạt động trên nền SSH, tuy nhiên hiện đang dần bị loại bỏ để nhường chỗ cho SFTP.
> đại khái là nó ko an toàn , tại vì nó ko mã hóa, nên bây giờ hầu như không ai dùng cả

Dù vậy, bạn vẫn có thể bắt gặp FTP truyền thống trong:
* Các hệ thống hoặc ứng dụng cũ chưa được nâng cấp.
* Các máy chủ FTP ẩn danh dùng để chia sẻ file công khai.
* Mạng nội bộ chưa từng triển khai tiêu chuẩn mã hóa.
* Thiết bị nhúng hoặc thiết bị mạng có tài nguyên phần cứng hạn chế.
* Các máy chủ cấu hình sai khi không kích hoạt giải pháp an toàn thay thế.

Trong quá trình kiểm thử xâm nhập (pentest), việc tìm thấy một FTP server mở (đặc biệt là cho phép đăng nhập ẩn danh) là một phát hiện phổ biến và rất giá trị.

### Tương tác thủ công với FTP
Vì FTP truyền dữ liệu dạng văn bản rõ, bạn có thể dùng Telnet (hoặc Netcat) để kết nối trực tiếp và đóng vai trò như một FTP client. Các bước thực hiện:

1. Kết nối tới FTP server qua Telnet client tới port 21 (port mặc định của FTP).
2. Nhập tên đăng nhập bằng lệnh `USER frank`.
3. Nhập mật khẩu bằng lệnh `PASS D2xc9CgD`.
4. Sau khi nhập đúng, hệ thống xác nhận đăng nhập thành công.

Lệnh `STAT` giúp xem thêm thông tin trạng thái máy chủ. Lệnh `SYST` hiển thị loại hệ điều hành của mục tiêu (ở đây là UNIX). Lệnh `PASV` chuyển kết nối sang chế độ thụ động (Passive mode). FTP có hai chế độ hoạt động chính:

* Active (Chủ động): Dữ liệu được gửi qua kênh riêng bắt nguồn từ port 20 của server. Server sẽ chủ động khởi tạo kết nối dữ liệu ngược lại client. Cách này thường gặp lỗi nếu client đứng sau tường lửa hoặc NAT.
* Passive (Thụ động): Dữ liệu truyền qua kênh riêng bắt nguồn từ port phía client (lớn hơn port 1023). Client chủ động khởi tạo cả hai kết nối. Chế độ này thân thiện với tường lửa hơn và là lựa chọn mặc định trên hầu hết FTP client hiện đại.

Lệnh `TYPE A` chuyển sang chế độ truyền file dạng ASCII, còn `TYPE I` chuyển sang chế độ nhị phân (Binary). Tuy nhiên, bạn không thể tải trọn vẹn file chỉ bằng Telnet vì FTP yêu cầu một kênh kết nối dữ liệu độc lập.
> *tổng hợp các lệnh*
> 	USER, PASS
> 	STAT, SYST, 
> 	PASV
> 	TYPE A, TYPE I

```tex
pentester@TryHackMe$ telnet 10.48.165.135 21
Trying 10.48.165.135...
Connected to 10.48.165.135.
Escape character is '^]'.
220 (vsFTPd 3.0.3)
USER frank
331 Please specify the password.
PASS D2xc9CgD
230 Login successful.
SYST
215 UNIX Type: L8
PASV
227 Entering Passive Mode (10,10,0,148,78,223).
TYPE A
200 Switching to ASCII mode.
STAT
211-FTP server status:
     Connected to ::ffff:10.10.0.1
     Logged in as frank
     TYPE: ASCII
     No session bandwidth limit
     Session timeout in seconds is 300
     Control connection is plain text
     Data connections will be plain text
     At session startup, client count was 1
     vsFTPd 3.0.3 - secure, fast, stable
211 End of status
QUIT
221 Goodbye.
Connection closed by foreign host.

```

Ở kết quả lệnh `STAT`, máy chủ thông báo rõ `Control connection is plain text` và `Data connections will be plain text`, xác nhận toàn bộ lưu lượng lẫn tài khoản đăng nhập đều không được mã hóa.

### Cách FTP truyền tập tin
Khi truyền file qua FTP, client sẽ khởi tạo kết nối tới server qua port 21. Tất cả lệnh được gửi qua kênh điều khiển (control channel). Khi client yêu cầu tải file, một kết nối TCP thứ hai sẽ được thiết lập riêng để truyền tải dữ liệu (data channel). <u>Kiến trúc hai kênh này là lý do FTP thường gặp sự cố khi đi qua tường lửa.</u>
> *=>* đại khái là khi truyền tin thì ftp sử dụng 1 kênh tcp để truyền

![[Pasted image 20260815112059.png|547]]
### Sử dụng FTP Client

Để tải file, bạn cần một ứng dụng FTP client thực thụ. Sau khi đăng nhập thành công, dấu nhắc `ftp>` sẽ xuất hiện để bạn thao tác:

* `ls`: Liệt kê các file trong thư mục.
* `ascsdsaaaii`: Đổi sang chế độ ASCII khi truyền file văn bản.
* `get <tên_file>`: Thiết lập kênh dữ liệu riêng để bắt đầu tải file về.

```tex
pentester@TryHackMe$ ftp 10.48.165.135
Connected to 10.48.165.135.
220 (vsFTPd 3.0.3)
Name: frank
331 Please specify the password.
Password: D2xc9CgD
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
227 Entering Passive Mode (10,20,30,148,201,180).
150 Here comes the directory listing.
-rw-rw-r--    1 1001     1001         4006 Sep 15 10:27 README.txt
226 Directory send OK.
ftp> ascii
200 Switching to ASCII mode.
ftp> get README.txt
local: README.txt remote: README.txt
227 Entering Passive Mode (10,10,0,148,125,55).
150 Opening BINARY mode data connection for README.txt (4006 bytes).
WARNING! 9 bare linefeeds received in ASCII mode
File may not have transferred correctly.
226 Transfer complete.
4006 bytes received in 0.000269 secs (14892.19 Kbytes/sec)
ftp> exit
221 Goodbye.

```

### Đăng nhập FTP ẩn danh (Anonymous FTP)

Một số FTP server cho phép đăng nhập ẩn danh bằng username `anonymous` hoặc `ftp`, mật khẩu có thể nhập địa chỉ email tùy ý (hoặc để trống). Khi làm pentest, hãy luôn thử đăng nhập ẩn danh:

```text
ftp> USER anonymous
331 Please specify the password.
ftp> PASS anything@example.com
230 Login successful.

```

FTP server mở ẩn danh có thể vô tình để lộ các tài liệu nhạy cảm, file cấu hình sao lưu, hoặc mở ra đường tải file độc hại lên server nếu được cấp quyền ghi (write permission).

### Phần mềm máy chủ và ứng dụng Client

<u>Một số phần mềm FTP server </u>thông dụng:
* vsftpd: Rất phổ biến trên các bản phân phối Linux.
* ProFTPD: Tính tùy biến cao, hỗ trợ nhiều module.
* Pure-FTPd: Tập trung vào tính bảo mật và sự gọn nhẹ.
* IIS trên Windows có tích hợp sẵn tính năng FTP server.

Về client, ngoài lệnh `ftp` trên terminal, bạn có thể dùng các phần mềm giao diện đồ họa như FileZilla.<u> Các trình duyệt web hiện nay đều đã bỏ tính năng hỗ trợ FTP.</u>
*=> tại sao lại bỏ ftp trên trình duyệt??*
### Đánh giá mức độ an toàn

Do FTP gửi thông tin đăng nhập, lệnh thực thi và dữ liệu file dưới dạng văn bản rõ, lưu lượng FTP rất dễ bị tấn công nghe lén. Bất kỳ ai bắt được gói tin mạng đều có thể thấy rõ tài khoản, mật khẩu, nội dung file truyền tải và cấu trúc thư mục của hệ thống.

Nếu bắt buộc phải dùng FTP, hãy giới hạn nó trong các phân vùng mạng cô lập hoặc chuyển sang FTPS. Trong hầu hết trường hợp khác, SFTP chạy trên nền SSH là lựa chọn được ưu tiên hàng đầu.

### Câu hỏi thực hành
Sử dụng FTP client trên AttackBox để kết nối tới địa chỉ `10.48.165.135` với tài khoản: `frank / D2xc9CgD`.
![[Pasted image 20260815112517.png|570]]

# 5-SMTP
Email là một trong những dịch vụ được sử dụng phổ biến nhất trên Internet. Có nhiều cách cấu hình máy chủ email khác nhau; ví dụ, bạn có thể thiết lập hệ thống email nội bộ để người dùng trao đổi thư mà không cần kết nối Internet. Tuy nhiên, nội dung dưới đây xem xét mô hình tổng quát hơn: các máy chủ email kết nối với nhau qua Internet.

### Các thành phần phân phối Email

Việc gửi nhận email qua Internet yêu cầu các thành phần sau:
* Mail User Agent (MUA): Ứng dụng email client (như Thunderbird, Outlook hoặc giao diện webmail).
* Mail Submission Agent (MSA): Nhận thư từ MUA, kiểm tra lỗi và chuyển tiếp thư đi.
* Mail Transfer Agent (MTA): Định tuyến và chuyển giao thư giữa các máy chủ với nhau.
* Mail Delivery Agent (MDA): Lưu trữ email vào hộp thư của người nhận để người dùng lấy về.
![[Pasted image 20260815130443.png]]
5 bước để một email đến được hộp thư người nhận:
1. MUA có email cần gửi, kết nối tới MSA để nộp thư.
2. MSA nhận thư, kiểm tra lỗi trước khi chuyển sang MTA (thường nằm trên cùng một máy chủ).
3. MTA gửi email tới MTA của người nhận. MTA cũng có thể kiêm luôn vai trò MSA.
4. Ở mô hình thông thường, máy chủ MTA cũng đảm nhận luôn vai trò MDA.
5. Người nhận lấy thư từ MDA bằng ứng dụng client của họ (MUA).

Quy trình này tương tự việc gửi thư bưu chính:
* Bạn (MUA) muốn gửi một lá thư bưu điện.
* Nhân viên bưu điện (MSA) kiểm tra thư trước khi bưu cục địa phương (MTA) tiếp nhận.
* Bưu cục địa phương kiểm tra địa chỉ và chuyển thư tới bưu cục (MTA) tại nơi nhận.
* Bưu cục nơi nhận (MTA) đưa thư vào hòm thư (MDA) của người nhận.
* Người nhận (MUA) định kỳ kiểm tra hòm thư và lấy thư về đọc.

### Các giao thức Email
<u>Để giao tiếp với MTA và MDA, hệ thống dựa vào các giao thức sau:</u>

* Simple Mail Transfer Protocol *(SMTP): Dùng để gửi email.*
* Post Office Protocol version 3 (POP3) hoặc Internet Message Access Protocol (IMAP): Dùng để nhận email.

### Cổng SMTP và Mã hóa
*SMTP dùng để giao tiếp với máy chủ MTA.* Giao thức SMTP ban đầu sử dụng dạng văn bản rõ (cleartext), mọi câu lệnh được gửi đi mà không qua mã hóa. Hiện nay, hạ tầng email sử dụng nhiều cổng với các cơ chế bảo mật khác nhau:

* Port 25: Cổng SMTP truyền thống dùng cho giao tiếp giữa các máy chủ (MTA sang MTA). Cổng này thường bị các ISP chặn ở mạng gia đình để chống spam. Mã hóa trên port 25 là tùy chọn và được thiết lập qua lệnh STARTTLS.
* Port 587: Cổng submission, dùng cho email client (MUA) gửi thư tới mail server (MSA). Đây là cổng khuyến nghị khi gửi thư và thường yêu cầu xác thực. <u>Mã hóa TLS được kích hoạt thông qua lệnh</u> *STARTTLS*.
* Port 465: Ban đầu dành cho SMTPS (SMTP qua TLS trực tiếp). Trên cổng này, kết nối TLS được thiết lập ngay từ đầu.

### Gửi Email thủ công bằng Telnet

Vì SMTP hỗ trợ văn bản rõ, bạn có thể dùng Telnet client kết nối tới SMTP server và đóng vai trò như một ứng dụng gửi thư (MUA). Sau khi kết nối, sử dụng lệnh `helo hostname` (hoặc `ehlo hostname` cho chuẩn ESMTP) rồi bắt đầu soạn thư:

```text
pentester@TryHackMe$ telnet 10.49.143.8 25
Trying 10.49.143.8...
Connected to 10.49.143.8.
Escape character is '^]'.
220 bento.localdomain ESMTP Postfix (Ubuntu)
helo telnet
250 bento.localdomain
mail from:
250 2.1.0 Ok
rcpt to:
250 2.1.5 Ok
data
354 End data with .
subject: Sending email with Telnet
Hello Frank,
I am just writing to say hi!
.
250 2.0.0 Ok: queued as C3E7F45F06
quit
221 2.0.0 Bye
Connection closed by foreign host.

```

Sau lệnh `helo`, các lệnh `mail from:` và `rcpt to:` xác định người gửi và người nhận. Lệnh `data` mở đầu phần nội dung thư. Để kết thúc nội dung, nhập một dấu chấm đứng riêng trên một dòng (`.` rồi nhấn Enter). Sau đó máy chủ SMTP sẽ đưa thư vào hàng đợi xử lý.

### Giả mạo Email (Email Spoofing)
Trong ví dụ trên, địa chỉ người gửi (from) được nhập thủ công và máy chủ tiếp nhận mà không kiểm tra xem người gửi có thực sự sở hữu địa chỉ email đó hay không. Đây chính là cách email spoofing hoạt động. Giao thức SMTP ra đời trong thời kỳ mạng máy tính dựa trên sự tin cậy lẫn nhau, không tích hợp sẵn cơ chế xác thực danh tính người gửi.

Do đó, các email lừa đảo (phishing) có thể hiển thị như được gửi từ các địa chỉ hợp lệ nếu hệ thống không có các lớp kiểm tra bổ sung.

### Ý nghĩa trong bảo mật

* Email vẫn là hướng tấn công hàng đầu cho các chiến dịch phishing.
* Máy chủ mail bị cấu hình sai có thể trở thành open relay bị lợi dụng để phát tán thư rác.
* SMTP không mã hóa làm lộ nội dung thư và thông tin tài khoản khi bị nghe lén mạng.
* Hiểu rõ SMTP giúp việc phân tích header email trong xử lý sự cố (incident response) chính xác hơn.

Các cơ chế xác minh hiện đại như SPF, DKIM, DMARC được phát triển để đối phó với tình trạng giả mạo email.

