# 2-OSI model
Mô hình OSI ban đầu có thể trông phức tạp, nhưng khi đi qua từng ví dụ, bạn sẽ thấy nó rất dễ hiểu.

Mô hình OSI (Open Systems Interconnection) do ISO phát triển nhằm định nghĩa khuôn mẫu giao tiếp trong mạng máy tính. Dù mang tính lý thuyết, đây là nền tảng quan trọng để hiểu sâu về networking. Mô hình gồm 7 tầng (đánh số từ 1 đến 7, từ dưới lên):

1. Physical Layer
2. Data Link Layer
3. Network Layer
4. Transport Layer
5. Session Layer
6. Presentation Layer
7. Application Layer

Mẹo nhớ thứ tự từ Layer 1 đến Layer 7: "Please Do Not Throw Spinach Pizza Away". Việc nhớ tên kèm số hiệu tầng rất quan trọng để hiểu các thuật ngữ như "Layer 3 switch" hay "Layer 7 firewall".

---

### Layer 1: Physical Layer

Xử lý <u>kết nối vật lý giữa các thiết bị, bao gồm môi trường truyền dẫn</u> (dây cáp, sóng) và quy định các bit nhị phân 0, 1. <u>Tín hiệu truyền có thể là điện, quang hoặc không dây</u> (như cáp Ethernet, cáp quang, băng tần WiFi 2.4 GHz, 5 GHz, 6 GHz).

### Layer 2: Data Link Layer

Cho phép truyền<u> dữ liệu giữa các node trong cùng một network segment </u>(nhóm thiết bị dùng chung môi trường truyền dẫn, ví dụ: 10 máy tính nối chung một switch).

* Ví dụ: Ethernet (802.3), WiFi (802.11).
* Sử dụng địa chỉ MAC (6 byte, dạng hexadecimal).
* Trong mỗi frame thực tế sẽ chứa Destination MAC address và Source MAC address.

### Layer 3: Network Layer

Đảm nhận việc<u> gửi dữ liệu giữa các mạng khác nhau thông qua logical addressing và routing</u> (tìm đường đi tối ưu cho packet). Nếu Layer 2 nối các máy trong 1 văn phòng, thì Layer 3 nối nhiều văn phòng ở các thành phố khác nhau lại với nhau.

* Ví dụ: IP, ICMP, IPSec, SSL VPN.

### Layer 4: Transport Layer

Cho phép giao tiếp end-to-end giữa các ứng dụng đang chạy trên các host khác nhau. Hỗ trợ các tính năng như flow control, error correction.

* Ví dụ: TCP, UDP.

### Layer 5: Session Layer

Chịu trách nhiệm thiết lập, duy trì và đồng bộ hóa phiên giao tiếp (session) giữa các ứng dụng. Đồng bộ hóa giúp dữ liệu truyền đúng thứ tự và có khả năng khôi phục khi gặp lỗi truyền dẫn.

* Ví dụ: NFS, RPC.

### Layer 6: Presentation Layer

Đảm bảo dữ liệu được định dạng sao cho Application Layer có thể hiểu được. Xử lý data encoding (ASCII, Unicode), compression và encryption.

* Ví dụ: MIME (mã hóa file đính kèm email), các định dạng ảnh/video (JPEG, PNG).

### Layer 7: Application Layer

Cung cấp dịch vụ mạng trực tiếp cho các ứng dụng của end-user (ví dụ: browser dùng HTTP/HTTPS để tải trang, gửi form).

* Ví dụ: HTTP, HTTPS, FTP, SMTP, DNS, DHCP.
### Bảng tổng hợp
![[Pasted image 20260809135109.png]]

| Số tầng | Tên tầng           | Chức năng chính                             | Ví dụ                           |
| ------- | ------------------ | ------------------------------------------- | ------------------------------- |
| Layer 7 | Application layer  | Cung cấp giao diện và dịch vụ cho ứng dụng  | HTTP, HTTPS, FTP, DNS           |
| Layer 6 | Presentation layer | Encoding, mã hóa và nén dữ liệu             | Unicode, ASCII, JPEG, PNG       |
| Layer 5 | Session layer      | Thiết lập, duy trì và đồng bộ session       | NFS, RPC                        |
| Layer 4 | Transport layer    | Giao tiếp end-to-end và truyền dữ liệu      | TCP, UDP                        |
| Layer 3 | Network layer      | Định địa chỉ logic và routing giữa các mạng | IP, ICMP, IPSec                 |
| Layer 2 | Data Link layer    | Truyền dữ liệu giữa các node liền kề        | Ethernet (802.3), WiFi (802.11) |
| Layer 1 | Physical layer     | Môi trường truyền dẫn vật lý                | Tín hiệu điện, quang, không dây |
*=> nhìn vào phần ví dụ là mình có thể hiêủ và nhớ được*
# 3-tcp/ip model
Sau khi tìm hiểu mô hình lý thuyết ISO OSI, chúng ta chuyển sang mô hình được triển khai thực tế: TCP/IP.

TCP/IP (Transmission Control Protocol/Internet Protocol) được Bộ Quốc phòng Mỹ (DoD) phát triển từ những năm 1970. Điểm mạnh của mô hình này là giữ cho mạng tiếp tục hoạt động ngay cả khi một phần hạ tầng bị gián đoạn (như trong một cuộc tấn công quân sự) nhờ khả năng tự điều chỉnh tuyến đường của các giao thức routing khi topology thay đổi.

Nhìn mô hình TCP/IP theo thứ tự từ trên xuống dưới:

* Application Layer: Gộp cả 3 tầng Session, Presentation và Application (Layer 5, 6, 7) của mô hình OSI. *=>quản lý phiên + mã hóa+ giao thức truyền dữ liệu*
* Transport Layer: Tương đương Layer 4 của OSI.
* Internet Layer: Tương đương Network Layer (Layer 3) của OSI.
* Link Layer: Tương đương Data Link Layer (Layer 2) của OSI.
![[Pasted image 20260809135121.png]]
>*tầng 4 và 7 khác nhau điểm nào, khi bọn chúng đều giúp giao tiếp giữa host với host??*
>Tầng 4 (Transport): Nối 2 host ở mức hạ tầng mạng và cổng (port) để đảm bảo dữ liệu được đóng gói và chuyển đi đúng nơi.
>
>Tầng 7 (Application): Nối 2 phần mềm thực tế với nhau. Đây là nơi người dùng trực tiếp tương tác (ví dụ: trình duyệt gửi yêu cầu tải trang web và server trả về nội dung HTML). 
### Bảng đối chiếu giữa TCP/IP và ISO OSI

| Số tầng OSI | Mô hình ISO OSI    | Mô hình TCP/IP    | Giao thức / Tiêu chuẩn                      |
| ----------- | ------------------ | ----------------- | ------------------------------------------- |
| 7           | Application Layer  | Application Layer | HTTP, HTTPS, FTP, SMTP, DNS, Telnet, SSH... |
| 6           | Presentation Layer |                   |                                             |
| 5           | Session Layer      |                   |                                             |
| 4           | Transport Layer    | Transport Layer   | TCP, UDP                                    |
| 3           | Network Layer      | Internet Layer    | IP, ICMP, IPSec                             |
| 2           | Data Link Layer    | Link Layer        | Ethernet (802.3), WiFi (802.11)             |
| 1           | Physical Layer     |                   |                                             |

Nhiều giáo trình hiện đại (như *Computer Networking: A Top-Down Approach* của Kurose & Ross) mô tả TCP/IP gồm 5 tầng bằng cách tách riêng Physical Layer:

1. Application
2. Transport
3. Network
4. Link
5. Physical

# 4-Ip adress and subnet
Khi nghe đến thuật ngữ IP address (địa chỉ IP), bạn thường nghĩ đến những chuỗi số quen thuộc như 192.168.0.1 hoặc một địa chỉ ít gặp hơn như 172.16.159.243. Cả hai trường hợp này đều chính xác. Chúng đều là các địa chỉ IP, cụ thể hơn là phiên bản IPv4 (IP version 4).

Mọi host trên mạng máy tính đều cần một định danh duy nhất để các host khác có thể thiết lập giao tiếp. Nếu không có định danh duy nhất này, hệ thống không thể xác định chính xác thiết bị cần truyền nhận dữ liệu mà không xảy ra xung đột. Trong bộ giao thức TCP/IP, việc gán địa chỉ IP cho từng thiết bị kết nối vào mạng là yêu cầu bắt buộc.

Một hình ảnh so sánh thực tế của IP address là địa chỉ nhà bưu chính. Địa chỉ nhà cho phép bạn nhận thư từ, bưu kiện từ khắp nơi trên thế giới và giúp bưu điện xác định chính xác ngôi nhà của bạn mà không nhầm lẫn với bất kỳ đâu khác.

Hiện nay hệ thống mạng sử dụng hai phiên bản chính là IPv4 và IPv6 (IP version 6). IPv4 vẫn là chuẩn phổ biến nhất trong hạ tầng mạng hiện tại. Khi một tài liệu đề cập đến "IP" mà không chỉ rõ phiên bản, mặc định điều đó đề cập đến IPv4.

### Cấu trúc chi tiết của một địa chỉ IPv4
![[Pasted image 20260809131402.png|587]]
Một địa chỉ IPv4 bao gồm 32 bit nhị phân, được chia làm 4 nhóm gọi là octet (mỗi octet gồm 8 bit).

Do mỗi *octet chứa 8 bit*, giá trị biểu diễn dưới dạng số thập phân của một octet nằm trong khoảng từ 0 đến 255 (tương ứng từ $00000000_2$ đến $11111111_2$).

Trong một dải mạng con (subnet), hai giá trị 0 và 255 ở octet cuối cùng thường được bảo lưu cho các chức năng đặc biệt của hệ thống:

* *Network address* (Địa chỉ mạng): Giá trị 0 ở phần host đại diện cho toàn bộ dải mạng (ví dụ: 192.168.1.0).
* *Broadcast address* (Địa chỉ quảng bá): Giá trị 255 đại diện cho địa chỉ quảng bá (ví dụ: 192.168.1.255). Khi <u>gói tin được gửi tới địa chỉ broadcast, tất cả các host nằm trong dải mạng đó đều sẽ nhận được dữ liệu.</u>

Về mặt lý thuyết toán học, với không gian 32 bit, tổng số địa chỉ IPv4 duy nhất có thể tạo ra là khoảng $2^{32} \approx 4,29$ tỷ địa chỉ. Con số thực tế khả dụng cho thiết bị cuối ít hơn mức này do phải loại trừ các dải địa chỉ bảo lưu cho Network address, Broadcast address, Loopback ($127.0.0.0/8$), Multicast ($224.0.0.0/4$) và các dải dành riêng cho kiểm thử.

---

### Tra cứu cấu hình mạng trên hệ điều hành

Trên hệ điều hành MS Windows, bạn có thể kiểm tra cấu hình IP bằng lệnh `ipconfig` trong Command Prompt hoặc PowerShell.

Trên các hệ thống Linux và UNIX, lệnh truyền thống là `ifconfig` hoặc lệnh hiện đại hơn là `ip address show` (có thể viết tắt là `ip a s`
Đảm nhận việc gửi dữ liệu giữa các mạng kh
Đảm nhận việc gửi dữ liệu giữa các mạng kh
Đảm nhận việc gửi dữ liệu giữa các mạng kh
Đảm nhận việc gửi dữ liệu giữa các mạng kh).

Ví dụ kết quả khi chạy `ifconfig` trên Linux:

```text
wlo1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.66.89  netmask 255.255.255.0  broadcast 192.168.66.255
        inet6 fe80::73e1:ca5e:3f93:b1b3  prefixlen 64  scopeid 0x20<link>
        ether cc:5e:f8:02:21:a7  txqueuelen 1000  (Ethernet)
        RX packets 19684680  bytes 18865072842 (17.5 GiB)
        RX errors 0  dropped 364  overruns 0  frame 0
        TX packets 14439678  bytes 8773200951 (8.1 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

Thông số hiển thị từ output trên bao gồm:

* Địa chỉ IP của host trên card mạng không dây `wlo1`: 192.168.66.89 (*inet* là ipv4, *inet6* là ipv6)
* Subnet mask (mặt nạ mạng con): 255.255.255.0
* Broadcast address: 192.168.66.255
* Địa chỉ MAC (Hardware address): cc:5e:f8:02:21:a7

So sánh với output khi sử dụng lệnh `ip a s`:

```text
4: wlo1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether cc:5e:f8:02:21:a7 brd ff:ff:ff:ff:ff:ff
    altname wlp3s0
    inet 192.168.66.89/24 brd 192.168.66.255 scope global dynamic noprefixroute wlo1
       valid_lft 36795sec preferred_lft 36795sec
    inet6 fe80::73e1:ca5e:3f93:b1b3/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever

```

Lệnh `ip a s` hiển thị địa chỉ IP dưới dạng ký hiệu CIDR: `192.168.66.89/24`.

Ký hiệu `/24` tương đương với *subnet mask* `255.255.255.0`. Con số 24 chỉ ra rằng 24 bit đầu tiên (tương ứng với 3 octet bên trái: `192.168.66`) được giữ cố định để định danh cho subnet. Tám bit còn lại dành cho phần host, cho phép gán địa chỉ cho các thiết bị trong khoảng từ `192.168.66.1` đến `192.168.66.254` (với `192.168.66.0` là Network address và `192.168.66.255` là Broadcast address).
*=>octet cuối để gán cho các thiết bị trong mạng, còn 3 octet đầu giữ cố định (để định danh cho các thiết bị trong network đó)*


### Phân loại IP Public và IP Private (RFC 1918)

Trong thực tế quản trị mạng, địa chỉ IP được chia làm hai loại chính:

* *Public IP address* (Địa chỉ IP công cộng): Được định tuyến trên toàn mạng Internet toàn cầu, do IANA và các tổ chức quản lý Internet cấp phát.
* *Private IP address* (Địa chỉ IP nội bộ): Được sử dụng trong các mạng LAN nội bộ và không thể định tuyến trực tiếp trên mạng Internet toàn cầu.

Theo tiêu chuẩn RFC 1918, *ba dải địa chỉ IP Private được quy định bao gồm:*

1. Class A: $10.0.0.0 - 10.255.255.255$ (Ký hiệu CIDR: $10.0.0.0/8$)
2. Class B: $172.16.0.0 - 172.31.255.255$ (Ký hiệu CIDR: $172.16.0.0/12$)
3. Class C: $192.168.0.0 - 192.168.255.255$ (Ký hiệu CIDR: $192.168.0.0/16$)

Mạng Private hoạt động tương tự như một khu đô thị biệt lập có hệ thống đánh số nhà nội bộ. Các ngôi nhà bên trong có thể dễ dàng trao đổi thư từ với nhau, nhưng không thể trực tiếp gửi thư ra thế giới bên ngoài nếu không qua bưu cục trung tâm.

*Để một thiết bị mang IP Private có thể truy cập Internet, router biên phải sở hữu một Public IP và thực hiện cơ chế NAT* (Network Address Translation) để biên dịch địa chỉ IP nội bộ thành địa chỉ IP công cộng khi gửi gói tin ra ngoài.

Việc ghi nhớ các dải IP Private giúp ích rất nhiều trong công việc thực tế, giúp bạn nhận biết ngay một địa chỉ như `10.1.33.7` hay `172.31.33.7` là IP nội bộ và không thể truy cập trực tiếp từ mạng Internet công cộng nếu không có kết nối VPN hoặc cấu hình chuyển tiếp cổng (port forwarding).


### Cơ chế Routing (Định tuyến)

Router đóng vai trò tương tự như một bưu cục trung chuyển. Khi bạn gửi một bưu kiện tới địa chỉ ở thành phố hoặc quốc gia khác, bưu cục cục bộ sẽ kiểm tra địa chỉ đích và quyết định tuyến đường tối ưu để chuyển tiếp bưu kiện đó.

Về mặt kỹ thuật, router hoạt động ở Layer 3 (Network Layer) trong mô hình OSI/TCP-IP. Bằng cách kiểm tra địa chỉ IP đích (Destination IP) trong phần header của gói tin (packet) và tra cứu bảng định tuyến (routing table), router xác định chặng tiếp theo (next-hop) để chuyển tiếp gói tin, giúp gói tin di chuyển qua nhiều mạng trung gian trước khi tới đúng đích.

# 5-UDP và TCP
Giao thức IP cho phép chúng ta kết nối tới host đích trên mạng; host được định danh bằng địa chỉ IP của nó. Chúng ta cần các giao thức cho phép các tiến trình trên các host giao tiếp với nhau. Có hai giao thức transport để đạt được điều đó: TCP và UDP
>*host ở đây là 1 node trong network như điện thoại, máy tính , vvv*.

### UDP
UDP (User Datagram Protocol) cho phép chúng ta kết nối tới một tiến trình cụ thể trên host đích. UDP là một giao thức connectionless đơn giản hoạt động ở transport layer (layer 4). <u>Connectionless nghĩa là nó không cần thiết lập kết nối. UDP thậm chí không cung cấp cơ chế để biết gói tin đã được giao hay chưa</u>.

Địa chỉ IP định danh host; chúng ta cần một cơ chế để xác định tiến trình gửi và nhận. Điều này đạt được bằng cách sử dụng các *port number*. Một port number sử dụng hai octet, do đó nằm trong khoảng từ 1 đến 65535; port 0 được bảo lưu. (Số 65535 được tính bằng biểu thức $2^{16} - 1$).

Một ví dụ thực tế tương tự UDP là dịch vụ thư tiêu chuẩn không có xác nhận giao hàng. Nói cách khác, không có gì đảm bảo gói tin UDP đã được nhận thành công, giống như việc gửi bưu kiện qua thư tiêu chuẩn không có xác nhận. Với thư tiêu chuẩn, chi phí rẻ hơn so với các tùy chọn giao thư có xác nhận. Với UDP, nó mang lại tốc độ tốt hơn so với một giao thức transport cung cấp xác nhận.

Nhưng nếu chúng ta muốn một giao thức transport xác nhận các gói tin đã nhận thì sao? Câu trả lời nằm ở việc sử dụng TCP thay vì UDP.

### TCP
TCP (Transmission Control Protocol) <u>là một giao thức transport connection-oriented</u>. Nó sử dụng nhiều cơ chế khác nhau để đảm bảo truyền dữ liệu tin cậy gửi bởi các tiến trình khác nhau trên các host. Giống như UDP, nó là giao thức layer 4. Là connection-oriented, nó yêu cầu thiết lập kết nối TCP trước khi có thể gửi bất kỳ dữ liệu nào.

*Trong TCP, mỗi octet dữ liệu có một sequence number;* điều này giúp bên nhận dễ dàng xác định các gói tin bị mất hoặc bị trùng lặp. Phía nhận xác nhận việc nhận dữ liệu bằng một acknowledgement number chỉ định octet cuối cùng nhận được.
>*tcp nó cần sự tin cậy, nên nó cần thiết lập kết nối=> do đó cần có cơ chế xác minh và thiết lập kết nối ==> đó chính là 3 way handsake*

*Kết nối TCP được thiết lập bằng quá trình three-way handshake.* Hai flag được sử dụng: SYN (Synchronise) và ACK (Acknowledgment). Các gói tin được gửi như sau:

* SYN Packet: Client khởi tạo kết nối bằng cách gửi gói tin SYN tới server. Gói tin này chứa initial sequence number được chọn ngẫu nhiên của client.
* SYN-ACK Packet: Server phản hồi gói tin SYN bằng gói tin SYN-ACK, bổ sung initial sequence number được chọn ngẫu nhiên bởi server.
* ACK Packet: Quá trình three-way handshake hoàn tất khi client gửi gói tin ACK để xác nhận việc nhận gói tin SYN-ACK.
![[Pasted image 20260809134547.png]]
*Tương tự UDP, TCP xác định tiến trình khởi tạo hoặc đang chờ (listening) kết nối bằng cách sử dụng các port number*. Như đã đề cập, một port number hợp lệ nằm trong khoảng từ 1 đến 65535 vì nó sử dụng hai octet và port 0 được bảo lưu.

# 6-encapsulation
### (Quan trọng)Vòng đời của một gói tin (The Life of a Packet)-

Dựa trên những gì chúng ta đã học cho đến nay, chúng ta có thể giải thích một phiên bản đơn giản hóa về vòng đời của gói tin. Hãy xem xét kịch bản khi bạn tìm kiếm một phòng trên TryHackMe.

* Trên trang tìm kiếm của TryHackMe, bạn nhập truy vấn tìm kiếm và nhấn Enter.
* Trình duyệt web của bạn, sử dụng HTTPS, chuẩn bị một HTTP request và đẩy nó xuống tầng bên dưới nó, tức là tầng transport.
* Tầng transport cần thiết lập kết nối thông qua quá trình bắt tay ba bước (three-way handshake) giữa trình duyệt của bạn và web server của TryHackMe. Sau khi thiết lập kết nối TCP, nó có thể gửi HTTP request chứa truy vấn tìm kiếm. Mỗi đoạn TCP được tạo ra sẽ được gửi xuống tầng bên dưới nó, tức là tầng Internet.
* Tầng IP thêm địa chỉ IP nguồn (tức là máy tính của bạn) và địa chỉ IP đích (tức là địa chỉ IP của web server TryHackMe). Để gói tin này đến được router, laptop của bạn chuyển nó xuống tầng bên dưới nó, tức là tầng link.
* Tùy thuộc vào giao thức, tầng link thêm header và trailer của tầng link thích hợp, và gói tin được gửi đến router.
* Router loại bỏ header và trailer của tầng link, kiểm tra địa chỉ IP đích cùng với các trường khác, và định tuyến gói tin đến liên kết phù hợp. Mỗi router lặp lại quá trình này cho đến khi nó đến được router của server đích.
* Các bước sau đó sẽ được đảo ngược khi gói tin đến router của mạng đích. Khi chúng ta tìm hiểu thêm các giao thức bổ sung, chúng ta sẽ quay lại bài tập này và tạo ra một phiên bản chi tiết hơn.

### encapsulation
Trước khi kết thúc, điều quan trọng là phải giải thích một khái niệm cốt lõi khác: encapsulation (đóng gói). Trong bối cảnh này, encapsulation đề cập đến quá trình mỗi tầng thêm một header (và đôi khi là một trailer) vào đơn vị dữ liệu nhận được và gửi đơn vị dữ liệu "đã đóng gói" đó xuống tầng bên dưới.

Encapsulation là một khái niệm thiết yếu vì nó cho phép mỗi tầng tập trung vào chức năng dự kiến của mình. Trong hình ảnh dưới đây, chúng ta có bốn bước sau:
![[Pasted image 20260809140016.png]]
* **Application data:** Mọi thứ bắt đầu khi người dùng nhập dữ liệu họ muốn gửi vào ứng dụng. Ví dụ: bạn viết một email hoặc một tin nhắn nhanh và nhấn nút gửi. Ứng dụng định dạng dữ liệu này và bắt đầu gửi nó theo giao thức ứng dụng được sử dụng, thông qua tầng bên dưới nó, tức là tầng transport.
* **Transport protocol segment or datagram:** Tầng transport, chẳng hạn như TCP hoặc UDP, thêm thông tin header thích hợp và tạo ra đoạn (segment) TCP (hoặc datagram UDP). Đoạn này được gửi xuống tầng bên dưới nó, tức là tầng network.
* **Network packet:** Tầng network, hay còn gọi là Internet layer, thêm một IP header vào đoạn TCP hoặc datagram UDP nhận được. Sau đó, gói tin IP này được gửi xuống tầng bên dưới nó, tức là tầng data link.
* **Data link frame:** Ethernet hoặc WiFi nhận gói tin IP và thêm header cùng trailer thích hợp, tạo thành một frame.

> Chúng ta bắt đầu với application data. Ở tầng transport, chúng ta thêm header TCP hoặc UDP để tạo thành đoạn TCP hoặc datagram UDP. Tiếp tục, ở tầng network, chúng ta thêm IP header thích hợp để có được một gói tin IP có thể định tuyến qua Internet. Cuối cùng, chúng ta thêm header và trailer phù hợp để có được một frame WiFi hoặc Ethernet ở tầng link.

Quá trình này phải được đảo ngược ở phía nhận cho đến khi application data được trích xuất.

