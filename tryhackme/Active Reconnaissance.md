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