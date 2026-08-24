# 1-intro
Have you ever wondered how your computer can dynamically configure its network settings when you turn it on or connect it to a new network? Have you ever wanted to know how many devices and countries your packets passed through before reaching their destination? <u>Are you curious how all your home devices can access the Internet even though your ISP gives you a single IP address</u>?

If you want to know the answers to these questions, among others, then this room is for you.

This room is the second room in a series of four rooms about computer networking:

- [Networking Concepts](https://tryhackme.com/r/room/networkingconcepts)
- Networking Essentials (this room)
- [Networking Core Protocols](https://tryhackme.com/r/room/networkingcoreprotocols)
- [Networking Secure Protocols](https://tryhackme.com/r/room/networkingsecureprotocols)

### Learning Prerequisites

To benefit from this room, we recommend that you know the following:

- ISO OSI model and layers
- /IP model and layers
- Ethernet, IP, and protocols

In other words, starting this room after [Networking Concepts](https://tryhackme.com/r/room/networkingconcepts) is the recommended approach.

### Learning Objectives

The objective of this room is to teach you about various standard protocols and technologies that glue things together:

- Dynamic Host Configuration Protocol ()
- Address Resolution Protocol ()
- Network Address Translation (NAT)
- Internet Control Message Protocol (ICMP)
    - Ping
    - Traceroute

# 2- DHCP
Bạn đến quán cà phê yêu thích, gọi một món đồ uống nóng và mở laptop. Máy tính tự động kết nối WiFi của quán và cấu hình mạng, giúp bạn có thể bắt đầu làm một room TryHackMe mới ngay lập tức. Bạn không cần gõ bất kỳ địa chỉ IP nào nhưng mọi thứ đã sẵn sàng. Hãy cùng xem điều này diễn ra như thế nào.

-<u>Mỗi khi muốn truy cập vào một mạng, tối thiểu chúng ta cần cấu hình các thông số sau:</u>
* Địa chỉ IP kèm subnet mask
* Router (hoặc gateway)
* DNS server
### giải thích dễ hiểu dhcp
Tóm gọn lại, DHCP là cơ chế tự động "xin và cấp" địa chỉ IP khi thiết bị của bạn kết nối vào mạng.

Quy trình 4 bước (DORA) hoạt động giống như một cuộc đối thoại:

1. Discover (Khám phá)
	Máy tính vừa bật WiFi, chưa có IP nên hô to cho cả mạng nghe: "Ở đây có DHCP Server nào không? Cho tôi xin một địa chỉ IP!"
2. Offer (Đề nghị)
	<u>Router (đóng vai DHCP Server)</u> nghe thấy liền trả lời: "Tôi đây! Mạng còn trống địa chỉ 192.168.1.100, bạn lấy dùng nhé?"
3. Request (Yêu cầu)
	Máy tính đáp lại: "Được luôn! Tôi chốt lấy đúng địa chỉ 192.168.1.100 này nhé."
4. Acknowledge (Xác nhận)
	Router chốt kèo: "Xác nhận địa chỉ 192.168.1.100 chính thức cấp cho bạn! Tôi gửi kèm luôn địa chỉ Router (Gateway) để đi ra ngoài mạng và địa chỉ DNS Server để bạn gõ tên miền lướt web."

S<u>au 4 bước này, thiết bị của bạn có đủ 3 thông số bắt buộc để ra được Internet</u>:
* IP của máy: Tên gọi riêng của bạn trong mạng LAN.
* Gateway: Địa chỉ của Router (cửa ngõ đẩy gói tin ra Internet).
* DNS Server: Máy chủ dịch tên miền (như google.com) thành IP.
# 3-ARP
### giải thích dễ hiểu ARP
<u> 1. Vấn đề nằm ở đâu?</u>
Trong mạng nội bộ (mạng LAN, WiFi):
* **IP** giống như *Tên gọi* của thiết bị.
* **MAC** giống như *Địa chỉ nhà / Khuôn mặt* thực tế của thiết bị đó.

Khi máy A muốn gửi dữ liệu cho máy B, máy A đã biết IP của máy B. Nhưng ở tầng phần cứng bên dưới, các thiết bị mạng chỉ có thể chuyển gói tin đi nếu biết chính xác địa chỉ MAC đích.

Làm sao để từ IP tìm ra địa chỉ MAC? Đó chính là nhiệm vụ duy nhất của ARP.

<u> 2. Ví dụ đời thực</u>
Hãy tưởng tượng một phòng họp:
* Thầy giáo (máy A) biết tên học sinh cần tìm là Nam (IP: 192.168.1.5).
* Nhưng thầy không biết Nam là ai trong phòng (chưa biết MAC).

	-  Thầy đứng lên **hô to** cho cả phòng cùng nghe: *"Cho thầy hỏi ai là Nam (192.168.1.5)?"* -> Đây là **ARP Request** (gửi broadcast đến toàn mạng).
	- Cả phòng nghe thấy, nhưng chỉ có bạn Nam đứng lên **nói riêng** với thầy: *"Em là Nam đây, em mặc áo xanh (MAC: 44:DF:65:D8:FE:6C)!"* -> Đây là **ARP Reply** (gửi unicast trực tiếp cho thầy).

<u>3. Máy tính làm gì sau khi hỏi xong?</u>
Sau khi nhận được câu trả lời, máy A sẽ lưu cặp thông tin `192.168.1.5 <-> 44:DF:65:D8:FE:6C` vào một danh sách tạm thời gọi là **ARP Cache**.

Nhờ vậy, trong các lần gửi dữ liệu tiếp theo, máy A chỉ việc mở danh sách này ra xem mà không cần phải đứng lên "hô to" hỏi lại nữa.
<u>**Tóm lại 1 câu**</u>
ARP là cơ chế: **Cho biết địa chỉ IP -> Tìm ra địa chỉ MAC tương ứng.**

### chi tiết về ARP
Chúng ta đã đề cập trong room Networking Concepts rằng khi hai host giao tiếp qua mạng, một IP packet sẽ được đóng gói (encapsulated) bên trong một data link frame khi truyền ở layer 2. Hai lớp data link phổ biến nhất là Ethernet (IEEE 802.3) và WiFi (IEEE 802.11). Bất cứ khi nào một host cần giao tiếp với một host khác trong cùng mạng Ethernet hoặc WiFi, nó phải gửi IP packet bên trong một data link layer frame. Mặc dù đã biết địa chỉ IP của host đích, nó vẫn cần tra cứu địa chỉ MAC tương ứng để tạo ra header data link phù hợp.

Tuy nhiên, các thiết bị trong cùng mạng Ethernet không cần phải liên tục lưu giữ địa chỉ MAC của nhau; chúng chỉ cần biết địa chỉ MAC tại thời điểm giao tiếp. Mọi hoạt động khởi đầu đều xoay quanh địa chỉ IP. Xét kịch bản: Bạn kết nối thiết bị vào mạng, nếu mạng có DHCP server, thiết bị sẽ tự động nhận cấu hình gateway (router) và DNS server. Nhờ đó, thiết bị biết địa chỉ IP của DNS server để phân giải tên miền, đồng thời biết địa chỉ IP của router để gửi packet ra Internet. Trong toàn bộ kịch bản này, chưa có địa chỉ MAC nào được trao đổi. Nhưng về mặt kỹ thuật, hai thiết bị trên cùng một mạng Ethernet không thể truyền dữ liệu cho nhau nếu không biết địa chỉ MAC của đối phương.

Nhắc lại một chút,<u> header của một Ethernet frame bao gồm:</u>
* Destination MAC address (Địa chỉ MAC đích)
* Source MAC address (Địa chỉ MAC nguồn)
* Type (Loại giao thức, ở đây là IPv4)
>*đ hiểu gì cả, nhưng mà như này*:*
>   -khi các host trong cùng 1 network giao tiếp vs nhau (layer2) thì nó cần biết địa chỉ MAC
>    -wifi và ethernet nằm trong layer 2
>    -ARP giúp biết được MAC của mỗi host ứng với IP , sau đó lưu lại trong ARP cache

Giao thức ARP (Address Resolution Protocol) giúp tìm địa chỉ MAC của một thiết bị khác trong mạng Ethernet. Trong ví dụ dưới đây, host có IP `192.168.66.89` muốn giao tiếp với host `192.168.66.1`. Nó gửi một gói tin ARP Request để hỏi địa chỉ MAC của `192.168.66.1`. <u>Yêu cầu này được gửi từ địa chỉ MAC của bên hỏi tới địa chỉ MAC broadcast `ff:ff:ff:ff:ff:ff`</u> (như trong gói tin đầu tiên). Ngay sau đó, gói ARP Reply trả về từ host `192.168.66.1` chứa địa chỉ MAC của nó. Từ thời điểm này, hai host đã có thể trao đổi các data link layer frame.
![[Pasted image 20260811080244.png]]
Nếu sử dụng tcpdump, các gói tin sẽ hiển thị dưới dạng thuật ngữ ARP Request và ARP Reply:
![[Pasted image 20260811080039.png]]
Gói tin **ARP Request** hay **ARP Reply** không được đóng gói bên trong TCP, UDP hay IP packet; <u>nó được đóng gói trực tiếp bên trong một Ethernet frame</u>.
![[Pasted image 20260811080603.png]]
<u>ARP thường được xếp vào layer 2 vì nó làm việc trực tiếp với địa chỉ MAC</u>. Tuy nhiên, cũng có quan điểm xếp nó vào layer 3 vì nó hỗ trợ cho các thao tác của IP. Điều quan trọng nhất cần nắm là ARP đóng vai trò cầu nối giúp biên dịch (translation) từ địa chỉ layer 3 sang địa chỉ layer 2.

### chi tiết hơn về dhcp
 DHCP là giao thức tầng Application chạy trên nền UDP, trong đó DHCP server listen ở UDP port 67 và client gửi gói tin từ UDP port 68. Điện thoại và laptop của bạn mặc định đều dùng DHCP.
*Dhcp giúp tự động cấu hình ip, gate way, server cho 1 thiết bị, khi nó muốn join 1 network*
Quy trình DHCP trải qua 4 bước DORA: Discover, Offer, Request và Acknowledge:
![[Pasted image 20260811072706.png]]
* Discover: Client broadcast một message DHCPDISCOVER để tìm DHCP server trong mạng local.
* Offer: Server phản hồi lại message DHCPOFFER kèm một địa chỉ IP khả dụng cho client.
* Request: Client gửi message DHCPREQUEST để thông báo chấp nhận địa chỉ IP được cấp.
* Acknowledge: Server phản hồi message DHCPACK để xác nhận địa chỉ IP đó chính thức được gán cho client.

File pcap bên dưới ghi lại 4 bước DORA trên. Trong ví dụ này, client nhận địa chỉ 192.168.66.133:
![[Pasted image 20260811073126.png]]
Trong quá trình trao đổi gói tin DHCP, có thể thấy:

* Client ban đầu chưa có cấu hình IP, chỉ có địa chỉ MAC. Ở gói tin 1 và 3 (DHCP Discover và DHCP Request), client chưa có IP và chưa áp dụng địa chỉ IP do server đề xuất, nên nó gửi gói tin từ IP 0.0.0.0 tới địa chỉ IP broadcast 255.255.255.255.
* Tại tầng link layer, ở gói 1 và 3, client gửi tới địa chỉ MAC broadcast ff:ff:ff:ff:ff:ff. DHCP server cấp IP kèm thông số mạng trong gói DHCP Offer bằng cách gửi tới MAC destination của client (và sử dụng IP tạm thời được đề xuất trong hệ thống ví dụ này).


# 4-ICMP
Internet Control Message Protocol (ICMP) chủ yếu được sử dụng cho chẩn đoán mạng và báo lỗi. Hai lệnh phổ biến phụ thuộc vào ICMP, và chúng rất quan trọng trong việc xử lý sự cố mạng và an ninh mạng. Các lệnh đó là:

* **ping:** Lệnh này sử dụng ICMP để kiểm tra kết nối tới hệ thống mục tiêu và đo thời gian phản hồi vòng (RTT - round-trip time). Nói cách khác, nó có thể được sử dụng để biết rằng mục tiêu đang hoạt động và phản hồi của nó có thể tới được hệ thống của chúng ta.
* **traceroute**: Lệnh này được gọi là traceroute trên các hệ thống Linux và UNIX-like, và là tracert trên các hệ thống MS Windows. Nó sử dụng ICMP để khám phá tuyến đường từ host của bạn tới mục tiêu.
>*=>* 2 cái này mình học rồi
### Ping

Bạn có thể chưa từng chơi bóng bàn (ping-pong) bao giờ; tuy nhiên, nhờ có ICMP, giờ đây bạn có thể chơi nó với máy tính! Lệnh ping gửi một ICMP Echo Request (ICMP Type 8). Ảnh màn hình bên dưới hiển thị thông điệp ICMP bên trong một IP packet.
![[Pasted image 20260811094031.png]]
Máy tính ở phía nhận sẽ phản hồi bằng một ICMP Echo Reply (ICMP Type 0).

Có nhiều nguyên nhân khiến chúng ta không nhận được phản hồi. Bên cạnh khả năng hệ thống mục tiêu đang ngoại tuyến hoặc đã tắt máy, một firewall trên đường truyền cũng có thể chặn các packet cần thiết để ping hoạt động. Trong ví dụ bên dưới, chúng ta sử dụng `-c 4` để yêu cầu lệnh ping dừng lại sau khi gửi 4 packet.

```text
user@TryHackMe$ ping 192.168.11.1 -c 4PING 192.168.11.1 (192.168.11.1) 56(84) bytes of data.
64 bytes from 192.168.11.1: icmp_seq=1 ttl=63 time=11.2 ms
64 bytes from 192.168.11.1: icmp_seq=2 ttl=63 time=3.81 ms
64 bytes from 192.168.11.1: icmp_seq=3 ttl=63 time=3.99 ms
64 bytes from 192.168.11.1: icmp_seq=4 ttl=63 time=23.4 ms

--- 192.168.11.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3003ms
rtt min/avg/max/mdev = 3.805/10.596/23.366/7.956 ms

```

Output cho thấy không có packet loss; hơn nữa, nó tính toán thời gian phản hồi vòng (RTT) nhỏ nhất, trung bình, lớn nhất và độ lệch chuẩn (mdev).

### Traceroute

Làm thế nào để mọi router giữa hệ thống của chúng ta và hệ thống mục tiêu lộ diện?

Giao thức Internet có một trường gọi là **Time-to-Live (TTL)** chỉ định số lượng router tối đa mà gói tin có thể đi qua trước khi bị hủy. Router sẽ giảm giá trị TTL của gói tin đi 1 đơn vị trước khi chuyển tiếp nó đi. Khi TTL giảm về 0, router sẽ hủy gói tin và gửi lại một thông điệp ICMP Time Exceeded (ICMP Type 11). (Trong bối cảnh này, "thời gian" được đo bằng số lượng router, không phải số giây.)
>*phần này mình đã tìm hiểu rồi , đọc tại đây=>*[[Active Reconnaissance#3-ping]]

Output terminal bên dưới hiển thị kết quả chạy traceroute để khám phá các router nằm giữa hệ thống của chúng ta và example.com. Một số router không phản hồi; nói cách khác, chúng hủy gói tin mà không gửi lại bất kỳ thông điệp ICMP nào. Các router thuộc về ISP có thể phản hồi, làm lộ địa chỉ IP private của chúng. Ngoài ra, một số router phản hồi và hiển thị địa chỉ IP public, điều này cho phép chúng ta tra cứu domain name và phát hiện vị trí địa lý của chúng. Cuối cùng, luôn có khả năng thông điệp ICMP Time Exceeded bị chặn và không bao giờ đến được chỗ chúng ta.
![[Pasted image 20260811094428.png]]
# 5-Routing
Hãy xét sơ đồ mạng bên dưới. Sơ đồ này chỉ có ba mạng; tuy nhiên, làm thế nào Internet có thể xác định cách chuyển một packet từ Network 1 tới Network 2 hoặc Network 3? Mặc dù đây là một sơ đồ cực kỳ đơn giản, chúng ta vẫn cần một thuật toán để tìm cách kết nối Network 1 với Network 2, Network 3 và ngược lại.
![Three networks are connected to the Internet through its own router.|566](https://cdn-images.tryhackme.com/user-uploads/5f04259cf9bf5b57aed2c476/room-content/5f04259cf9bf5b57aed2c476-1719849271800.svg)
Hãy xét một sơ đồ chi tiết hơn. Internet bao gồm hàng triệu router và hàng tỷ thiết bị. Mạng bên dưới là một tập hợp con rất nhỏ của Internet. Người dùng di động có thể truy cập web server; tuy nhiên, để điều này diễn ra, mỗi router trên tuyến đường cần gửi các packet qua link phù hợp. Rõ ràng có nhiều hơn một đường đi (route) kết nối người dùng di động và web server. Chúng ta cần một *routing algorithm* để router xác định nên sử dụng link nào.
![[Pasted image 20260811100508.png|583]]

Các routing algorithm nằm ngoài phạm vi bài học này; tuy nhiên, chúng ta sẽ mô tả ngắn gọn một vài routing protocol để bạn làm quen với tên gọi của chúng:

**OSPF** (Open Shortest Path First): OSPF là một routing protocol cho phép các router chia sẻ thông tin về network topology và tính toán các tuyến đường tối ưu nhất để truyền dữ liệu. Nó thực hiện điều này bằng cách <u>cho các router trao đổi bản cập nhật về trạng thái của các link và mạng kết nối với chúng.</u> Bằng cách này, mỗi router có một bản đồ hoàn chỉnh về mạng và có thể xác định các route tốt nhất để đến bất kỳ đích nào.

**EIGRP** (Enhanced Interior Gateway Routing Protocol): EIGRP là một routing protocol độc quyền của Cisco, kết hợp các khía cạnh của nhiều routing algorithm khác nhau. <u>Nó cho phép các router chia sẻ thông tin về các mạng mà chúng có thể truy cập và chi phí (cost - như băng thông hoặc độ trễ) gắn liền với các route đó</u>. Sau đó, các router sử dụng thông tin này để chọn tuyến đường hiệu quả nhất để truyền dữ liệu.

**BGP** (Border Gateway Protocol): <u>BGP là routing protocol chính được sử dụng trên Internet</u>. Nó cho phép các mạng khác nhau (như mạng của các nhà cung cấp dịch vụ Internet - ISP) trao đổi thông tin định tuyến và thiết lập đường đi cho dữ liệu di chuyển giữa các mạng này. BGP giúp đảm bảo dữ liệu có thể được định tuyến hiệu quả trên toàn bộ Internet, ngay cả khi đi qua nhiều mạng khác nhau.

**RIP** (Routing Information Protocol): RIP là một routing protocol đơn giản thường được sử dụng trong các mạng nhỏ. Các router chạy RIP chia sẻ thông tin về các mạng mà chúng có thể truy cập và số lượng hop (router) cần thiết để đến đó. Kết quả là mỗi router sẽ xây dựng một routing table dựa trên thông tin này, chọn các route có số lượng hop ít nhất để đến từng điểm đích.
*=> phần này đọc đau đầu quá, khi nào cần học thì tra lại*

# 6-Nat
Như đã thảo luận trong room Networking Concepts, chúng ta đã tính toán rằng IPv4 có thể hỗ trợ tối đa 4 tỷ thiết bị. Với sự gia tăng số lượng thiết bị kết nối Internet, từ máy tính, smartphone đến camera an ninh hay máy giặt, không gian địa chỉ IPv4 rõ ràng sẽ nhanh chóng bị cạn kiệt. <u>Một giải pháp cho việc cạn kiệt địa chỉ là Network Address Translation (NAT).</u>

<u>Ý tưởng đằng sau NAT là sử dụng một public IP address để cung cấp quyền truy cập Internet cho nhiều private IP address.</u> Nói cách khác, nếu bạn kết nối một công ty gồm 20 máy tính, bạn có thể cấp quyền truy cập Internet cho cả 20 máy tính bằng cách dùng một public IP address duy nhất thay vì 20 public IP address. (Lưu ý: Về mặt kỹ thuật, số lượng địa chỉ IP luôn được biểu diễn dưới dạng lũy thừa của 2. Nói cho chính xác, khi dùng NAT, bạn đăng ký 2 public IP address thay vì 32. Nhờ đó, bạn đã tiết kiệm được 30 public IP address.)
![[Pasted image 20260811101627.png]]
Khác với routing (vốn là cách tự nhiên để định tuyến packet đến host đích), các router hỗ trợ NAT phải tìm cách theo dõi các kết nối đang diễn ra. Do đó, router hỗ trợ NAT <u>duy trì một bảng biên dịch địa chỉ mạng giữa mạng internal và external.</u> Thông thường, <u>mạng internal sẽ sử dụng dải private IP address, trong khi mạng external sẽ sử dụng public IP address.</u>

Trong sơ đồ bên dưới, nhiều thiết bị truy cập Internet thông qua một router hỗ trợ NAT. Router duy trì một bảng ánh xạ internal IP address và port number với external IP address và port number tương ứng. Ví dụ, laptop có thể thiết lập kết nối tới một web server nào đó. Từ phía laptop, kết nối được khởi tạo từ IP address 192.168.0.129 với TCP source port number 15401; tuy nhiên, web server sẽ thấy chính kết nối đó xuất phát từ 212.3.4.5 và TCP port number 19273, đúng như hiển thị trong translation table. Router thực hiện việc biên dịch địa chỉ này một cách hoàn toàn mượt mà và trong suốt.