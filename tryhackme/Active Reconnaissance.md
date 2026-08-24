# 1-Intro
Active reconnaissance là quá trình tương tác trực tiếp với hệ thống hoặc mạng mục tiêu để thu thập thông tin về nó. Passive reconnaissance thu thập dữ liệu từ các nguồn công khai mà không gửi bất kỳ traffic nào đến mục tiêu. Ngược lại, active reconnaissance yêu cầu truyền các packet, tạo kết nối và probe các service. Sự khác biệt này rất quan trọng vì các kỹ thuật active sẽ để lại dấu vết dưới dạng log entry, alert, WAF block và trigger.

Room trước về passive reconnaissance đã đề cập đến việc thu thập thông tin tình báo từ DNS record, dữ liệu WHOIS, certificate log và các dịch vụ như Shodan mà mục tiêu không hề hay biết. Room này chuyển sang tương tác trực tiếp. Bạn sẽ sử dụng *web browser*, *ping*, *traceroute*, *telnet* và *netcat* để tương tác với mục tiêu và khai thác những thông tin mà các phương pháp passive không thể tiết lộ.
*=> bắt đầu học về các công cụ trên*

Active reconnaissance bao gồm việc truy cập trực tiếp bằng cách truy cập website, ping host, trace route và kết nối tới các port. Điều này có thể tiết lộ các live service, open port, banner và network path, nhưng nó để lại dấu vết (footprint).

Yêu cầu tiên quyết: Room này yêu cầu bạn đã quen thuộc với các khái niệm networking cốt lõi như TCP, UDP, ICMP và port number. Nếu chưa quen thuộc với các thuật ngữ này, hãy hoàn thành module [[Networking]] trước khi tiếp tục.

<u>Active reconnaissance hiện mang nhiều rủi ro và dễ bị phát hiện hơn bao giờ hết. </u>Hầu hết các tổ chức đều triển khai các CDN như Cloudflare và Akamai cùng với WAF và mô hình zero-trust giúp ghi log hoặc chặn thẳng các probe bất thường. Việc áp dụng IPv6 đồng nghĩa với việc nhiều host phản hồi ping6 nhưng lại filter ICMPv4. HTTPS chiếm ưu thế trong web traffic, khiến các giao thức plaintext hầu như không còn được sử dụng cho các tương tác web. Về phía phòng thủ, SIEM, giải pháp EDR và các hệ thống cloud-native monitoring phát hiện các chuỗi hành vi reconnaissance một cách nhanh chóng và chính xác.
*=> tức là hiện nay các công nghệ ghi log đã phát triển rất mạnh để decteect active recon một cách nhanh chóng và chính xác*

Dù các giao thức mã hóa đang chiếm ưu thế, các công cụ như telnet vẫn rất đáng để tìm hiểu. Chúng minh họa các nguyên lý cơ bản của banner grabbing và cách tương tác với giao thức cleartext, giúp bạn hiểu lý do *tại sao các giải pháp thay thế hiện đại như netcat và curl lại được ưu tiên hơn.* Bạn cũng sẽ gặp các lỗ hổng liên quan đến telnet trong các môi trường legacy, nên việc hiểu cách nó hoạt động vẫn mang lại giá trị thực tế.

Quy tắc quan trọng: Không bao giờ thực hiện active reconnaissance nếu không có ủy quyền hợp pháp rõ ràng bằng văn bản đã ký, chẳng hạn như hợp đồng penetration testing hoặc phạm vi (scope) của chương trình bug bounty. Việc thăm dò trái phép là bất hợp pháp ở hầu hết các quốc gia.

*Dưới góc độ red team, mục tiêu là hòa nhập vào traffic thông thường (blend in)*. Một web browser truy cập website trông hoàn toàn giống với traffic bình thường của hàng ngàn user hợp lệ, và các request được tùy biến khéo léo với chuỗi User-Agent thực tế cùng khoảng thời gian gửi chậm đôi khi có thể qua mặt được các cơ chế phát hiện cơ bản. *Dưới góc độ blue team, các active probe sẽ xuất hiện trong access log, firewall log, WAF event và IDS alert*. Việc tự theo dõi mức độ bộc lộ của hệ thống sẽ giúp phát hiện sớm các hoạt động reconnaissance.

Mục tiêu bài học
Khi hoàn thành room này, bạn có thể:
- Sử dụng web browser và Developer Tools để kiểm tra header, file nguồn JavaScript và certificate phục vụ mục đích reconnaissance.
- Sử dụng ping để kiểm tra khả năng kết nối tới host (reachability) và suy luận thông tin hệ điều hành từ giá trị TTL.
- Sử dụng *traceroute* và *mtr* để *vẽ bản đồ network path và phát hiện các intermediate hop. => cái này là sao z*
- Sử dụng telnet cho công việc l*egacy banner grabbing* và hiểu lý do các công cụ hiện đại được ưu tiên hơn.
- Sử dụng netcat (nc) để thực hiện *banner grabbing,* port probing cơ bản và giao tiếp client-server đơn giản.
 *=> khái niệm banner grabing là gì z?*

Các công cụ này rất trực quan và thường được cài sẵn trên các distro như AttackBox, Kali và Parrot. Room này phù hợp cho người mới bắt đầu muốn nắm vững nền tảng active reconnaissance trước khi chuyển sang các room tiếp theo.

# 2- web browser
Web browser là một trong những công cụ tiện lợi và ít gây nghi ngờ nhất cho active reconnaissance. Nó có mặt trên hầu như mọi hệ thống, và traffic của nó hòa nhập hoàn toàn với hoạt động thông thường của user. Điều này khiến defender rất khó phân biệt giữa reconnaissance và truy cập hợp lệ.
### Khái niệm cơ bản ở tầng Transport
Browser mặc định kết nối tới *port 80 cho HTTP* thông thường, dù điều này hiện khá hiếm vì hầu hết các trang web đều tự động redirect sang HTTPS. *Port 443 là tiêu chuẩn cho HTTPS* và được hầu như toàn bộ website hiện nay sử dụng.

Nhiều trang web hiện đại cũng hỗ trợ HTTP/3, giao thức sử dụng QUIC. QUIC là transport protocol do Google phát triển ban đầu, kết hợp chức năng của TCP và TLS thành một giao thức duy nhất chạy trên UDP port 443. Kết quả là kết nối nhanh và tin cậy hơn so với mô hình TCP+TLS truyền thống. Bạn có thể nhận biết traffic HTTP/3 trong tab Network của browser, nơi cột protocol hiển thị `h3`.

Bạn có thể truy cập các service trên các non-standard port bằng cách chỉ định rõ chúng trong URL. Ví dụ: truy cập [https://target.com:8443/](https://target.com:8443/) hoặc [http://192.168.1.100:8080/](http://192.168.1.100:8080/) sẽ khiến browser thử kết nối tới port đó. Nếu web server đang listening, trang web sẽ tải thành công.

### Developer Tools
Nhấn `Ctrl + Shift + I` trên Windows và Linux, hoặc `Option + Command + I` trên macOS trong Firefox, Chrome, Edge hoặc hầu hết các browser dựa trên Chromium để mở Developer Tools. Nhiều tab trong đây trực tiếp phục vụ cho reconnaissance:
- **Tab Network:** Hiển thị tất cả request và response theo thời gian thực. Bao gồm request và response header như Server, X-Powered-By và Content-Security-Policy, cùng với dữ liệu thời gian, status code, cũng như cookie gửi và nhận.
- **Tab Console:** Cho phép bạn thực thi các đoạn JavaScript snippet trực tiếp trong page context, xem lỗi và tương tác với DOM.
- **Tab Sources:** Cho phép bạn duyệt các file JavaScript, CSS và HTML được trang tải lên. Đây là một trong những kỹ thuật reconnaissance thực tế nhất thông qua browser. Các file nguồn JavaScript thường chứa hardcoded API endpoint, cấu trúc thư mục, tham chiếu tới các internal service và comment của developer chưa từng có ý định công khai. Kiểm tra các file này có thể làm lộ ra thông tin ẩn trên trang đã render.
- **Tab Application:** Trong mục Storage, cho phép bạn kiểm tra cookie, Local Storage và Session Storage. Các vùng lưu trữ này đôi khi chứa session token, API key vô tình bị lộ phía client-side, tracking parameter hoặc dữ liệu xác thực.
- **Tab Security:** Cung cấp chi tiết certificate bao gồm issuer, thời hạn hiệu lực và Subject Alternative Names (SANs). SANs thường tiết lộ thêm các subdomain hoặc domain liên quan thuộc về cùng một tổ chức.
Bên dưới là hình ảnh Firefox Developer Tools. Chrome và Edge cung cấp giao diện tương tự.
[[devtool]]
![[Pasted image 20260808063033.png]]
### Browser Extensions
Các browser extension có thể biến browser thành một nền tảng reconnaissance mạnh mẽ hơn. Dưới đây là các công cụ phổ biến và đang được duy trì tích cực:
- **FoxyProxy:** Cho phép bạn chuyển đổi linh hoạt giữa các proxy như Burp Suite, OWASP ZAP và các SOCKS5 tunnel. Điều này rất hữu ích khi cần intercept hoặc điều hướng traffic qua các công cụ khác nhau trong quá trình làm dự án.
- **User-Agent Switcher and Manager:** Thay đổi chuỗi User-Agent để giả lập các browser, hệ điều hành hoặc thiết bị khác nhau. Bạn có thể giả dạng thành Mobile Safari hoặc phiên bản browser cũ hơn để phát hiện các endpoint dành riêng cho thiết bị di động hoặc hành vi theo phiên bản. <u>Tuy nhiên, nhiều WAF và CDN hiện đại có thể phát hiện việc thay đổi User-Agent bất thường hoặc liên tục</u>.
- **Wappalyzer:** Tự động nhận diện các công nghệ được sử dụng trên trang web, bao gồm nền tảng CMS, web server, JavaScript framework, công cụ analytics, CDN và database. Nó chạy ngầm (passively) trong khi bạn duyệt web và là một trong những extension phổ biến nhất cho việc fingerprinting công nghệ nhanh chóng.
![[Pasted image 20260808063049.png|617]]
Các lựa chọn thay thế hữu ích khác bao gồm BuiltWith Technology Profiler (tương tự Wappalyzer nhưng đôi khi nhận diện được các công nghệ khác), WhatRuns (lựa chọn thay thế nhẹ hơn) và Library Detector cho Chrome/Firefox (tập trung cụ thể vào các thư viện và framework JavaScript). Theo thời gian, hầu hết mọi người đều rút gọn còn khoảng 3 đến 5 extension phù hợp nhất với workflow của mình.

Mặc dù việc duyệt web trông có vẻ bình thường, các pattern bất thường vẫn có thể kích hoạt các quy tắc hành vi trên WAF hoặc endpoint detection system hiện đại. Việc tải trang quá nhanh, header bị chỉnh sửa, dùng DevTools liên tục và chuỗi User-Agent bất thường đều là dấu hiệu mà defender tìm kiếm. Mục tiêu là luôn mô phỏng hành vi của user hợp lệ bất cứ khi nào có thể.

# 3-ping
Tên gọi ping bắt nguồn từ âm thanh của xung sonar: gửi đi một tín hiệu và lắng nghe tiếng vọng trả về. Trong networking, lệnh ping cũng thực hiện điều tương tự. Nó gửi một packet thử nghiệm nhỏ tới host từ xa và chờ phản hồi. Quá trình trao đổi đơn giản này cho biết mục tiêu có thể kết nối tới qua mạng hay không, cũng như có đang online và phản hồi hay không.
### Cách Ping hoạt động
Ping sử dụng giao thức ICMP (Internet Control Message Protocol). Nó gửi một packet ICMP Echo Request (type 8). Nếu mục tiêu nhận được packet và được phép trả lời, nó sẽ gửi lại ICMP Echo Reply (type 0). Quá trình trao đổi này rất nhẹ và nhanh, đó là lý do ping trở thành bước kiểm tra tiêu chuẩn đầu tiên trước khi dành thời gian cho việc scanning chi tiết hơn.
### Cách sử dụng cơ bản
Trên Linux và macOS, sử dụng flag `-c` để chỉ định số lượng packet cần gửi:
```
ping -c 5 MACHINE_IP
```
Bạn cũng có thể ping một hostname, khi đó quá trình DNS resolution sẽ diễn ra trước:

```
ping -c 5 tryhackme.com
```

Trên Windows, flag tương đương là `-n`:
```
ping -n 5 MACHINE_IP
```
Nếu bạn bỏ qua số lượng packet trên Linux, ping sẽ chạy vô hạn. Nhấn Ctrl+C để dừng.

Bạn có thể bắt buộc sử dụng một phiên bản IP cụ thể bằng các flag `-4` và `-6`. Điều này hữu ích trong môi trường dual-stack khi một hostname phân giải ra cả địa chỉ IPv4 và IPv6. Trên một số hệ thống, `ping6` sẵn có dưới dạng một lệnh độc lập cho IPv6:

```
ping -4 -c 5 MACHINE_IP
ping -6 -c 5 MACHINE_IPV6
```
### Đọc hiểu output: Ping thành công
Ví dụ dưới đây cho thấy một mục tiêu đang alive và cho phép ICMP:

```
user@AttackBox$ ping -c 5 MACHINE_IP
PING MACHINE_IP (MACHINE_IP) 56(84) bytes of data.
64 bytes from MACHINE_IP: icmp_seq=1 ttl=64 time=0.512 ms
64 bytes from MACHINE_IP: icmp_seq=2 ttl=64 time=0.478 ms
64 bytes from MACHINE_IP: icmp_seq=3 ttl=64 time=0.491 ms
64 bytes from MACHINE_IP: icmp_seq=4 ttl=64 time=0.503 ms
64 bytes from MACHINE_IP: icmp_seq=5 ttl=64 time=0.485 ms
--- MACHINE_IP ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4098ms
rtt min/avg/max/mdev = 0.478/0.494/0.512/0.012 ms
```
Mục tiêu đã trả lời cả 5 request, xác nhận nó đang online và có thể kết nối tới. Mức packet loss 0% xác nhận network path ổn định. Thời gian round-trip khoảng 0.5 ms là rất thấp, cho thấy mục tiêu có thể nằm trong cùng mạng nội bộ.

Trường *TTL* (Time To Live) cần được chú ý đặc biệt. Mặc dù có từ "Time" trong tên gọi, TTL thực chất đại diện cho số lượng router (hop) tối đa mà một packet có thể đi qua trước khi bị hủy. Mỗi router trên đường đi sẽ giảm TTL đi 1. Giá trị TTL ban đầu do hệ điều hành thiết lập, khiến nó trở thành một dấu hiệu hữu ích cho việc fingerprinting OS. Linux thường dùng giá trị TTL bắt đầu là 64, trong khi Windows thường dùng 128.*=> đại khái mỗi lần đi qua một hop, TTL -=1*
Tuy nhiên, các router trung gian sẽ giảm giá trị này trước khi packet đến tay bạn. Giá trị TTL là 58 trong phản hồi có khả năng chỉ ra một hệ thống Linux cách bạn 6 hop, chứ không phải một hệ điều hành hoàn toàn khác. Hãy ghi nhớ điều này khi đọc hiểu kết quả.
### Đọc hiểu output: Không có phản hồi

```
user@AttackBox$ ping -c 5 MACHINE_IP
PING MACHINE_IP (MACHINE_IP) 56(84) bytes of data.
From ATTACKBOX_IP icmp_seq=1 Destination Host Unreachable
From ATTACKBOX_IP icmp_seq=2 Destination Host Unreachable
From ATTACKBOX_IP icmp_seq=3 Destination Host Unreachable
From ATTACKBOX_IP icmp_seq=4 Destination Host Unreachable
From ATTACKBOX_IP icmp_seq=5 Destination Host Unreachable
--- MACHINE_IP ping statistics ---
5 packets transmitted, 0 received, +5 errors, 100% packet loss, time 4098ms
```
Có một số lý do phổ biến dẫn đến việc không nhận được phản hồi:
- Máy phòng thí nghiệm có thể đã tắt nguồn, bị crash hoặc vẫn đang khởi động.
- Router hoặc firewall trên đường đi có thể đang chặn ICMP Echo Request.
- Mục tiêu có thể nằm sau NAT vốn loại bỏ ICMP.
- Windows Firewall mặc định chặn ping trên hầu hết các phiên bản.
- Corporate firewall, các nhà cung cấp cloud như AWS, Azure, GCP, cùng các WAF và CDN hiện đại thường chặn hoàn toàn ICMP.
- Mạng hoặc máy tính của chính bạn cũng có thể đang chặn ICMP chiều outbound.
### Bảng tham khảo nhanh
| Kết quả                                      | Ý nghĩa có khả năng nhất                 | Bước tiếp theo                       |
| -------------------------------------------- | -------------------------------------------- | ---------------------------------------- |
| Phản hồi nhanh, packet loss thấp hoặc bằng 0 | Mục tiêu đang online và cho phép ICMP        | Chuyển sang port scanning                |
| "Destination Host Unreachable"               | Mục tiêu bị tắt hoặc không có route          | Kiểm tra xem máy đã bật chưa             |
| Packet loss 100% không kèm thông báo lỗi     | ICMP bị filter hoặc bị chặn                  | Thử host discovery qua TCP/UDP bằng Nmap |
| Latency cao hoặc loss nặng                   | Mạng bị nghẽn, khoảng cách xa hoặc bị filter | Điều tra network path bằng traceroute    |

# 4-traceroute
Lệnh traceroute dùng để<u> dò đường đi của các packet từ hệ thống của bạn tới host mục tiêu</u>. Mục đích của nó là phát hiện địa chỉ IP của các router (hop) trên đường đi và xác định có bao nhiêu hop nằm giữa bạn và đích đến. Thông tin này rất hữu ích để hiểu topology mạng, xác định vị trí xảy ra nghẽn (latency) hoặc chặn lọc (filtering), và vẽ bản đồ hạ tầng.
>*hop:  là các ip trên đường đi từ mình tới host*

![[Pasted image 20260809071039.png|637]]
Tuy nhiên, tuyến đường mà packet đi qua không cố định. Nhiều router sử dụng các giao thức routing động như BGP hoặc OSPF để tự điều chỉnh theo thay đổi của mạng. Các mạng hiện đại cũng áp dụng load balancing và anycast routing, nghĩa là các packet có thể đi theo các con đường khác nhau ngay cả trong các lần chạy lệnh liên tiếp.
>*tức là route ko cố định, mỗi lần chạy sẽ cho ra 1 route khác nhau*

Trên Linux và macOS, lệnh sử dụng là *traceroute 10.49.128.55*. Trên Windows, lệnh tương đương là `tracert 10.49.128.55`. Với IPv6, dùng `traceroute -6 MACHINE_IPV6` hoặc lệnh độc lập `traceroute6`.

### Cách Traceroute hoạt động

Không có cách trực tiếp nào để tìm toàn bộ tuyến đường từ hệ thống của bạn tới mục tiêu. Thay vào đó, traceroute lợi dụng trường TTL (Time To Live) trong IP header. Mỗi router xử lý packet sẽ giảm giá trị TTL đi 1 trước khi forward nó đi. Khi TTL giảm xuống 0, router sẽ drop packet và gửi thông điệp ICMP Time-to-Live Exceeded ngược về sender.

Bằng cách gửi các packet với giá trị TTL tăng dần bắt đầu từ 1, traceroute ép từng router tiếp theo trên đường đi phải phản hồi địa chỉ IP của mình. Giá trị TTL bằng 1 khiến router đầu tiên drop packet và phản hồi. TTL bằng 2 sẽ tới router thứ hai trước khi bị drop. Quá trình này tiếp tục cho đến khi packet đến được đích.

Một số router được cấu hình để không gửi thông điệp ICMP Time-to-Live Exceeded. Điều này rất phổ biến trong các môi trường bảo mật nhằm ngăn chặn hoạt động reconnaissance. Các router này sẽ hiển thị dưới dạng dấu `*` trong output của traceroute.

Mặc định trên Linux, traceroute gửi các UDP datagram. Để chuyển sang tracing dựa trên TCP (hữu ích để vượt qua các bộ lọc UDP), dùng `traceroute -T 10.49.128.55`. Để dùng tracing dựa trên ICMP, dùng `traceroute -I 10.49.128.55`.
>*phần này đọc không hiểu gì hết*
### Đọc hiểu Output

Hai ví dụ dưới đây hiển thị lệnh `traceroute tryhackme.com` được thực hiện 2 lần từ AttackBox của TryHackMe. Chúng minh họa cách tuyến đường có thể thay đổi giữa các lần chạy liên tiếp.

 **Traceroute A**![[Pasted image 20260809072117.png|700]]
*Output này chứa 14 dòng được đánh số, mỗi dòng đại diện cho một hop*. H<u>ệ thống gửi 3 packet tại mỗi giá trị TTL, do đó bạn thấy tối đa 3 địa chỉ IP và 3 khoảng thời gian round-trip trên mỗi dòng.</u> <u>Một số dòng hiển thị các IP khác nhau ở cùng một số hop vì load balancing khiến 3 packet đi theo các tuyến đường hơi khác nhau</u>.
![[Pasted image 20260809071841.png]]
Xét dòng 12: router tại `99.83.69.207` phản hồi cả 3 packet với thời gian round-trip lần lượt là 17.603 ms, 15.827 ms, và 17.351 ms. Ở dòng 3, chỉ 1 trong 3 packet nhận được phản hồi. Hai dấu `*` cho biết hai thông điệp ICMP Time-to-Live Exceeded còn lại đã bị drop hoặc bị chặn bởi router.

Dòng cuối cùng (hop 14) hiển thị `172.67.69.208`, khớp với địa chỉ IP đích của `tryhackme.com` hiển thị ở phần header của traceroute. Điều này có nghĩa là quá trình traceroute đã hoàn thành thành công qua tổng cộng 14 hop. Hops từ 1 đến 13 là các router trung gian, và hop 14 chính là điểm đích.
>*chốt lại:
>- Tại mỗi hop nó sẽ gửi 3 gói tin để check, nên mỗi dòng nó mới có 3 khoảng thời gian.
>-còn tại các dòng có 3 địa chỉ ip thì là do ở đó có load balancing, khiến 3 gói tin đi theo 3 hướng khác nhau*

 **Traceroute B**![[Pasted image 20260809072349.png]]
Ở lần chạy thứ hai này, các packet đi theo một tuyến đường dài hơn nhiều và hoàn thành trong 26 hop. Hop 26 chính là điểm đích `104.26.11.229`, còn các hop từ 1 đến 25 là các router trung gian. So sánh hai lần chạy cho thấy cả số lượng hop lẫn các router cụ thể trên đường đi đều không giữ cố định. *Đây là hành vi bình thường của traffic khi đi qua các mạng bên ngoài, đặc biệt khi có sự tham gia của các CDN như Cloudflare hay Akamai* (những bên sử dụng anycast và load balancing để tối ưu hóa tuyến đường).

*Có ba nhận xét quan trọng từ các ví dụ này:*

1. Số lượng hop giữa hệ thống của bạn và mục tiêu phụ thuộc vào thời điểm bạn chạy lệnh.
2. Không có gì đảm bảo các packet sẽ đi theo cùng một tuyến đường, ngay cả trong khoảng thời gian ngắn, vì load balancing, cơ chế failover và cập nhật routing động đều có thể làm thay đổi con đường.
3. Một số router trả về các địa chỉ IP public mà bạn có thể xem xét tùy theo scope của dự án, mặc dù chúng có thể thuộc về bên thứ ba. Một số router không trả về phản hồi, điều này có thể do rate limiting, quy tắc firewall, hoặc cấu hình ẩn phản hồi ICMP.

### Các kỹ thuật bổ sung
Dùng *mtr 10.49.128.55* (My Traceroute) để xem thông tin liên tục theo thời gian thực kết hợp giữa traceroute và số liệu thống kê kiểu ping, hiển thị tỷ lệ packet loss và latency trên từng hop. Để vượt qua các bộ lọc, thử chế độ TCP với `traceroute -T 10.49.128.55` hoặc chế độ ICMP với `traceroute -I 10.49.128.55`. Với IPv6, lệnh độc lập `traceroute6` đảm bảo tính tương thích trong các mạng dual-stack.

# 5-Telnet
Giao thức TELNET (Teletype Network) được phát triển vào năm 1969 để giao tiếp với hệ thống từ xa thông qua giao diện command-line. Lệnh telnet sử dụng giao thức này cho việc quản trị từ xa, với *port mặc định là 23*. Từ góc độ bảo mật, telnet gửi toàn bộ dữ liệu dưới dạng cleartext, bao gồm cả username và password. Điều này khiến bất kỳ ai có quyền truy cập vào kênh truyền thông đều dễ dàng đánh chặn login credential. *Giải pháp thay thế bảo mật hơn là SSH* (Secure Shell), vốn mã hóa toàn bộ traffic và hiện là tiêu chuẩn cho việc truy cập CLI từ xa.
![[Pasted image 20260809073920.png]]
Mặc dù có những hạn chế về bảo mật, telnet client lại có một đặc tính hữu ích cho reconnaissance. Vì hoạt động trên nền TCP, *bạn có thể dùng nó để kết nối tới bất kỳ TCP port nào và quan sát phản hồi từ server*. Kỹ thuật này được gọi là **banner grabbing**. *Bạn kết nối tới một service và đọc phản hồi ban đầu (gọi là "banner") mà server gửi lại.* <u>Các banner này thường tiết lộ tên phần mềm và phiên bản đang chạy trên port đó.</u>
>*TCP port của cái app nó là 1 cổng dịch vụ trên cái  app đó à?
>grab banner là gì??
>telnet có thể kết nối tới bất kỳ TCP port nào*

Nếu hệ thống chưa cài sẵn telnet, bạn có thể cài đặt trên Debian và Ubuntu bằng lệnh `apt install telnet`. Tuy nhiên, *netcat (nc) và curl thường là những lựa chọn thay thế được ưu tiên hơn vì cung cấp tính năng tương tự với độ linh hoạt cao hơn*.

Ví dụ dưới đây minh họa kỹ thuật banner grabbing đối với một web server trên port 80. Bạn kết nối bằng lệnh `telnet 10.49.128.55 80`, sau đó gửi một HTTP request tối giản. Lệnh `GET / HTTP/1.1` theo sau bởi `host: telnet` và nhấn phím Enter hai lần là đủ để nhận phản hồi từ server.
![[Pasted image 20260809084624.png]]
Kỹ thuật banner grabbing này hoạt động tương tự trên bất kỳ TCP-based service nào. Nếu kết nối tới mail server, bạn sẽ sử dụng các lệnh SMTP hoặc POP3 thay vì HTTP. Nếu kết nối tới FTP server trên port 21, server thường sẽ gửi banner ngay lập tức sau khi kết nối mà không yêu cầu gửi bất kỳ lệnh nào. 
![[Pasted image 20260809084203.png]]
>đaị khái là *banner grabing* nó áp dụng cho các *TCP based service* , như SMTP , POP, HTTP, ...  đều có thể sử dụng
>*=>* Nguyên lý cốt lõi luôn giống nhau: kết nối tới port, đọc thông tin server gửi lại, và tùy chọn gửi các lệnh theo đúng giao thức để khai thác thêm thông

Trong môi trường hiện đại, nhiều service bắt buộc sử dụng mã hóa. Ví dụ, SMTPS chạy trên port 465, và HTTPS chạy trên port 443. *Telnet không thể xử lý các kết nối mã hóa.* *Đối với HTTPS, hãy dùng `curl --head [https://10.49.128.55](https://10.49.128.55)` hoặc `openssl s_client -connect 10.49.128.55:443`. Đối với các service bọc TLS khác, `openssl s_client` hoặc `ncat --ssl` là những công cụ phù hợp.*

# 6-netcat
đọc thêm ở đây[[netcat]]
Netcat (hoặc đơn giản là nc) là một tiện ích networking linh hoạt hỗ trợ cả giao thức TCP và UDP. Nó có thể hoạt động như một client kết nối tới một listening port, hoặc như một server listening trên port do bạn chọn. Khả năng kép này khiến nó trở nên hữu ích cho banner grabbing, port probing, chuyển file đơn giản và giao tiếp client-server cơ bản. Các phiên bản hiện đại như ncat từ dự án Nmap cũng hỗ trợ IPv6 và SSL encryption, giúp nó linh hoạt hơn các công cụ cũ như telnet.
*==> netcat nó xin hơn telnet*
### Banner Grabbing với Netcat

Kỹ thuật banner-grabbing được mô tả ở task trước hoạt động hoàn toàn tương tự với nc. Cú pháp là `nc 10.49.181.64 PORT`. Bạn kết nối tới target port, sau đó gửi các lệnh phù hợp với giao thức để đọc phản hồi của server. Lưu ý rằng bạn có thể cần nhấn Shift+Enter sau dòng GET.

```text
pentester@TryHackMe$ nc 10.49.181.64 80
GET / HTTP/1.1
host: netcat

HTTP/1.1 200 OK
Server: nginx/1.6.2
Date: Tue, 17 Aug 2021 11:39:49 GMT
Content-Type: text/html
Content-Length: 867
Last-Modified: Tue, 17 Aug 2021 11:12:16 GMT
Connection: keep-alive
ETag: "611b9990-363"
Accept-Ranges: bytes
...

```

*=> tương tự như telnet ở bên trên: kết nối+ gửi bản tin=> nhận được response tiết lộ thông tin*
*=> tương tụ thay http bằng ftp, pop,... tùy theo dịch vụ TCP của web server.*

---

### Listening với Netcat

Netcat cũng có thể hoạt động như một server, listening trên một port chỉ định. Điều này hữu ích để kiểm tra kết nối, truyền dữ liệu đơn giản, hoặc thiết lập các kênh giao tiếp cơ bản trong quá trình đánh giá.
*đọc thêm về cách tạo reverseshell sử dụng netcat* [[netcat]]

Trên hệ thống server, chạy `nc -vnlp 1234` để bắt đầu listening trên port 1234. Trên hệ thống client, chạy `nc 10.49.181.64 1234` để kết nối. Sau khi kết nối được thiết lập, bất kỳ văn bản nào được gõ ở một bên sẽ được truyền sang bên còn lại. Như bạn có thể nhớ từ module Linux Fundamentals, t<u>hứ tự chính xác của các flag không quan trọng miễn là số port đứng ngay sau </u>`-p`.
*=> bên gửi là ko cầm cờ gì
=> bên nhận thì cần cờ -l listening để lắng nghe kết nối*

| Flag | Ý nghĩa |
| --- | --- |
| -l | Chế độ Listen |
| -p | Chỉ định số port |
| -n | Chỉ dùng định dạng số; không thực hiện DNS resolution cho hostname |
| -v | Verbose output, hữu ích cho việc debug |
| -vv | Very verbose output |
| -k | Tiếp tục listening sau khi client ngắt kết nối |

*=>Flag* `-p` phải đứng ngay trước số port. Flag `-n` giúp tránh việc tra cứu DNS và các cảnh báo liên quan. Các số port dưới 1024 yêu cầu quyền root để listen. Để listening trên IPv6, thêm flag `-6` bằng `nc -6 -lp 1234`. Nếu cần mã hóa khi truyền dữ liệu nhạy cảm, hãy dùng `ncat --ssl` hoặc kết hợp `nc` với một công cụ như stunnel.

# 7-tổng kết
This room covered five core tools for active reconnaissance. The web browser with Developer Tools reveals server technologies, headers, JavaScript sources, and certificate details. `ping` confirms whether a target is reachable and provides TTL-based clues about its operating system. `traceroute` maps the network path between you and the target, revealing intermediate routers and potential filtering points. `telnet` and `netcat` connect to individual ports to grab banners and identify running services along with their versions.

These tools are simple individually, but combining them gives you a structured picture of a target before moving on to more advanced scanners. <u>You might use `ping` to confirm a host is alive, `traceroute` to understand the network path, and then `nc` to probe specific ports and identify services.</u> For HTTP-based services, prefer `curl -I 10.48.188.11` or `nc 10.48.188.11 PORT` over telnet for banner grabbing, as they provide more secure and flexible options.

## Quick Reference

| Command              | Example                                                         |
| -------------------- | --------------------------------------------------------------- |
| ping                 | `ping -c 10 10.48.188.11` on Linux or macOS                     |
| ping                 | `ping -n 10 10.48.188.11` on Windows                            |
| ping (IPv6)          | `ping -6 MACHINE_IPV6` or `ping6 MACHINE_IPV6`                  |
| traceroute           | `traceroute 10.48.188.11` on Linux or macOS                     |
| tracert              | `tracert 10.48.188.11` on Windows                               |
| traceroute (IPv6)    | `traceroute -6 MACHINE_IPV6` or `traceroute6 MACHINE_IPV6`      |
| mtr                  | `mtr 10.48.188.11` for real-time path monitoring                |
| telnet (legacy)      | `telnet 10.48.188.11 PORT_NUMBER`                               |
| netcat as client     | `nc 10.48.188.11 PORT_NUMBER`                                   |
| netcat as server     | `nc -lvnp PORT_NUMBER`                                          |
| netcat (IPv6)        | `nc -6 MACHINE_IPV6 PORT_NUMBER`                                |
| curl for HTTP banner | `curl -I http://10.48.188.11` or `curl -I https://10.48.188.11` |

| Operating System | Developer Tools Shortcut |
| ---------------- | ------------------------ |
| Linux or Windows | `Ctrl + Shift + I`       |
| macOS            | `Option + Command + I`   |

## Next Steps

The tools covered in this room represent the foundation of active reconnaissance. The next rooms in this module cover [Nmap](https://tryhackme.com/room/nmap01), which automates and extends host discovery and port scanning far beyond what `ping` and `nc` can achieve individually. For deeper exploration of web-based reconnaissance, the [Walking An Application](https://tryhackme.com/room/walkinganapplication) room provides a thorough treatment of Developer Tools and manual web inspection techniques. The advanced topics hinted at in this room, such as slow timing, chaining, and blended traffic, are covered in the stealth scanning and rooms.
