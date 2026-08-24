# 2-DNS
Bạn có nhớ địa chỉ IP của các trang web yêu thích của mình không? Trừ khi đó là địa chỉ IP private của một thiết bị trong mạng nội bộ, hầu như không ai cần phải bận tâm ghi nhớ các địa chỉ IP. Điều này có được phần lớn nhờ vào Hệ thống Tên miền (DNS - Domain Name System), nơi chịu trách nhiệm ánh xạ chính xác một tên miền sang một địa chỉ IP.

DNS hoạt động ở tầng Application (tức Tầng 7 trong mô hình ISO OSI). Mặc định, lưu lượng DNS sử dụng UDP port 53 và dùng TCP port 53 làm phương án dự phòng. Có rất nhiều loại bản ghi DNS (DNS records); tuy nhiên, trong bài học này, chúng ta sẽ tập trung vào 4 loại sau:

* **A record:** Bản ghi A (Address) <u>ánh xạ một hostname tới một hoặc nhiều địa chỉ IPv4</u>. Ví dụ, bạn có thể thiết lập `example.com` phân giải về địa chỉ `172.17.2.172`.
* **AAAA record:** Bản ghi AAAA tương tự như bản ghi A, nhưng <u>dành cho IPv6</u>. Hãy nhớ rằng nó là AAAA (quad-A), vì AA và AAA thường dùng để chỉ kích thước pin; ngoài ra AAA còn là viết tắt của Authentication, Authorization và Accounting; cả hai đều không thuộc về DNS.
* **CNAME record:** Bản ghi CNAME (Canonical Name) <u>ánh xạ một tên miền sang một tên miền khác</u>. Ví dụ, `[www.example.com](https://www.example.com)` có thể được ánh xạ tới `example.com` hoặc thậm chí là `example.org`.
* **MX record:** Bản ghi MX (Mail Exchange) <u>chỉ định mail server chịu trách nhiệm xử lý email</u> cho một tên miền.
*=>*[[DNS record là gi]]
Nói cách khác, khi bạn gõ `example.com` vào trình duyệt, trình duyệt sẽ cố gắng phân giải tên miền này bằng cách gửi truy vấn tìm bản ghi A tới DNS server. Tuy nhiên, khi bạn gửi một email tới `test@example.com`, mail server sẽ truy vấn DNS server để tìm bản ghi MX.

Nếu muốn tra cứu địa chỉ IP của một tên miền từ dòng lệnh, bạn có thể sử dụng công cụ như **nslookup**. Hãy xem ví dụ trong terminal bên dưới khi chúng ta tra cứu `example.com`:

```text
user@TryHackMe$ nslookup www.example.com
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
Name:   www.example.com
Address: 93.184.215.14
Name:   www.example.com
Address: 2606:2800:21f:cb07:6820:80da:af6b:8b2c

```

Truy vấn trên đã tạo ra 4 gói tin. Trong terminal bên dưới, chúng ta có thể thấy gói tin thứ nhất và thứ ba lần lượt gửi các truy vấn DNS cho bản ghi A và AAAA. Gói tin thứ hai và thứ tư hiển thị phản hồi cho các truy vấn DNS đó:
![[Pasted image 20260811130849.png]]
# 3-WHOIS
Trong bài học trước, chúng ta đã tìm hiểu cách một tên miền được phân giải thành địa chỉ IP. Tuy nhiên, để điều này xảy ra, phải có ai đó có quyền thiết lập các bản ghi A, AAAA, MX cũng như các bản ghi DNS khác cho tên miền đó. Bất kỳ ai đăng ký tên miền đều được cấp quyền này. Do đó, <u>nếu bạn đăng ký example.com, bạn có thể thiết lập bất kỳ bản ghi DNS hợp lệ nào cho example.com.</u>

Bạn có thể đăng ký bất kỳ tên miền nào còn trống trong thời hạn một hoặc nhiều năm. Bạn cần nộp phí hằng năm và bắt buộc phải cung cấp thông tin liên hệ chính xác với tư cách là người đăng ký. <u>Thông tin này nằm trong dữ liệu của bản ghi WHOIS</u> và được công khai. (Mặc dù được viết hoa toàn bộ, WHOIS không phải là từ viết tắt; nó được đọc là "who is".) Tuy nhiên, đừng quá lo lắng nếu bạn muốn đăng ký tên miền mà không muốn công khai thông tin cá nhân; bạn có thể sử dụng các dịch vụ bảo vệ quyền riêng tư để ẩn toàn bộ dữ liệu này khỏi bản ghi WHOIS.

<u>Bạn có thể tra cứu bản ghi WHOIS của bất kỳ tên miền nào đã đăng ký bằng các dịch vụ trực tuyến hoặc thông qua công cụ dòng lệnh whois trên Linux và một số nền tảng khác</u>. Bản ghi WHOIS cung cấp thông tin về chủ thể đăng ký tên miền, bao gồm tên, số điện thoại, email và địa chỉ. Trong ảnh chụp màn hình bên dưới, bạn có thể thấy thời điểm bản ghi được tạo lần đầu và lần cập nhật gần nhất. Ngoài ra, bạn cũng tìm thấy tên, địa chỉ, số điện thoại và email của người đăng ký.
![[Pasted image 20260811131408.png]]
Trong kết quả hiển thị trên terminal bên dưới, chúng tôi đã sử dụng lệnh whois để tra cứu một tên miền có bản ghi WHOIS được bảo vệ bởi dịch vụ riêng tư.

# 4-HTTP(s)
Khi mở trình duyệt, bạn chủ yếu sử dụng các giao thức HTTP và HTTPS. HTTP là viết tắt của Hypertext Transfer Protocol; chữ S trong HTTPS đại diện cho Secure. Giao thức này dựa trên TCP và định nghĩa cách trình duyệt web của bạn giao tiếp với các web server.

Một số lệnh hoặc phương thức mà trình duyệt web của bạn thường gửi đến web server gồm:

* GET:<u> Lấy dữ liệu từ server</u>, chẳng hạn như file HTML hoặc hình ảnh.
* POST: Cho phép gửi dữ liệu mới lên server, chẳng hạn như gửi biểu mẫu (form) hoặc tải file lên.
* PUT: Dùng để tạo một tài nguyên mới trên server hoặc cập nhật, ghi đè lên thông tin hiện có.
* DELETE: Đúng như tên gọi, dùng để xóa một file hoặc tài nguyên cụ thể trên server.

HTTP và HTTPS thường sử dụng các TCP port lần lượt là 80 và 443, hoặc ít phổ biến hơn là các port như 8080 và 8443.

Trong ví dụ sau, chúng ta sử dụng trình duyệt Firefox để truy cập web server tại địa chỉ MACHINE_IP. Trình duyệt tải trang web và hiển thị hoàn hảo; tuy nhiên, điều chúng ta quan tâm là những gì diễn ra đằng sau hậu trường.

Sử dụng Wireshark, chúng ta có thể quan sát kỹ hơn quá trình trao đổi giữa trình duyệt Firefox và web server. Ảnh chụp màn hình từ Wireshark hiển thị văn bản do <u>trình duyệt gửi đi bằng màu đỏ và phản hồi từ web server bằng màu xanh dương</u>. Có thể thấy, rất nhiều thông tin được trao đổi giữa client và server mà không hiển thị trực tiếp cho người dùng, chẳng hạn như phiên bản của web server hoặc thời điểm trang web được chỉnh sửa lần cuối.
![[Pasted image 20260811132335.png]]

Nhớ lại kiến thức trong bài Networking Concepts, chúng ta từng dùng công cụ telnet để kết nối tới web server chạy trên MACHINE_IP tại port 80. Chúng ta đã phải gửi một vài dòng lệnh: `GET / HTTP/1.1` và `Host: anything` để lấy trang mong muốn. (Trên một số server, bạn có thể nhận được file mà không cần gửi `Host: anything`.) Bạn có thể dùng phương pháp này để truy cập bất kỳ trang nào chứ không chỉ trang mặc định `/`. <u>Ví dụ, để lấy `file.html`, bạn sẽ gửi `GET /file.html HTTP/1.1` (hoặc chỉ `GET /file.html` </u>tùy thuộc vào web server đang dùng). Cách tiếp cận này rất hiệu quả cho việc xử lý sự cố (troubleshooting) vì bạn đang trực tiếp "giao tiếp" với server.
[[Active Reconnaissance#5-Telnet]]
![[Pasted image 20260811132914.png]]
# 5-FTP
Khác với HTTP vốn được thiết kế để lấy các trang web, File Transfer Protocol (FTP) được thiết kế để truyền tải tập tin. Nhờ đó, FTP rất hiệu quả trong việc truyền file và trong cùng điều kiện, nó có thể đạt tốc độ cao hơn HTTP.

<u>Một số lệnh</u> ví dụ được định nghĩa bởi giao thức FTP:

* **USER**: nhập tên người dùng
* **PASS**: nhập mật khẩu
* **RETR** (retrieve): tải tập tin từ FTP server về client
* **STOR** (store): tải tập tin từ client lên FTP server

Mặc định, FTP server lắng nghe trên TCP port 21; việc truyền dữ liệu được thực hiện qua một kết nối riêng biệt khác giữa client và server.

Trong terminal bên dưới, chúng tôi đã thực thi lệnh `ftp 10.49.166.206` để kết nối tới FTP server từ xa bằng FTP client cục bộ. <u>Sau đó tiến hành các bước</u>:

* Sử dụng username `anonymous` để đăng nhập
* Không cần cung cấp mật khẩu
* Chạy lệnh `ls` để hiển thị danh sách các file có sẵn để tải về
* Dùng `type ascii` để chuyển sang chế độ ASCII vì đây là file văn bản
* Dùng `get coffee.txt` để tải file mong muốn về

Quá trình trao đổi lệnh qua FTP client hiển thị như sau:

```tex
user@TryHackMe$ ftp 10.49.166.206
Connected to 10.49.166.206 (10.49.166.206).
220 (vsFTPd 3.0.5)
Name (10.49.166.206:strategos): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
227 Entering Passive Mode (10,10,41,192,134,10).
150 Here comes the directory listing.
-rw-r--r--    1 0        0            1480 Jun 27 08:03 coffee.txt
-rw-r--r--    1 0        0              14 Jun 27 08:04 flag.txt
-rw-r--r--    1 0        0            1595 Jun 27 08:05 tea.txt
226 Directory send OK.
ftp> type ascii
200 Switching to ASCII mode.
ftp> get coffee.txt
local: coffee.txt remote: coffee.txt
227 Entering Passive Mode (10,10,41,192,57,100).
150 Opening BINARY mode data connection for coffee.txt (1480 bytes).
WARNING! 47 bare linefeeds received in ASCII mode
File may not have transferred correctly.
226 Transfer complete.
1480 bytes received in 8e-05 secs (18500.00 Kbytes/sec)
ftp> quit
221 Goodbye.

```


Chúng tôi sử dụng Wireshark để quan sát kỹ hơn các thông điệp được trao đổi. Thông điệp của client có màu đỏ, còn phản hồi của server có màu xanh dương. Hãy lưu ý sự khác biệt giữa một số lệnh giao diện client và lệnh giao thức thực tế. Ví dụ, khi bạn gõ `ls` trên client, client sẽ gửi lệnh `LIST` tới server. Lưu ý cuối cùng là danh sách thư mục (`LIST`) và file tải về đều được truyền qua các kết nối dữ liệu riêng biệt.
![[Pasted image 20260811133619.png|494]]
![[Pasted image 20260811133950.png|566]]
# 6-SMTP sending email
Giống như việc duyệt web hay tải tập tin, gửi email cũng cần có giao thức riêng. Simple Mail Transfer Protocol <u>(SMTP) định nghĩa cách mail client giao tiếp với mail server và cách các mail server giao tiếp với nhau.</u>

Có thể ví giao thức SMTP như việc bạn đến bưu điện địa phương để gửi một kiện hàng. Bạn chào nhân viên, cho họ biết nơi muốn gửi, cung cấp thông tin người gửi trước khi giao kiện hàng. Tùy thuộc vào từng quốc gia, bạn có thể được yêu cầu xuất trình thẻ căn cước. Quy trình này không khác biệt nhiều so với một phiên làm việc SMTP.
>bưu điện ứng với *mail server,* còn bản thân người gửi là *mail client*

Dưới đây là một số<u> lệnh mà mail client sử dụng khi truyền email tới SMTP server:</u>
* HELO hoặc EHLO: Khởi tạo phiên làm việc SMTP
* MAIL FROM: Chỉ định địa chỉ email người gửi
* RCPT TO: Chỉ định địa chỉ email người nhận
* DATA: Bắt đầu gửi nội dung thông điệp email
* Dấu chấm (`.`): Được gửi trên một dòng riêng biệt để đánh dấu kết thúc nội dung email

Terminal bên dưới hiển thị ví dụ về một email được gửi qua telnet. <u>Mặc định, SMTP server lắng nghe trên TCP port 25</u>.

```tex
user@TryHackMe$ telnet 10.49.166.206 25
Trying 10.49.166.206...
Connected to 10.49.166.206.
Escape character is '^]'.
220 example.thm ESMTP Exim 4.95 Ubuntu Thu, 27 Jun 2024 16:18:09 +0000
HELO client.thm
250 example.thm Hello client.thm [10.11.81.126]
MAIL FROM: <user@client.thm>
250 OK
RCPT TO: <strategos@server.thm>
250 Accepted
DATA
354 Enter message, ending with "." on a line by itself
From: user@client.thm
To: strategos@server.thm
Subject: Telnet email

Hello. I am using telnet to send you an email!
.
250 OK id=1sMrpq-0001Ah-UT
QUIT
221 example.thm closing connection
Connection closed by foreign host.

```

Rõ ràng, việc gửi email bằng telnet khá cồng kềnh; tuy nhiên, nó giúp bạn hiểu rõ hơn các lệnh mà email client thực thi ngầm bên dưới. File bắt gói tin Wireshark hiển thị quá trình trao đổi qua màu sắc: thông điệp của client có màu đỏ, còn phản hồi của server có màu xanh dương.
![[Pasted image 20260811134714.png|487]]

Khi đã tìm hiểu một số lệnh cơ bản của HTTP, FTP và SMTP, bạn đã nắm vững cách các giao thức được thiết kế và vận hành. Việc tìm hiểu cách hoạt động của các giao thức dạng văn bản khác như POP3 hay IMAP cũng sẽ trở nên dễ dàng hơn.

# 7-POP3
Bạn nhận được một email và muốn tải nó về mail client cục bộ của mình. Post Office Protocol phiên bản 3 (POP3) được thiết kế để<u> cho phép client giao tiếp với mail server</u> và lấy các tin nhắn email về.

Không cần đi sâu vào chi tiết kỹ thuật phức tạp, <u>một email client gửi thư đi bằng cách dựa vào SMTP và lấy thư về bằng POP3</u>. SMTP giống như việc bạn giao phong bì hoặc bưu kiện cho bưu điện, còn POP3 tương tự như việc bạn kiểm tra hòm thư nhà mình để nhận thư mới.
>SMTP để gửi thư, còn POP3 dừng để lấy thư về

<u>Một số lệnh POP3 phổ biến:</u>

* USER : xác định người dùng
* PASS : cung cấp mật khẩu người dùng
* STAT: yêu cầu số lượng thư và tổng dung lượng
* LIST: liệt kê tất cả thư cùng dung lượng của chúng
* RETR <message_number>: lấy thư được chỉ định về
* DELE <message_number>: đánh dấu một thư để xóa
* QUIT: kết thúc phiên POP3 và áp dụng các thay đổi (như xóa thư)

Trong terminal bên dưới, chúng ta có thể thấy một phiên làm việc POP3 qua telnet. Do <u>POP3 server mặc định lắng nghe trên TCP port 110</u>, lệnh kết nối là `telnet 10.49.166.206 110`. Quá trình trao đổi bên dưới lấy về email đã được gửi ở bài học trước.

```tex
user@TryHackMe$ telnet 10.49.166.206 110
Trying 10.49.166.206...
Connected to 10.49.166.206.
Escape character is '^]'.
+OK [XCLIENT] Dovecot (Ubuntu) ready.
AUTH
+OK
PLAIN
.
USER strategos
+OK
PASS 
+OK Logged in.
STAT
+OK 3 1264
LIST
+OK 3 messages:
1 407
2 412
3 445
.
RETR 3
+OK 445 octets
Return-path: <user@client.thm>
Envelope-to: strategos@server.thm
Delivery-date: Thu, 27 Jun 2024 16:19:35 +0000
Received: from [10.11.81.126] (helo=client.thm)
        by example.thm with smtp (Exim 4.95)
        (envelope-from <user@client.thm>)
        id 1sMrpq-0001Ah-UT
        for strategos@server.thm;
        Thu, 27 Jun 2024 16:19:35 +0000
From: user@client.thm
To: strategos@server.thm
Subject: Telnet email

Hello. I am using telnet to send you an email!
.
QUIT
+OK Logging out.
Connection closed by foreign host.

```

<u>Bất kỳ ai bắt gói tin trên mạng đều có thể chặn lưu lượng trao đổi này</u>. Giống như các file bắt gói tin bằng Wireshark trước đó, các lệnh màu đỏ do client gửi, còn các dòng màu xanh là phản hồi của server. <u>Rõ ràng là người bắt lưu lượng mạng có thể đọc được cả mật khẩu dưới dạng văn bản thuần</u>.
![[Pasted image 20260811135409.png|493]]
# 8-imap
[POP3] là đủ khi bạn chỉ làm việc trên một thiết bị, chẳng hạn như ứng dụng email trên máy tính cá nhân. <u>Tuy nhiên, nếu bạn muốn kiểm tra email từ máy tính ở cơ quan, laptop và cả điện thoại thông minh thì sao</u>? Khi đó, bạn cần một giao thức cho phép đồng bộ hóa tin nhắn thay vì xóa chúng sau khi tải về. <u>Giải pháp để duy trì hộp thư đồng bộ trên nhiều thiết bị chính là Internet Message Access Protocol (IMAP).
</u>
  
IMAP cho phép đồng bộ các thao tác đọc, di chuyển và xóa thư. Giao thức này rất tiện lợi khi bạn sử dụng nhiều thiết bị hoặc ứng dụng để kiểm tra email. Khác với POP3 (thường tối ưu không gian lưu trữ trên server do thư bị xóa sau khi tải về), IMAP tốn nhiều dung lượng lưu trữ hơn vì toàn bộ email được giữ lại trên server và đồng bộ tới tất cả thiết bị.

Các lệnh của giao thức IMAP phức tạp hơn POP3. Dưới đây là một số lệnh ví dụ:
- LOGIN : xác thực người dùng
- SELECT : chọn thư mục hộp thư để làm việc
- FETCH <mail_number> <data_item_name>: ví dụ FETCH 3 BODY[] để lấy toàn bộ header và nội dung của thư số 3
- MOVE <sequence_set> : di chuyển các thư được chỉ định sang hộp thư khác
- COPY <sequence_set> <data_item_name>: sao chép các thư được chỉ định sang hộp thư khác
- LOGOUT: đăng xuất
Mặc định, IMAP server lắng nghe trên TCP port 143. Chúng ta sẽ dùng telnet kết nối tới port 143 của IP 10.48.135.6 để lấy thư đã gửi ở bài học trước.
# 9-TỔNG KẾT
![[Pasted image 20260811142155.png]]