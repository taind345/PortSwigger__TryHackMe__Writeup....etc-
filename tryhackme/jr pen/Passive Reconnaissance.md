# 1-INTRO

trong room này, *passive reconnaissance* là việc thu thập thông tin tình báo từ các nguồn công khai mà không tương tác với mục tiêu. điều này trái ngược với active reconnaissance, nơi bạn tương tác trực tiếp với mục tiêu và có nguy cơ bị phát hiện.
passive recon vẫn là một trong những giai đoạn mạnh mẽ và ít rủi ro nhất trong penetration testing, bug bounty và threat hunting. ngay cả khi có các luật riêng tư khắt khe hơn (như gdpr, ccpa), một lượng lớn dữ liệu hữu ích vẫn lộ ra công khai thông qua dns, whois, certificate logs, search engine và các nền tảng quét thiết bị.
>*đại khái là tìm thông tin trên internet-các nguồn công khai., các nền tảng như shodan,..*

**mục tiêu bài học:**
khi hoàn thành room này, bạn có thể:
- dùng *whois* để truy vấn thông tin đăng ký domain.
- dùng *dig* (và *nslookup*) để truy vấn các dns record. [[DNS record là gi]]
- hiểu lý do tại sao truy vấn các server whois và dns công khai lại được coi là passive.
- phát hiện subdomain bằng *dnsdumpster* và *certificate transparency logs*.
- thu thập thông tin về các dịch vụ đang mở bằng *shodan.io.*

yêu cầu kiến thức tiên quyết:
- nắm vững các khái niệm networking cơ bản và sử dụng thành thạo terminal. nếu cần ôn lại, hãy hoàn thành room networking fundamentals trước.
lưu ý quan trọng:
- nếu bạn chưa đăng ký trả phí, attackbox trên trình duyệt sẽ không có truy cập internet trực tiếp. đối với các câu hỏi yêu cầu tra cứu web (dnsdumpster, shodan, crt.sh...), hãy kết nối qua openvpn vào mạng tryhackme. điều này giúp máy cục bộ (hoặc attackbox) có đầy đủ kết nối internet.
- không cần deploy máy trong room này. tất cả bài tập đều dùng dữ liệu công khai trên internet, chủ yếu là các domain liên quan đến tryhackme.

# 2-passive recon vs active recon
trước khi mạng máy tính tồn tại, tôn tử đã viết trong binh pháp tôn tử: "biết địch biết ta, trăm trận trăm thắng"

trong cybersecurity, nguyên lý này ứng với hai vai trò. ở vị trí attacker (hoặc ethical hacker), bạn thu thập thông tin tình báo về mục tiêu để tìm lỗ hổng. ở vị trí defender, bạn phải hiểu đối thủ có thể thu thập những gì về hệ thống của bạn từ các nguồn công khai để tối thiểu hóa mức độ bộc lộ đó.

reconnaissance (recon) là bước khảo sát ban đầu để thu thập thông tin về mục tiêu. đây là giai đoạn đầu tiên trong các attack framework hiện đại như unified kill chain (nơi recon giúp nắm thông tin ban đầu trước khi chiếm foothold) và các biến thể của cyber kill chain truyền thống. reconnaissance được chia thành hai loại chính.

**passive reconnaissance**
![[Pasted image 20260807231431.png|364]]
passive reconnaissance hoàn toàn phụ thuộc vào thông tin công khai. không có packet nào được gửi đến mục tiêu và không có tương tác trực tiếp nào diễn ra. nó tương tự như việc dùng ống nhòm quan sát lãnh thổ mục tiêu từ khoảng cách an toàn mà không đặt chân lên đất của họ.
các hoạt động passive phổ biến bao gồm:
- truy vấn các record công khai từ open resolver (a, mx, txt...).
- tìm kiếm trên certificate transparency logs (ví dụ: crt.sh) để phát hiện subdomain và certificate được cấp.
- xem thông tin tuyển dụng trên linkedin hoặc trang tuyển dụng của công ty để đoán tech stack.
- đọc tin tức công khai, thông cáo báo chí, hoặc tài liệu rò rỉ trên các trang paste site.
- kiểm tra các thiết bị bị bộc lộ qua các search engine như shodan hoặc censys.
- quét các public github repository để tìm hardcoded credential hoặc configuration file.
>*các tool này dùng kiểu gì??-mình biết dùng mỗi shodan *

**active reconnaissance**
![[Pasted image 20260807231454.png|416]]
active reconnaissance yêu cầu tương tác trực tiếp với mục tiêu. các probe của bạn có thể bị ghi log, phát hiện hoặc chặn. nó tương tự như việc đi đến tận cửa ra vào và cửa sổ để thử khóa, camera và báo động.
các hoạt động active phổ biến bao gồm:
- gửi packet để phát hiện live host (ví dụ: icmp ping, request...).
- port scanning hoặc service enumeration (nmap, masscan...).
- tương tác với web application hoặc api (*fuzzing endpoint*...).
- thử nghiệm social engineering (phishing, vishing, gọi điện pretexting...).
- tiếp cận vật lý (tailgating, giả danh nhà cung cấp...).
>*fuzzing endpoint như thế nào?*frsaax

vì *active reconnaissance có thể bị phát hiện* (bởi ids/ips, waf, logging), nó mang rủi ro cao khiến mục tiêu đề phòng. nếu không có ủy quyền rõ ràng (ví dụ: bug bounty scope hoặc hợp đồng pentest), nó có thể dẫn đến rắc rối pháp lý. passive recon kín đáo hơn nhiều và thường là bước đầu tiên trong thực tế.

lưu ý rằng bất kỳ tương tác trực tiếp nào với cá nhân thuộc tổ chức mục tiêu đều được tính là active reconnaissance, ngay cả khi không liên quan đến packet mạng. ví dụ, tham gia một sự kiện giao lưu và hỏi một nhân viên về technology stack của công ty họ chính là active reconnaissance vì bạn đang tương tác trực tiếp với tổ chức mục tiêu.

mẹo cho defender: <u>các tổ chức hiện nay tự theo dõi passive footprint của chính họ bằng cách dùng cảnh báo từ shodan/censys, ct log watcher và các công cụ tự động </u>để giảm thiểu những gì attacker có thể tìm thấy mà chưa cần chạm vào mạng nội bộ.

# 3-Whois
WHOIS là một query/response protocol được quy định trong RFC 3912. Các WHOIS server *lắng nghe trên port 43* và *cung cấp thông tin đăng ký chi tiết cho domain* name. Domain registrar duy trì các bản ghi này cho các domain mà họ cho thuê.

Từ kết quả trả về của WHOIS, các thông tin sau có thể hiển thị (khi chưa bị ẩn):
- Registrar: Công ty (ví dụ: Namecheap, GoDaddy) thực hiện đăng ký domain.
- Registrant contact information: Tên, tổ chức, địa chỉ, số điện thoại và email. Tuy nhiên, các dịch vụ privacy (trở thành tiêu chuẩn từ 2018) thường thay thế thông tin này thành "Withheld for Privacy" hoặc tương tự.
- Dates: Creation (ngày đăng ký), Updated (lần thay đổi cuối) và Expiration (hạn gia hạn).
- Name servers: Các server đóng vai trò authoritative cho domain.
- Status codes: Ví dụ, clientTransferProhibited cho biết domain đang bị khóa để tránh transfer trái phép.
- Abuse contacts: Email và số điện thoại của registrar để báo cáo vi phạm.
>*Bên trên là mẫu kết quả trả về từ whois, có thể hiểu WHOIS nó sẽ cung cấp cho ta thông tin về 1 domain name* 


Thông tin cá nhân đầy đủ hiện khá hiếm do luật bảo vệ quyền riêng tư (GDPR, CCPA) và việc sử dụng phổ biến các dịch vụ privacy proxy. Trong thực tế, attacker tập trung vào dates (để ước tính độ tuổi domain hoặc căn thời điểm social engineering xung quanh kỳ gia hạn), registrar (để tạo kịch bản phishing), name servers (các điểm yếu tiềm năng) và lịch sử thay đổi. Các dịch vụ như whoxy.com cung cấp các bản snapshot WHOIS trong quá khứ. Dữ liệu WHOIS lịch sử có thể tiết lộ chủ sở hữu cũ, sự thay đổi registrar hoặc quá trình chuyển đổi name server, từ đó chỉ ra các đợt bị xâm nhập hoặc thay đổi hạ tầng trong quá khứ.

**WHOIS đang được thay thế bởi RDAP**
Từ ngày 28 tháng 1 năm 2025, ICANN đã chính thức loại bỏ giao thức WHOIS truyền thống cho các generic top-level domain (gTLDs) để chuyển sang Registration Data Access Protocol (RDAP).

*RDAP* là giải pháp thay thế hiện đại: nó sử dụng HTTPS (bảo mật), trả về kết quả cấu trúc JSON (máy có thể đọc và đồng nhất), hỗ trợ quốc tế hóa, cung cấp cơ chế kiểm soát quyền riêng tư tốt hơn (phân quyền truy cập) và tuân thủ các quy định bảo vệ dữ liệu hiện hành. Mặc dù các legacy whois client vẫn hoạt động (thường qua cơ chế failover hoặc kết nối tới server cũ), RDAP hiện là tiêu chuẩn chính thức. Nhiều công cụ và trình duyệt tự động chuyển hướng sang RDAP, và việc truy cập từ command line rất đơn giản bằng curl hoặc các client chuyên dụng như OpenRDAP.

Để truy vấn WHOIS, bạn dùng *whois command-line* client (nhanh hơn hầu hết các công cụ web) hoặc các trang tra cứu online cho các truy vấn cũ.

Cú pháp: whois DOMAIN_NAME
Trên AttackBox (hoặc Kali/Parrot), chạy:
![[Pasted image 20260807232857.png]]

Ví dụ về RDAP:
Dùng curl để truy vấn một RDAP endpoint công khai (ví dụ: Verisign cho các domain .com). Tiện ích jq giúp định dạng JSON output cho dễ đọc; nó đã được cài sẵn trên AttackBox. Nếu dùng máy cá nhân, bạn có thể cài qua package manager (ví dụ: sudo apt install jq).
![[Pasted image 20260808000159.png]]
*RDAP output có cấu trúc rõ ràng (dễ parse và viết script) và bảo mật hơn.* Bạn sẽ thấy định dạng này xuất hiện ngày càng nhiều trong các công cụ hiện đại.

Những thông tin cần chú ý:
- Redirection chain (chuyển hướng từ Verisign sang registrar server).
- Dates: hữu ích để ước tính độ tuổi công ty hoặc xác định thời điểm phishing liên quan đến gia hạn.
- Name servers: các mục tiêu tiềm năng mới (nếu nằm trong scope).
- Status: các domain bị khóa (ví dụ: clientTransferProhibited) sẽ khó bị chiếm quyền hơn.
Các công cụ online thay thế (nếu lệnh whois gặp lỗi):
- [https://whois.icann.org/](https://whois.icann.org/) (legacy WHOIS)
- [https://lookup.icann.org/](https://lookup.icann.org/) (tra cứu tập trung vào RDAP hiện đại)
- [https://www.whoxy.com/](https://www.whoxy.com/) (snapshot WHOIS lịch sử, miễn phí có giới hạn)
Trên AttackBox, mở terminal và chạy lệnh whois tryhackme.com (hoặc thử ví dụ RDAP curl) để trả lời các câu hỏi bên dưới.

# 4-nslookup và dig
Trong phần trước, WHOIS đã cung cấp cho chúng ta các authoritative name server của domain. Phần này chuyển sang việc truy vấn các DNS record, đây vẫn là một quá trình hoàn toàn passive vì các truy vấn được gửi đến các public hoặc open resolver chứ không gửi trực tiếp đến server của mục tiêu.
![[Pasted image 20260808045048.png|568]]
Các công cụ này *giúp dịch domain name thành IP address*, tìm mail server, hiển thị các *TXT record* (SPF, DMARC, chuỗi xác minh) và nhiều thông tin khác.
[[DNS record là gi]]
**Tại sao nên ưu tiên dig hơn nslookup?**
Phần này giới thiệu hai công cụ truy vấn DNS: nslookup và dig. Cả hai đều truy vấn DNS, nhưng <u>dig (viết tắt của "Domain Information Groper") là lựa chọn hiện đại và được ưu tiên hơn. </u>Nó cung cấp output sạch hơn, hiển thị mặc định các giá trị TTL (cho biết thời gian record được lưu trong cache) và tin cậy hơn cho các truy vấn phức tạp cũng như khi viết script. <u>nslookup được đề cập ở đây để đảm bảo tính tương thích, vì bạn sẽ gặp nó trong các tài liệu cũ và trên hệ thống Windows,</u> nhưng dig nên là công cụ mặc định của bạn.

### -nslookup
nslookup (Name Server Lookup) là công cụ cũ hơn trong hai công cụ.
Cú pháp:
- *nslookup DOMAIN_NAME*: thực hiện tra cứu cơ bản bằng resolver mặc định của bạn.
- *nslookup -type=TYPE DOMAIN_NAME [SERVER]:* chỉ định loại record và một DNS server tùy chọn.

### -Các loại DNS record phổ biến:
- A: Địa chỉ IPv4 của domain.
- AAAA: Địa chỉ IPv6 của domain.
- *CNAME*: Canonical Name - tên biệt danh trỏ từ domain này sang domain khác.
- *MX*: Mail Servers - các server chịu trách nhiệm xử lý email cho domain.
- *SOA*: Start of Authority - name server chính, email quản trị viên và zone serial number.
- *TXT*: Text Records - văn bản tùy ý, thường dùng cho SPF, DKIM, DMARC và xác minh domain.
>*phần này là cái cho vào -type =... trong câu lệnh trên*

Ví dụ (địa chỉ IPv4 qua resolver của Cloudflare):
```
user@TryHackMe$ nslookup -type=A tryhackme.com 1.1.1.1
Server:         1.1.1.1
Address:        1.1.1.1#53
Non-authoritative answer:
Name: tryhackme.com
Address: 172.67.69.208
Name: tryhackme.com
Address: 104.26.11.229
Name: tryhackme.com
Address: 104.26.10.229
```
>*có thể thấy bên trên hiển thị các ip adress khác nhau ứng với 1 domain*

Các IP này thường là anycast (trong trường hợp này là Cloudflare). Đối với penetration testing, *mỗi IP có thể host các dịch vụ khác nhau*, vì vậy hãy kiểm tra xem chúng có nằm trong scope không.

Ví dụ về MX (mail server):
```
user@TryHackMe$ nslookup -type=MX tryhackme.com
Server:         127.0.0.53
Address:        127.0.0.53#53
Non-authoritative answer:
tryhackme.com mail exchanger = 1 aspmx.l.google.com.
tryhackme.com mail exchanger = 5 alt1.aspmx.l.google.com.
tryhackme.com mail exchanger = 5 alt2.aspmx.l.google.com.
tryhackme.com mail exchanger = 10 alt3.aspmx.l.google.com.
tryhackme.com mail exchanger = 10 alt4.aspmx.l.google.com.
....
```
*=> liệt kê các mailsever phục vụ domain này*
Số đứng trước mỗi server thể hiện độ ưu tiên: giá trị càng nhỏ thì độ ưu tiên càng cao. Trong trường hợp này, Google Workspace xử lý email, điều này khá phổ biến đối với nhiều tổ chức và thường được vá lỗi tốt (well-patched).
### -dig
dig là công cụ truy vấn DNS hiện đại và được ưu tiên hơn.
Cú pháp: *dig [@SERVER] DOMAIN_NAME [TYPE]*
Ví dụ (MX record qua Cloudflare):
```
user@TryHackMe$ dig @1.1.1.1 tryhackme.com MX
; <<>> DiG 9.18.28-0ubuntu0.22.04.1-Ubuntu <<>> @1.1.1.1 tryhackme.com MX
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12345
;; flags: qr rd ra; QUERY: 1, ANSWER: 5, AUTHORITY: 0, ADDITIONAL: 1
;; ANSWER SECTION:
tryhackme.com. 300 IN MX 1 aspmx.l.google.com.
tryhackme.com. 300 IN MX 5 alt1.aspmx.l.google.com.
tryhackme.com. 300 IN MX 5 alt2.aspmx.l.google.com.
tryhackme.com. 300 IN MX 10 alt3.aspmx.l.google.com.
tryhackme.com. 300 IN MX 10 alt4.aspmx.l.google.com.
;; Query time: 28 msec
;; SERVER: 1.1.1.1#53(1.1.1.1) (UDP)
;; WHEN: Mon Jan 26 22:00:00 UTC 2026
;; MSG SIZE rcvd: 152
```
*==>Mẹo*  <u>Sử dụng các public resolver như 1.1.1.1</u> (hỗ trợ DNS over HTTPS và DNS over TLS) để tránh việc ISP ghi log các truy vấn của bạn.
*=> đó là lý do tại sao trên câu lệnh kia có @1.1.1.1 à ??*

Ghi chú cho defender: Giám sát các thay đổi DNS bất thường (MX record mới, TXT entry lạ). Đây có thể là dấu hiệu của subdomain takeover hoặc lỗi cấu hình.
![[Pasted image 20260808050400.png|643]]
# 5-DnsDumpster
link : https://dnsdumpster.com/
Các tra cứu tiêu chuẩn (dig/nslookup) chỉ giải mã được những tên miền bạn đã biết trước. <u>Chúng không thể phát hiện các subdomain không được công bố công khai </u>như blog.tryhackme.com, app.tryhackme.com hay dev.internal.company.com.

Subdomain rất quan trọng vì chúng thường bộc lộ những dịch vụ bị lãng quên hoặc chứa lỗ hổng (như phiên bản CMS cũ, trang quản trị của nhà phát triển), shadow IT, ứng dụng cấu hình sai, cũng như các attack surface bổ sung như API hoặc admin portal mở ra ngoài.
*==> tức là công cụ này giúp mình phát hiện các subdomain*

Trong passive recon, các subdomain này được tìm ra bằng cách khai thác nguồn OSINT công khai mà không gửi bất kỳ truy vấn nào trực tiếp đến mục tiêu.

Một công cụ miễn phí phổ biến là *DNSDumpster*. <u>Nó tổng hợp dữ liệu DNS công khai từ các nguồn như cache của search engine,</u> cơ sở dữ liệu zone transfer và bản ghi certificate. Công cụ này không thực hiện brute-force enumeration, nên hoạt động hoàn toàn passive. Kết quả trả về gồm danh sách subdomain và host, IP kèm vị trí địa lý (geolocation), các bản ghi MX, TXT, CNAME, cùng sơ đồ trực quan thể hiện mối quan hệ giữa chúng.
*=> na ná với gobuster, nhưng gobusster nó bruteforce để enumeration, còn thằng này thì nó là dạng thu thập thông tin trên interner-> không đụng chạm tới server*
Khi tìm kiếm tryhackme.com trên DNSDumpster, bạn sẽ thấy các kết quả như blog.tryhackme.com mà các tra cứu DNS cơ bản thường bỏ sót.
![[Pasted image 20260808051813.png]]
DNSDumpster cũng biểu diễn dữ liệu dưới dạng đồ thị, hiển thị mối liên hệ giữa subdomain, IP và mail server.
![[Pasted image 20260808051830.png]]
### -Certificate Transparency (CT) Logs
link : crt.sh *đ vào được =))*
Phương pháp phát hiện subdomain thụ động hiệu quả nhất hiện nay là sử dụng Certificate Transparency logs, thông qua trang crt.sh.

Certificate Transparency là một framework ghi log công khai (bắt buộc áp dụng từ khoảng năm 2015), lưu lại mọi SSL/TLS certificate được cấp bởi các Certificate Authority. Mỗi certificate chứa trường Subject Alternative Name (SAN) liệt kê tất cả domain và subdomain mà nó bảo vệ. Bằng cách tra cứu các log này, bạn có thể phát hiện subdomain mà không tạo ra bất kỳ lưu lượng mạng nào tới mục tiêu.

Để sử dụng crt.sh, truy cập [https://crt.sh](https://crt.sh) và tìm kiếm với từ khóa %.tryhackme.com. Ký tự wildcard % đại diện cho bất kỳ chuỗi subdomain nào. Kết quả sẽ liệt kê toàn bộ certificate từng được cấp cho các subdomain của tryhackme.com, thường mang lại kết quả gấp 10 đến 100 lần so với DNSDumpster.

crt.sh hoàn toàn passive, cập nhật theo thời gian thực và không giới hạn lượt truy vấn (rate limit) đối với nhu cầu dùng cơ bản.

Một số giải pháp phát hiện subdomain passive khác gồm SecurityTrails (cho phép tìm kiếm miễn phí có giới hạn) hoặc các công cụ command-line như Subfinder để tổng hợp dữ liệu từ nhiều nguồn passive cùng lúc.

Góc nhìn cho defender: Các tổ chức cần chủ động theo dõi CT log và danh sách subdomain để kịp thời phát hiện các dangling record (vốn dẫn đến nguy cơ subdomain takeover) hoặc các subdomain được khởi tạo trái phép.

# 6-shodan
Trong passive reconnaissance, các công cụ như Shodan.io cho phép bạn thu thập thông tin tình báo về các internet-facing asset của mục tiêu mà không gửi bất kỳ traffic nào đến chúng.
[[sodan]]
*Shodan là* *một search engine dành cho các thiết bị kết nối internet*. *Nó liên tục quét public internet, thu thập banner và response từ các open port cũng như service, sau đó lập chỉ mục (index) để phục vụ tìm kiếm.* Khác với Google vốn lập chỉ mục cho các trang web, Shodan tập trung vào thiết bị: server, thiết bị mạng, camera, router, industrial control system và nhiều thiết bị khác.

Giá trị phòng thủ: Các tổ chức theo dõi Shodan (qua cảnh báo hoặc kiểm tra thủ công) để phát hiện những sự bộc lộ ngoài ý muốn như rogue server, máy chủ kiểm thử bị lãng quên hoặc các service chứa lỗ hổng.
![[Pasted image 20260808053941.png]]
**Sử dụng giao diện Shodan**
Để bắt đầu, truy cập [https://www.shodan.io](https://www.shodan.io). Các tìm kiếm cơ bản không yêu cầu tài khoản. Nhập domain name (ví dụ: tryhackme.com) hoặc địa chỉ IP thu được từ các bước tra cứu DNS trước đó (ví dụ: 104.26.10.229) vào thanh tìm kiếm.
Trang kết quả hiển thị danh sách các host phù hợp. Chọn một host sẽ mở ra giao diện chi tiết chứa các thông tin sau:
- Địa chỉ IP và ASN (Autonomous System Number): xác định dải mạng (network block).
- Hosting provider/organisation (ví dụ: Cloudflare, AWS): tiết lộ hạ tầng đằng sau domain.
- Vị trí địa lý (quốc gia, thành phố): vị trí vật lý tương đối của server.
- Open ports và services: đi kèm version string và banner (ví dụ: loại và phiên bản HTTP server).
- Tags: như cdn hoặc vuln nếu có lỗ hổng đã biết khớp với phiên bản service được phát hiện.

**Mẹo tìm kiếm**
Shodan hỗ trợ nhiều bộ lọc tìm kiếm để thu hẹp kết quả:
- hostname:tryhackme.com lọc theo hostname cụ thể.
- org:"TryHackMe" lọc theo tên tổ chức (organisation).
- port:443 country:US lọc theo port và quốc gia.
- http.component:"wordpress" xác định tech stack (nếu bị bộc lộ).
Để xem hướng dẫn đầy đủ, truy cập: [https://help.shodan.io/the-basics/search-query-fundamentals](https://help.shodan.io/the-basics/search-query-fundamentals)
Để tìm hiểu thêm, Censys.io (cho phép tìm kiếm cơ bản miễn phí) cung cấp dữ liệu host và certificate tương tự. Nó có thể đóng vai trò bổ trợ hữu ích khi đối chiếu chéo (cross-reference) kết quả.
Bài tập
1. Truy cập [https://www.shodan.io](https://www.shodan.io) (không cần tài khoản cho các tìm kiếm cơ bản).
2. Tìm kiếm một trong các IP của tryhackme.com (từ Task 4, ví dụ: 104.26.10.229) hoặc dùng cú pháp hostname:tryhackme.com.
3. Khám phá kết quả: chú ý nhà cung cấp (Cloudflare), vị trí (US), các open port (phổ biến là 443/HTTPS) và banner.
4. Sử dụng Shodan để trả lời các câu hỏi bên dưới. Tất cả câu trả lời đều có thể nhìn thấy mà không cần tài khoản premium.