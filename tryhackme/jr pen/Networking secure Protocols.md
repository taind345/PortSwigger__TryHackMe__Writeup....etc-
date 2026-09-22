# 2-TLS
Đã có thời điểm bạn chỉ cần một công cụ bắt gói tin là có thể đọc toàn bộ đoạn chat, email và mật khẩu của những người dùng trong cùng mạng. Việc một kẻ tấn công chuyển card mạng sang chế độ promiscuous (chế độ hỗn tạp) để bắt tất cả gói tin — kể cả những gói không gửi cho mình — là điều không hề hiếm gặp. Sau đó, họ sẽ duyệt lại file pcap để thu thập thông tin đăng nhập của các nạn nhân. Người dùng hoàn toàn không có cách nào ngăn mật khẩu của mình bị gửi dưới dạng văn bản thuần (cleartext). Ngày nay, việc gặp một dịch vụ gửi thông tin đăng nhập ở dạng cleartext đã trở nên rất hiếm.

Vào đầu những năm 1990, Netscape Communications đã nhận ra tầm quan trọng của việc bảo mật truyền thông trên World Wide Web. Họ phát triển SSL (Secure Sockets Layer) và phát hành phiên bản công khai đầu tiên là SSL 2.0 vào năm 1995. Đến năm 1999, Internet Engineering Task Force (IETF) đã phát triển TLS (Transport Layer Security). Dù rất giống nhau, TLS 1.0 là bản nâng cấp từ SSL 3.0 với nhiều biện pháp bảo mật được cải tiến. Năm 2018, giao thức này trải qua đợt đại trùng tu và phiên bản TLS 1.3 ra đời. Điểm cốt lõi không phải là ghi nhớ chính xác các mốc thời gian, mà là nhận ra công sức và thời gian đã được đầu tư để hoàn thiện TLS 1.3. Trải qua hơn hai thập kỷ, mỗi phiên bản đều mang lại những bài học kinh nghiệm và sự cải tiến vượt bậc.

Tương tự như SSL, <u>TLS là một giao thức mã hóa hoạt động ở tầng Transport của mô hình OSI</u>. Nó cho phép <u>truyền thông an toàn giữa client và server qua một mạng không an toàn</u>. Tính an toàn ở đây đề cập đến tính bảo mật (confidentiality) và tính toàn vẹn (integrity); <u>TLS đảm bảo không ai có thể đọc hay chỉnh sửa dữ liệu được trao đổi</u>. Hãy dành một phút suy ngẫm xem việc mua sắm, giao dịch ngân hàng, hay gửi tin nhắn, email trực tuyến sẽ ra sao nếu không thể bảo đảm tính bảo mật và toàn vẹn của các gói tin. Nếu không có TLS, chúng ta sẽ không thể sử dụng Internet cho nhiều ứng dụng đang phục vụ đời sống hằng ngày.

<u>Hiện nay, hàng chục giao thức đã được nâng cấp bảo mật chỉ bằng cách tích hợp thêm TLS</u>. <u>Ví dụ như HTTP, DNS, MQTT và SIP đã trở thành HTTPS, DoT (DNS over TLS), MQTTS và SIPS, </u>trong đó chữ "S" thêm vào đại diện cho Secure nhờ việc ứng dụng SSL/TLS. Trong các bài tiếp theo, chúng ta sẽ tìm hiểu về HTTPS, SMTPS, POP3S và IMAPS.

### Kiến thức nền tảng kỹ thuật

Chúng ta sẽ không đi sâu vào quá trình bắt tay (handshake) của TLS; bài viết này chỉ cung cấp tổng quan về cách TLS được thiết lập và vận hành.

Bước đầu tiên đ<u>ể bất kỳ server (hoặc client) nào muốn xác thực danh tính là sở hữu một chứng chỉ TLS đã được ký</u>. Thông thường, quản trị viên server sẽ tạo một yêu cầu ký chứng chỉ (**CSR** - Certificate Signing Request) và gửi tới Tổ chức phát hành chứng chỉ (**CA** - Certificate Authority). **CA** sẽ kiểm tra **CSR** và cấp chứng chỉ số. Khi nhận được chứng chỉ đã ký, server (hoặc client) có thể dùng nó để chứng minh danh tính với bên khác, và bên kia có thể xác minh tính hợp lệ của chữ ký. Để một máy chủ xác nhận tính hợp lệ của chứng chỉ, các chứng chỉ của cơ quan cấp phát (CA) tương ứng phải được cài đặt sẵn trên máy đó. Trong thế giới thực, điều này tương tự việc nhận biết con dấu của các cơ quan có thẩm quyền.

Nói chung, việc cấp chứng chỉ được ký thường tốn chi phí hằng năm. Tuy nhiên, dịch vụ Let's Encrypt cho phép bạn nhận chứng chỉ đã ký hoàn toàn miễn phí.

Cuối cùng, cần lưu ý rằng một số người dùng lựa chọn tạo chứng chỉ tự ký (self-signed certificate). Chứng chỉ tự ký không thể chứng minh tính xác thực của server vì không có bên thứ ba nào đứng ra xác nhận.
# 3-HTTPS
### HTTP
Như đã học trong bài Networking Core Protocols, HTTP dựa trên TCP và mặc định sử dụng port 80. Chúng ta cũng thấy toàn bộ lưu lượng HTTP được gửi dưới dạng văn bản thuần (cleartext) để bất kỳ ai cũng có thể chặn và theo dõi. Ảnh chụp màn hình bên dưới từ bài trước cho thấy rõ cách một kẻ tấn công dễ dàng đọc toàn bộ lưu lượng trao đổi giữa client và server.
![[Pasted image 20260811163744.png]]
Hãy cùng điểm lại các bước phổ biến nhất trước khi trình duyệt web yêu cầu một trang qua HTTP. Sau khi phân giải tên miền thành địa chỉ IP, client sẽ thực hiện hai bước sau:

1. Thiết lập bắt tay 3 bước (three-way handshake) TCP với server đích.
2. Giao tiếp bằng giao thức HTTP; ví dụ gửi các yêu cầu như `GET / HTTP/1.1`.

Hai bước trên được hiển thị trong cửa sổ bên dưới. Ba gói tin cho quá trình bắt tay TCP (được đánh dấu 1) diễn ra trước gói tin HTTP đầu tiên chứa lệnh GET. Luồng giao tiếp HTTP được đánh dấu 2. Ba gói tin cuối cùng dùng để kết thúc kết nối TCP và được đánh dấu 3.![[Pasted image 20260811164022.png]]

### HTTP Over TLS

HTTPS là viết tắt của Hypertext Transfer Protocol Secure.<u> Về bản chất, đây là HTTP chạy trên TLS</u>. Do đó, việc yêu cầu một trang qua HTTPS sẽ cần ba bước sau (sau khi phân giải tên miền):

1. Thiết lập bắt tay 3 bước TCP với server đích.
2. Thiết lập một phiên TLS.
3. Giao tiếp bằng giao thức HTTP; ví dụ gửi các yêu cầu HTTP như `GET / HTTP/1.1`
Ảnh chụp màn hình bên dưới cho thấy một phiên TCP được thiết lập trong 3 gói tin đầu tiên (đánh dấu 1). Sau đó, nhiều gói tin được trao đổi để thương lượng giao thức TLS (đánh dấu 2). <u>Vùng 1 và 2 là nơi diễn ra quá trình thương lượng và thiết lập TLS.</u>
![[Pasted image 20260811164147.png]]
Cuối cùng, dữ liệu ứng dụng HTTP được trao đổi (đánh dấu 3). Nhìn vào ảnh Wireshark, chúng ta thấy nó hiển thị "Application Data" vì không có cách nào biết liệu đó thực sự là HTTP hay một giao thức nào khác gửi qua port 443.

Đúng như dự đoán,<u> nếu ai đó cố gắng theo dõi luồng gói tin và ghép nối toàn bộ nội dung của chúng, họ chỉ nhận được chuỗi ký tự vô nghĩa (gibberish).</u> Lưu lượng trao đổi đã được mã hóa; văn bản màu đỏ do client gửi và màu xanh do server gửi. Không thể biết nội dung nếu không có khóa mã hóa.
![[Pasted image 20260811164410.png]]

### Lấy khóa mã hóa

<u>Việc bổ sung TLS vào HTTP khiến tất cả gói tin bị mã hóa.</u> Chúng ta không thể xem nội dung các gói tin trao đổi trừ khi truy cập được vào **private key**. Dù khả năng có được khóa dùng để mã hóa trong một phiên thực tế là rất thấp, chúng tôi đã thực hiện lại việc chụp màn hình sau khi <u>cung cấp khóa giải mã cho Wireshark. </u>Quá trình bắt tay TCP và TLS không thay đổi; điểm khác biệt chính bắt đầu từ giao thức HTTP được đánh dấu 3. Chẳng hạn, chúng ta có thể thấy thời điểm client gửi lệnh GET.*=> tức là cung cấp khóa gaiir mã cho wireshark , thì mình sẽ đọc được gói tin http được truyền bởi TLS*
![[Pasted image 20260811164722.png]]
Nếu muốn xem dữ liệu được trao đổi, bây giờ là cơ hội của bạn! Đó vẫn là lưu lượng HTTP thông thường nhưng đã được ẩn khỏi những ánh mắt tò mò.
![[Pasted image 20260811164839.png]]
Bài học cốt lõi là TLS cung cấp tính bảo mật cho HTTP mà không đòi hỏi bất kỳ thay đổi nào ở các giao thức tầng thấp hơn hoặc cao hơn. Nói cách khác, TCP và IP không bị chỉnh sửa, trong khi<u> HTTP được gửi qua TLS tương tự như cách nó được gửi qua TCP.</u>

# 4- SMTPS,POP3S , IMAPS
Việc thêm TLS vào SMTP, POP3 và IMAP không khác gì việc thêm TLS vào HTTP. Tương tự như cách HTTP thêm hậu tố S (Secure) để trở thành HTTPS, các giao thức SMTP, POP3 và IMAP lần lượt trở thành SMTPS, POP3S và IMAPS. Việc sử dụng các giao thức này qua TLS hoàn toàn tương tự như dùng HTTP qua TLS; do đó, hầu hết các điểm đã thảo luận ở phần HTTPS đều áp dụng cho các giao thức này.

Các phiên bản không mã hóa sử dụng các số port TCP mặc định dưới đây:

| Giao thức | Số Port mặc định |
| --- | --- |
| HTTP | 80 |
| SMTP | 25 |
| POP3 | 110 |
| IMAP | 143 |

Các phiên bản an toàn (chạy qua TLS) sử dụng các số port TCP mặc định sau:

| Giao thức | Số Port mặc định |
| --- | --- |
| HTTPS | 443 |
| SMTPS | 465 và 587 |
| POP3S | 995 |
| IMAPS | 993 |

TLS có thể được tích hợp vào nhiều giao thức khác; nguyên lý và lợi ích mang lại cũng tương tự.
# 5- SSH
Chúng ta đã sử dụng giao thức TELNET trong room Networking Concepts. Mặc dù rất tiện lợi để đăng nhập và quản trị hệ thống từ xa, <u>việc gửi toàn bộ lưu lượng dưới dạng văn bản thuần (cleartext) lại tiềm ẩn nhiều rủi r</u>o. Bất kỳ ai giám sát lưu lượng mạng đều có thể dễ dàng thu thập thông tin đăng nhập của bạn khi dùng telnet. Vấn đề này đòi hỏi một giải pháp mới. Tatu Ylönen đã phát triển giao thức Secure Shell (SSH) và phát hành SSH-1 vào năm 1995 dưới dạng phần mềm miễn phí. (Đáng chú ý, đây cũng là năm Netscape Communications phát hành giao thức SSL 2.0.) Một phiên bản an toàn hơn là SSH-2 được định nghĩa vào năm 1996. Đến năm 1999, các nhà phát triển OpenBSD đã phát hành OpenSSH, một bản triển khai mã nguồn mở của SSH. Ngày nay, khi bạn sử dụng một SSH client, rất có thể nó dựa trên các thư viện và mã nguồn của OpenSSH.
>*SSH* là phiên bản bảo mật hơn cho *telnet* [[Active Reconnaissance#5-Telnet]]

OpenSSH mang lại nhiều lợi ích quan trọng:
* Xác thực an toàn: Ngoài xác thực bằng mật khẩu, <u>SSH hỗ trợ xác thực bằng public key và xác thực hai yếu tố (2FA).</u>
* Tính bảo mật: <u>OpenSSH cung cấp mã hóa đầu-cuối </u>(end-to-end encryption), chống lại việc nghe lén. Ngoài ra, nó sẽ thông báo cho bạn khi có key mới của server để phòng chống tấn công man-in-the-middle.
* Tính toàn vẹn: Bên cạnh việc bảo vệ tính bảo mật của dữ liệu trao đổi, kỹ thuật mã hóa cũng bảo vệ tính toàn vẹn của lưu lượng mạng.
* Tạo đường hầm (Tunneling): SSH có thể tạo một "đường hầm" an toàn để định tuyến các giao thức khác qua SSH. Thiết lập này tạo ra một kết nối tương tự VPN.
* X11 Forwarding: Nếu bạn kết nối tới một hệ thống họ Unix có giao diện đồ họa, SSH cho phép bạn chạy ứng dụng đồ họa đó qua mạng.

Bạn có thể sử dụng lệnh `ssh username@hostname` để kết nối tới SSH server. Nếu tên người dùng trùng với username bạn đang đăng nhập, bạn chỉ cần gõ `ssh hostname`. Sau đó hệ thống sẽ yêu cầu nhập mật khẩu; tuy nhiên, nếu sử dụng xác thực public key, bạn sẽ được đăng nhập ngay lập tức.
![[Pasted image 20260811170341.png]]
Ảnh chụp màn hình bên dưới minh họa ví dụ chạy Wireshark trên một hệ thống Kali Linux từ xa. Tham số `-X` là bắt buộc để hỗ trợ chạy giao diện đồ họa, ví dụ: `ssh 192.168.124.148 -X`. (Hệ thống cục bộ cần cài đặt sẵn môi trường đồ họa phù hợp.)

Trong khi TELNET server lắng nghe trên port 23, thì SSH server lắng nghe trên port 22.

# 6- VPN
Hãy xét một công ty có nhiều văn phòng ở các vị trí địa lý khác nhau. Công ty này có thể kết nối tất cả các văn phòng và chi nhánh về trụ sở chính để bất kỳ thiết bị nào cũng có thể truy cập tài nguyên chia sẻ như thể đang ngồi trực tiếp tại trụ sở chính không? Cầu trả lời là có; hơn nữa, giải pháp tiết kiệm chi phí nhất là thiết lập một mạng riêng ảo (VPN - Virtual Private Network) sử dụng cơ sở hạ tầng Internet. Điểm nhấn ở đây là chữ V trong Virtual (ảo).

Khi Internet được thiết kế, bộ giao thức TCP/IP tập trung vào việc chuyển giao các gói tin. Ví dụ, nếu một router ngừng hoạt động, các giao thức định tuyến có thể thích ứng và chọn tuyến đường khác để gửi gói tin. Nếu một gói tin không nhận được xác nhận, TCP có các cơ chế tích hợp sẵn để phát hiện và gửi lại. <u>Tuy nhiên, không có cơ chế nào sẵn có để đảm bảo mọi dữ liệu ra hoặc vào máy tính đều được bảo vệ khỏi bị tiết lộ hay chỉnh sửa.</u> Một giải pháp phổ biến là thiết lập kết nối VPN. Điểm nhấn ở đây là chữ P trong Private (riêng tư).

Hầu hết các công ty đều yêu cầu trao đổi thông tin "riêng tư" trong mạng ảo của họ. Do đó, VPN mang lại giải pháp rất tiện lợi và tương đối tiết kiệm. Yêu cầu chính chỉ là kết nối Internet cùng với một VPN server và VPN client.

Sơ đồ mạng bên dưới minh họa ví dụ về một công ty có hai chi nhánh từ xa kết nối về trụ sở chính. Một **VPN client** ở các chi nhánh từ xa sẽ kết nối tới **VPN server** ở trụ sở chính. Trong trường hợp này, <u>VPN client sẽ mã hóa lưu lượng và truyền về trụ sở chính thông qua đường hầm VPN đã thiết lập </u>(hiển thị màu xanh dương). Lưu lượng VPN được giới hạn trong các đường màu xanh dương; còn các đường màu xanh lá cây sẽ truyền lưu lượng đã được giải mã.
![[Pasted image 20260811171910.png]]
Trong sơ đồ mạng tiếp theo, chúng ta thấy hai người dùng từ xa sử dụng VPN client để kết nối tới VPN server ở trụ sở chính. Lúc này, VPN client kết nối cho từng thiết bị riêng lẻ.
![[Pasted image 20260811172101.png]]
<u>Khi đường hầm VPN được thiết lập, toàn bộ lưu lượng Internet của chúng ta thường sẽ được định tuyến qua kết nối VPN, tức là qua đường hầm VPN</u>. <u>Do đó, khi chúng ta truy cập một dịch vụ Internet hay ứng dụng web, họ sẽ không thấy địa chỉ IP public của chúng ta mà sẽ thấy IP của VPN serve</u>r. Đây là lý do một số người dùng kết nối VPN để vượt qua các rào cản địa lý. Ngoài ra, nhà cung cấp dịch vụ Internet (ISP) địa phương sẽ chỉ thấy lưu lượng đã mã hóa, làm hạn chế khả năng chặn hoặc kiểm duyệt truy cập.

Nói cách khác, nếu người dùng kết nối tới một VPN server ở Nhật Bản, đối với các server họ truy cập, họ sẽ xuất hiện như thể đang ở Nhật Bản. Các server này sẽ tùy chỉnh trải nghiệm tương ứng, chẳng hạn như chuyển hướng họ sang phiên bản tiếng Nhật của dịch vụ. Ảnh chụp màn hình bên dưới hiển thị trang Google Search sau khi kết nối tới một VPN server tại Nhật Bản.
>*=> tức là toàn bộ lưu lượng internet của mình đi qua đường hầm vpn --> đi tới von server, vpn server sẽ đóng vai trò định tuyến packet cuả ta từ đây*

![[Pasted image 20260811172134.png]]
### cách 1 gói tin đi tới vpn server
>*packet của chúng ta khi di tới đường hầm vpn có cần đi qua 1 cái gateway nào ko, và có cần 1 server phụ đặt tại quốc gia của ta, để packet có thể đi tới đó và được đưa tới vpn server của các nước khác?*.
>**1. Có cần đi qua Gateway không?**
> **Có.** Sau khi ứng dụng VPN mã hóa dữ liệu, nó sẽ bọc dữ liệu đó vào một gói tin IP mới. Để gói tin này rời khỏi máy tính và ra ngoài Internet, nó bắt buộc phải gửi đến Default Gateway (chính là Router Wi-Fi hoặc modem mạng tại nhà/công ty bạn).
> 
> **2. Có cần một server phụ đặt ở trong nước không?**
> **Không cần.** Máy tính của bạn sẽ thiết lập kết nối trực tiếp tới địa chỉ IP công cộng (Public IP) của VPN server ở nước ngoài. Gói tin sẽ đi qua hạ tầng mạng và các router định tuyến quốc tế thông thường của nhà mạng (ISP) để tới VPN server, chứ không cần thêm một server trung chuyển riêng nào đặt trong nước.
> 
> _(Ngoại lệ duy nhất là khi bạn sử dụng các kỹ thuật đặc biệt như Multi-hop VPN/Double VPN, hoặc sử dụng các proxy vượt tường lửa có trạm trung chuyển)._
> 
>  Luồng di chuyển thực tế của gói tin:
> 
> Máy tính (Client) $\rightarrow$ Router/Gateway tại nhà $\rightarrow$ ISP trong nước $\rightarrow$ Router định tuyến quốc tế $\rightarrow$ VPN Server nước ngoài $\rightarrow$ Internet / Trang web đích

### vpn bảo vệ ip kiểu gì
Các router trong nước (ISP, router nhà) chỉ đóng vai trò là bên vận chuyển. Dữ liệu và IP thực sự của bạn vẫn được bảo vệ khi đi qua các router này nhờ vào 2 cơ chế: **Mã hóa (Encryption)** và **Đóng gói (Encapsulation)**.
<u>1. Bảo mật dữ liệu bằng Mã hóa</u>
Trước khi gói tin rời khỏi máy tính, phần mềm VPN sẽ mã hóa toàn bộ nội dung bên trong (URL, nội dung chat, form dữ liệu...).
* Khi đi qua các router trong nước, họ chỉ thấy một chuỗi ký tự ngẫu nhiên vô nghĩa.
* Nếu không có khóa giải mã (chỉ có ở máy bạn và VPN Server), các router không thể xem bạn đang truyền tải nội dung gì

<u>2. bảo mật địa chỉ IP bằng Đóng gói (Tunneling)</u>
VPN sử dụng kỹ thuật đóng gói gói tin thành 2 lớp IP riêng biệt:
* **Lớp vỏ bên ngoài (Chưa mã hóa):**
* Source IP: Địa chỉ IP thực của bạn.
* Destination IP: Địa chỉ IP của VPN Server.
* Router trong nước chỉ nhìn vào lớp vỏ này để biết đường chuyển gói tin sang VPN Server ở nước ngoài.

* **Ruột bên trong (Đã mã hóa hoàn toàn):**
* Destination IP thực tế: Địa chỉ IP của trang web bạn muốn truy cập.
* Toàn bộ yêu cầu dữ liệu thực sự

| Router trong nước THẤY              | Router trong nước KHÔNG THẤY             |
| ----------------------------------- | ---------------------------------------- |
| IP nhà bạn và IP của VPN Server     | Trang web thực sự bạn đang truy cập      |
| Bạn đang sử dụng VPN                | IP của trang web đích                    |
| Dung lượng dữ liệu gửi/nhận         | Nội dung tin nhắn, mật khẩu, dữ liệu web |
| Thời điểm bạn bắt đầu/ngừng kết nối | Lịch sử duyệt web của bạn                |

Nhờ cơ chế này, các router trung gian trong nước chỉ biết bạn đang trao đổi dữ liệu mã hóa với một máy chủ VPN, còn điểm đến cuối cùng và nội dung giao dịch thì hoàn toàn bị giấu kín.

# 7- Tổng kết
Trong bài học này, chúng ta đã tìm hiểu ba phương pháp chính để bảo mật lưu lượng mạng.

Phương pháp đầu tiên là sử dụng TLS, cung cấp giải pháp tiện lợi để bảo mật nhiều giao thức như HTTP, SMTP và POP3. Các giao thức được bảo mật bằng TLS thường được thêm chữ S (Secure) vào tên, chẳng hạn như HTTPS, SMTPS và POP3S.

Phương pháp thứ hai là sử dụng SSH. Mặc dù SSH chủ yếu được dùng để truy cập thiết bị từ xa, nó cũng có thể truyền file an toàn và thiết lập các đường hầm bảo mật. Tạo một đường hầm SSH là lựa chọn tối ưu nếu bạn muốn truyền lưu lượng của một giao thức dạng văn bản thuần (plaintext), chẳng hạn như VNC.

Phương pháp cuối cùng là sử dụng kết nối VPN. Kết nối VPN thường là lựa chọn hoàn hảo để kết nối hai chi nhánh của một công ty.

Chúng ta sẽ kết thúc bài học này bằng một thử thách thực hành.

### Thử thách

Nhấn nút Start Lab Machine bên dưới. Máy thực hành sẽ khởi chạy ở chế độ Split-Screen (chia đôi màn hình). Nếu không thấy, hãy sử dụng nút Show Split View màu xanh dương ở đầu trang.

Chúng tôi đã thiết lập trình duyệt ghi lại các khóa TLS của phiên làm việc để có thể phân tích kỹ hơn lưu lượng mạng bằng Wireshark. Việc ghi log này được thực hiện bằng cách thêm một tham số vào phím tắt của trình duyệt. Lệnh `chromium --ssl-key-log-file=~/ssl-key.log` sẽ xuất các khóa TLS ra file `ssl-key.log`.

File bắt gói tin có tên `randy-chromium.pcapng` được lưu trong thư mục `Documents`. Khi mở file này trong Wireshark, bạn có thể cấu hình Wireshark sử dụng file `ssl-key.log` để giải mã toàn bộ lưu lượng TLS. Bạn có thể xem 5 bước thực hiện điều này trong hai ảnh chụp màn hình bên dưới.

Đầu tiên, nhấn chuột phải vào bất kỳ gói tin TLS nào và chọn "Protocol Preferences". Từ menu phụ, chọn "Transport Layer Security". Thứ ba, nhấn vào "Open Transport Layer Security preferences".
![[Pasted image 20260811175518.png]]
Hộp thoại cấu hình sẽ xuất hiện. Bạn nhấn vào nút "Browse" (được đánh dấu số 4) để chọn file `ssl-key.log` trong thư mục `Documents`. Cuối cùng, nhấn OK để Wireshark giải mã toàn bộ lưu lượng TLS. Một trong số các gói tin này chứa thông tin đăng nhập.
![[Pasted image 20260811175528.png]]
![[Pasted image 20260811181244.png]]