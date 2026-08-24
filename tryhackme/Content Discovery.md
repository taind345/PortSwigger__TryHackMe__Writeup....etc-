# 1-Intro
*phần này liên kết với 2 room* [[Active Reconnaissance]] và [[Passive Reconnaissance]]
Trong bảo mật ứng dụng web,<u> nội dung (content) đề cập đến bất kỳ thành phần nào được lưu trữ trên web server</u>, chẳng hạn như các<u> trang, file, thư mục, trang quản trị (admin portal),</u> file cấu hình và các bản sao lưu (backup archive). Phát hiện nội dung (Content discovery) là quá trình <u>tìm kiếm những nội dung vốn không được chủ đích công khai</u> hoặc không có liên kết rõ ràng.

Nội dung này có thể là cổng thông tin dành cho nhân viên, các phiên bản cũ của website, file sao lưu bị rò rỉ hoặc các bảng điều khiển quản trị (admin panel). Tìm ra chúng là một phần cốt lõi trong mọi bài kiểm thử xâm nhập (penetration test) ứng dụng web.
*=> tức là tìm endpoint à ?*
Có *ba phương pháp chính để phát hiện nội dung*: thủ công (*manual*), tự động (*automated*) và *OSINT* (Tình báo nguồn mở). Phòng học này sẽ bao quát cả ba phương pháp.

### Mục tiêu bài học

Hoàn thành phòng học này, bạn sẽ có thể:

* Phát hiện nội dung ẩn thủ công bằng cách khai thác `robots.txt`, `sitemap.xml`, favicon, HTTP header và phân tích framework.
* Sử dụng các công cụ OSINT bao gồm Google dorking, Wappalyzer, Wayback Machine, GitHub và thu thập thông tin S3 bucket.*=>*[[Passive Reconnaissance]]
* Sử dụng Gobuster để brute-force thư mục, tên miền phụ (subdomain) và virtual host.*=>* [[Active Reconnaissance]]
* Áp dụng phương pháp luận phát hiện nội dung có cấu trúc vào bài kiểm thử xâm nhập.

### Yêu cầu tiên quyết
Bạn nên hoàn thành và nắm vững kiến thức từ các phòng học sau trước khi bắt đầu:
* HTTP in Detail
* How Websites Work
# 2-Manual discovery - Common file
Một số file mà web server công khai theo quy ước có thể tiết lộ nhiều thông tin hơn dự kiến. Kiểm tra các file này thủ công nên là bước đầu tiên trong bất kỳ quá trình phát hiện nội dung (content discovery) nào.
*=> các file này là gì ??*
### robots.txt

File robots.txt <u>hướng dẫn</u> *search engine crawler* <u>những trang nào được phép lập chỉ mục</u> (index). <u>Chủ trang web thường liệt kê các thư mục nhạy cảm ở đây để tránh chúng xuất hiện trong kết quả tìm kiếm</u>, vô tình tạo ra một danh sách các vị trí tiềm năng sẵn có cho penetration tester.

Xem file robots.txt trên trang web Acme IT Support bằng cách mở Firefox trên AttackBox và truy cập địa chỉ http://MACHINE_IP/robots.txt (URL này sẽ cập nhật sau 2 phút kể từ khi bạn khởi chạy máy).
![[Pasted image 20260812142848.png]]
<u>File robots.txt này chỉ dẫn các web crawler (như công cụ tìm kiếm) cách tương tác với trang web</u>. Nó cho phép tất cả các bot truy cập hầu hết trang web (Allow: /) nhưng yêu cầu chúng không truy cập vào /staff-portal. Hãy lưu ý, <u>đây chỉ là hướng dẫn dành cho bot chứ không phải biện pháp bảo mật,</u> <u>vì vậy các đường dẫn bị hạn chế vẫn có thể truy cập được nếu bạn truy cập trực tiếp.</u>
*=> tức là nó khiến cho các công cụ tự động của search engine ko auto đánh chỉ mục, và ko để bị crawler của search engine quét*
*=> đọc thêm ở đây* https://tuoitre.vn/anthropic-xu-ly-su-co-khien-doan-chat-claude-xuat-hien-tren-google-100260730104947264.htm
### sitemap.xml
Khác với robots.txt (vốn hạn chế crawler), <u>sitemap.xml cho các công cụ tìm kiếm biết những trang nào mà chủ sở hữu muốn hiển thị</u>. Các file này đôi khi chứa các trang chạy thử nghiệm (staging page), nội dung cũ hoặc các URL khó truy cập thông qua cách duyệt web thông thường. Kiểm tra file này tại địa chỉ http://MACHINE_IP/sitemap.xml.
![[Pasted image 20260812144052.png]]
Như hiển thị trong hình, <u>sitemap này liệt kê các endpoint cụ thể trên ứng dụng mục tiêu, bao gồm các trang tiêu chuẩn như /news, /contact và nhiều ID bài viết (/news/article?id=1,2,3).</u> Quan trọng hơn, nó tiết lộ các đường dẫn nhạy cảm như /customers/login và /s3cr3t-area, những nơi khó phát hiện qua cách duyệt web thông thường. Sự xuất hiện của các tham số như id= cũng gợi ý các vị trí đầu vào tiềm năng đáng để thử nghiệm. Điều này biến sitemap trở thành nguồn thông tin giá trị trong giai đoạn trinh sát (reconnaissance) nhằm vẽ bản đồ bề mặt tấn công (attack surface).
### đặt câu hỏi 
*giải thích trực quan hơn*
![[Pasted image 20260812143636.png]]
*-2 cái file này trang web nào cũng có , và tên cố định như này*![[Pasted image 20260812143528.png]]
### thực hành
-để vào *robots.txt* ta đơn giản là truy cập dựa trên url thôi![[Pasted image 20260812145047.png]]
-để vào *sitemap.xml*, thì cũng tương tự , ta đi vào url 
![[Pasted image 20260812145509.png]]-để cho dễ nhìn, ta nhấn ctr+u
![[Pasted image 20260812145717.png]]*=> các vùng được khoanh là các enpoint mà dev cho phép trình duyệt hiển thị, đánh chỉ mục và đề xuất trên công cụ tìm kiếm*

# 3-Manual discovery - Header and framework stack
### Headers
Khi web server phản hồi một yêu cầu, nó bao gồm các header có thể tiết lộ nhiều chi tiết kỹ thuật hữu ích. Các header như Server và X-Powered-By thường để lộ phần mềm web server cùng ngôn ngữ hoặc framework mà ứng dụng đang chạy.

Chạy lệnh sau tới web server của Acme IT Support. Tham số `-v` bật chế độ hiển thị chi tiết (verbose), bao gồm cả các response header:
![[Pasted image 20260812153257.png]]
*=> nó sẽ hiển thị HTTP response kèm theo header*
### Framework Stack
*-frame work là gì?*![[Pasted image 20260812153819.png]]
Khi đã xác định được framework (từ favicon, header, hoặc bằng cách kiểm tra comment và thông báo bản quyền trong mã nguồn trang), hãy truy cập trang web của chính framework đó để tìm hiểu thêm<u>. Trang tài liệu hướng dẫn (documentation) thường mô tả cấu trúc thư mục mặc định, đường dẫn trang quản trị và thông tin đăng nhập mặc định.</u>
*=> mỗi framework thường có các trang quản trị quy định trong document .Pentester dựa vào điều này để có thể tìm ra enpoint đăng nhập*
![[Pasted image 20260812152914.png]]
Xem mã nguồn trang của Acme IT Support tại địa chỉ `[http://10.48.142.71](http://10.48.142.71)`, bạn sẽ thấy một comment ở cuối trang chứa liên kết tới trang web của framework. Hãy mở liên kết đó, đọc phần tài liệu để tìm đường dẫn cổng quản trị (admin portal). Sau đó truy cập đường dẫn này trên trang Acme IT Support và đăng nhập bằng tài khoản mặc định `admin / admin` để lấy flag.

### thực hành
-nhìn vào phần comment trong html, ta thấy được url đi tới framework
![[Pasted image 20260812154039.png]]
-vào cái url này để đi tới framework của trang web, từ đây ta thấy được <u>enpoint cho login portal</u>![[Pasted image 20260812153607.png]]-vào đây để đăng nhập,ta được flag
![[Pasted image 20260812154145.png|504]]
# 4-OSINT - search engine and web tool
>=> phần này nói về cách dùng công cụ tìm kiếm thu thập thông tin của trang web
### Google Hacking / Dorking
Các toán tử tìm kiếm nâng cao của Google cho phép bạn lọc kết quả để làm lộ các nội dung nhạy cảm được lập chỉ mục từ trang mục tiêu. Bằng cách kết hợp các toán tử, bạn có thể tìm thấy các trang quản trị (admin panel), tài liệu bị rò rỉ, hoặc các trang đăng nhập không được chủ sở hữu công khai.
>*=>* cái này giúp mình có thể tìm kiếm những cái bị ẩn ..?

| Toán tử  | Ví dụ               | Mô tả                                                      |     |
| -------- | ------------------- | ---------------------------------------------------------- | --- |
| site     | site:tryhackme.com  | Chỉ trả về kết quả từ tên miền được chỉ định               | **  |
| inurl    | inurl:admin         | Trả về kết quả chứa từ chỉ định trong URL                  |     |
| filetype | filetype:pdf        | Trả về kết quả có định dạng file cụ thể                    |     |
| intitle  | intitle:admin       | Trả về kết quả chứa từ chỉ định trong tiêu đề trang        |     |
| intext   | intext:password     | Trả về kết quả chứa từ chỉ định trong nội dung trang       |     |
| cache    | cache:tryhackme.com | Hiển thị phiên bản lưu trong bộ nhớ đệm (cache) của Google |     |
|          |                     |                                                            |     |

Ví dụ: cú pháp `site:tryhackme.com filetype:pdf` sẽ trả về toàn bộ file PDF được lập chỉ mục từ `tryhackme.com`. Bạn có thể kết hợp nhiều bộ lọc trong cùng một truy vấn.
>*=>*
### Wappalyzer
Wappalyzer là một tiện ích mở rộng trên trình duyệt và công cụ trực tuyến giúp xác định các công nghệ mà website đang sử dụng, bao gồm framework, nền tảng, CDN, công cụ phân tích, cổng thanh toán... Nó thường có thể phát hiện cả số phiên bản, giúp ích rất nhiều cho việc tìm kiếm các lỗ hổng đã biết. Bạn chỉ cần cài đặt từ cửa hàng tiện ích của trình duyệt và truy cập trang web bất kỳ để xem toàn bộ danh mục công nghệ (tech stack) ngay lập tức.
## thực hành

- site![[Pasted image 20260813045206.png|607]]
- inurl![[Pasted image 20260813045719.png]]
- tìm các trang web có chứa headeing là "messi"![[Pasted image 20260813050221.png|499]]
# 5-OSINT -Repo and Archives
### Wayback Machine

Wayback Machine là kho lưu trữ kỹ thuật số của Internet từ cuối những năm 1990. Khi tìm kiếm một tên miền, bạn sẽ thấy toàn bộ các  (snapshot) được ghi lại theo thời gian. Công cụ này rất hữu ích để tìm các trang đã bị xóa khỏi website thực tế nhưng có thể vẫn còn tồn tại: biểu mẫu đăng nhập cũ, các endpoint bị lãng quên, hoặc nội dung từng được đăng tải ngắn hạn trước khi bị gỡ xuống.
### GitHub

GitHub là nền tảng lưu trữ đám mây phổ biến nhất cho các kho mã nguồn Git (repository). Lập trình viên đôi khi vô tình commit các dữ liệu nhạy cảm như API key, thông tin đăng nhập, file cấu hình và file .env trước khi nhận ra repository đang ở chế độ công khai.

Hãy tìm kiếm trên GitHub theo tên công ty hoặc tên miền của mục tiêu. Khi tìm thấy repository liên quan, hãy kiểm tra kỹ toàn bộ lịch sử commit chứ không chỉ xem các file ở phiên bản hiện tại. Dữ liệu nhạy cảm thường được xóa ở các commit sau, nhưng dấu vết vẫn còn lưu lại trong lịch sử.

### S3 Buckets
Amazon S3 (Simple Storage Service) là nền tảng lưu trữ đám mây được nhiều tổ chức sử dụng để chứa file và nội dung website tĩnh. Định dạng URL tiêu chuẩn cho một S3 bucket là `https://{name}.s3.amazonaws.com`. Chủ sở hữu sẽ phân quyền cho bucket, nhưng lỗi cấu hình sai (misconfiguration) xảy ra rất phổ biến: một bucket bị mở công khai có thể làm lộ nhiều file nhạy cảm.

Các cấu trúc đặt tên phổ biến gồm: `{company}-assets`, `{company}-backup`, `{company}-www`, và `{company}-dev`. Bạn có thể thử kết hợp các mẫu này với tên công ty mục tiêu, hoặc tìm URL của S3 bucket trong mã nguồn trang web và các repository GitHub.

# 6-Gobuster fundalmental
Các kỹ thuật manual và OSINT cũng chỉ giúp bạn đi được một giới hạn nhất định. Automated discovery sử dụng các công cụ chuyên dụng để bắn hàng trăm hoặc hàng nghìn request đến web server trong thời gian ngắn, qua đó kiểm tra xem các directory, file hay tài nguyên nào thực sự tồn tại. Kỹ thuật này hoạt động dựa trên các wordlist — file text chứa danh sách tên directory, file và path thông dụng.

Gobuster là một công cụ enumeration mã nguồn mở viết bằng Go. Tool hỗ trợ nhiều mode: directory/file enumeration (dir), DNS subdomain enumeration (dns), và virtual host enumeration (vhost). Nó đã được cài sẵn trên AttackBox và mặc định có trong Kali Linux.

Chạy lệnh `gobuster --help` để xem toàn bộ danh sách command và global flag:

| Flag            | Description                                                                        |
| --------------- | ---------------------------------------------------------------------------------- |
| -t / --threads  | Số thread chạy đồng thời (mặc định: 10). Tăng lên để scan nhanh hơn.               |
| -w / --wordlist | Đường dẫn tới file wordlist. Bắt buộc cho mọi mode.                                |
| -o / --output   | Xuất kết quả ra file thay vì in ra stdout.                                         |
| --delay         | Thời gian delay giữa các request: hữu ích khi bypass cơ chế rate limit của server. |

### Wordlists

Một wordlist tốt đóng vai trò mang tính quyết định. SecLists là bộ sưu tập wordlist chuẩn mực nhất hiện nay, được đặt sẵn trên AttackBox tại `/usr/share/wordlists/SecLists/`. Đối với tác vụ directory enumeration, hai file `Discovery/Web-Content/common.txt` và `Discovery/Web-Content/directory-list-2.3-medium.txt` là đủ đáp ứng hầu hết mọi kịch bản.
>ko hiểu sao máy mình ko có cái seclisst này mà phải cài ngoài, nên thư mục nó cũng hơi khác
### dir Mode

Mode `dir` dùng để brute-force directory và file trên web server. Cú pháp cơ bản:

```bash
gobuster dir -u "http://MACHINE_IP" -w /path/to/wordlist

```

Flag `-u` chỉ định target URL cần quét. Flag `-w` chỉ định file wordlist chứa danh sách directory và file name để Gobuster lần lượt thử nghiệm trên target. Cả `-u` và `-w` đều là tham số bắt buộc; nếu thiếu một trong hai thì tool sẽ báo lỗi.

Một số flag bổ sung hữu ích cho mode `dir`:

| Flag                     | Description                                                         |
| ------------------------ | ------------------------------------------------------------------- |
| -x / --extensions        | Extension của file cần tìm (ví dụ: -x .php,.txt,.js)                |
| -r / --followredirect    | Tự động đi theo HTTP redirect                                       |
| -k / --no-tls-validation | Bỏ qua việc verify TLS certificate (rất tiện trong môi trường lab)  |
| -s / --status-codes      | Chỉ lọc và hiển thị các HTTP status code cụ thể (ví dụ: -s 200,301) |

Chạy lệnh sau tới web server Acme IT Support và xem kết quả:
![[Pasted image 20260815134401.png]]

> *=>* đại khái là nhớ 
> 	-w 
> 	-u 
> 	dir
> 	và mấy thằng còn lại trong bảng

### thực hành
câu lệnh mình thực hiện là 
``` shell
gobuster dir -u  http://10.49.164.235 -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt
```
![[Pasted image 20260815140045.png]]

# 7-cách kiểm tra các Subdoamain với Gobuster
Chế độ tiếp theo chúng ta tìm hiểu là **dns** và **vhost mode**. Chế độ dns cho phép Gobuster brute-force các subdomain. Trong một bài penetration test, việc kiểm tra các subdomain thuộc domain chính của mục tiêu là điều thiết yếu. Một lỗ hổng được vá trên domain chính chưa chắc đã được vá trên subdomain, do đó cơ hội khai thác có thể nằm ở các subdomain này.
>có thể dùng DNSsumpster để passive recon

Ví dụ, nếu TryHackMe sở hữu `tryhackme.thm` và `mobile.tryhackme.thm`, có thể tồn tại lỗ hổng trên `mobile.tryhackme.thm` mà trang chính `tryhackme.thm` không có. Đó là lý do tại sao việc dò quét subdomain lại quan trọng.
### Subdomains vs Virtual Hosts
Cần phân biệt rõ hai khái niệm này trước khi dùng Gobuster để enumerate:
* **Subdomain**: Được phân giải thông qua hệ thống DNS. Ví dụ: `blog.example.thm` là một bản ghi DNS trỏ tới một địa chỉ IP. *=> nó là tên miền phụ, một thành phần trong cái hệ thống gồm nhiều dịch vụ con*
* **Virtual host** (vhost): Được phân giải bởi web server. Nhiều trang web khác nhau có thể cùng chạy trên một địa chỉ IP; web server sẽ đọc header `Host:` trong HTTP request để quyết định trả về nội dung của trang nào.
*=> tức là 1 IP nó có nhiều domain cùng trỏ tới, vhost nó là các tên miền trỏ tới ip đó*
![[Pasted image 20260815144831.png]]
Gobuster tách biệt hai nhiệm vụ này thành hai chế độ: `dns` dành cho subdomain và `vhost` dành cho virtual host.
> cái **vhost** thì mình đã gặp nó trong loadbalancing rồi
### Chuẩn bị môi trường
>Phần này đơn thuần là chuẩn bị môi trường cho lab thôi:
>-file resolve có tác dụng khai báo cái tên miền example.com
>-file dnsmasq giúp mình ánh địa chỉ ip với example.com

Trong bài thực hành này, chúng ta hoạt động trong mạng nội bộ với DNS server đặt trực tiếp trên web server. Để đảm bảo phân giải đúng các domain trong bài lab, bạn cần chỉnh sửa file `/etc/resolv-dnsmasq`:

1. Mở terminal trên AttackBox và chạy lệnh: `sudo nano /etc/resolv-dnsmasq`.
2. Chèn dòng `nameserver 10.49.164.235` lên dòng đầu tiên.
3. Lưu file bằng tổ hợp phím CTRL+O, nhấn ENTER, sau đó thoát bằng CTRL+X.
4. Chạy lệnh `/etc/init.d/dnsmasq restart` để khởi động lại dịch vụ Dnsmasq.

Nội dung file sau khi sửa:

```text
nameserver 10.49.164.235
nameserver 169.254.169.253
```

 **Cập nhật file Hosts**
Để domain trong bài lab phân giải chính xác, chúng ta cần ánh xạ thủ công domain với target IP thông qua file `/etc/hosts`:

1. Mở terminal trên AttackBox và chạy: `sudo nano /etc/hosts`.
2. Thêm dòng sau vào cuối file: `10.49.164.235 example.thm`.
3. Lưu file (CTRL+O, ENTER) và thoát (CTRL+X).
4. Kiểm tra lại bằng lệnh: `ping example.thm`.

Nội dung file sau khi thêm:

```text
127.0.0.1	localhost
127.0.0.1   vnc.tryhackme.tech
127.0.1.1	tryhackme.lan	tryhackme
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
10.49.164.235 example.thm

```

### dns Mode
Chế độ dns thực hiện DNS lookups bằng cách lấy các mục trong wordlist làm subdomain thử nghiệm. H<u>ai flag bắt buộc là `-d` (domain) và `-w` (wordlist). </u>Tùy chọn `--wildcard` trong Gobuster dùng để ép tool tiếp tục brute-force ngay cả khi phát hiện cấu hình Wildcard DNS, giúp không bỏ sót kết quả dù có thể xuất hiện một số trường hợp dương tính giả (false positive).
> Cái này tương tự với -u thôi, -u là url còn -d là domain

```bash
gobuster dns -d example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --wildcard

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Domain:            example.thm
[+] Threads:           10
[+] Wildcard forced:   true
[+] Timeout:           1s
[+] Wordlist:          /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
===============================================================
Starting gobuster in DNS enumeration mode
===============================================================
Found: shop.example.thm
Found: www.shop.example.thm
Found: webdisk.shop.example.thm
Found: autodiscover.shop.example.thm
Found: autoconfig.shop.example.thm
Found: academy.example.thm
Found: primary.example.thm

Progress: 4997 / 4998 (99.98%)
===============================================================
Finished
===============================================================

```

<u>Các flag hữu ích trong chế độ dns:</u>
* `-d / --domain`: Domain mục tiêu cần dò quét.
* `-i / --show-ips`: Hiển thị địa chỉ IP tương ứng mà subdomain phân giải về.
* `-r / --resolver`: Chỉ định DNS server tùy chỉnh để thực hiện tra cứu.

### vhost Mode
Chế độ vhost không sử dụng DNS. Thay vào đó, <u>tool gửi các HTTP request trực tiếp đến IP mục tiêu và lần lượt thay đổi giá trị trong header</u> `Host:` dựa theo danh sách trong wordlist. Kỹ thuật này giúp phát hiện các virtual host nội bộ không được khai báo trên public DNS.

Chạy lệnh quét vhost. Flag `--append-domain` ghép từng từ khóa trong wordlist với domain chỉ định, và `--exclude-length` giúp lọc bỏ các phản hồi có cùng kích thước (thường là trang mặc định hoặc lỗi để tránh nhiễu kết quả):

```bash
gobuster vhost -u "http://10.49.164.235" --domain example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain --exclude-length 250-320

```

Kiểm tra kết quả và xác định các virtual host phản hồi với status code 200 OK, sau đó truy cập từng trang qua trình duyệt để khám phá nội dung bên trong.

Ở output mẫu, quá trình enumeration virtual host đã được thực hiện kèm bộ lọc kích thước phản hồi để giảm nhiễu. Việc không phát hiện thêm virtual host hợp lệ nào cho thấy ứng dụng này có thể không cấu hình thêm subdomain dạng vhost, do đó bước kiểm thử tiếp theo nên tập trung vào domain chính và các directory đã tìm thấy trước đó.

# 8- tổng kết
![[Pasted image 20260815143647.png]]