# Bug Bounty Notes (Ghi Chép Bug Bounty)

## Recon (Thu Thập Thông Tin)

- Tìm kiếm tất cả các subdomain, kể cả các subdomain không còn hoạt động (inactive).
- Luôn quét toàn bộ các cổng (scan all ports).

### Các công cụ cần thiết lập:
- `massDns`: Phục vụ brute-force subdomain tốc độ cao.
- `Intrigue.io`: Phục vụ thu thập thông tin mục tiêu (recon).

### Việc cần làm - Script tùy chỉnh (Custom Scripts):
1. Xây dựng script Python tự động tìm kiếm trên GitHub và trả về danh sách các kết quả tìm kiếm có ít nhất 1 kết quả - **ĐÃ HOÀN THÀNH!**


### Quy trình Recon (Recon Methodology):

#### 1. Thu thập thông tin mục tiêu, bao gồm các thương vụ Mua lại / Sáp nhập (Acquisitions)
- [crunchbase.com](https://www.crunchbase.com) (Tìm các công ty con/thương vụ mua lại)

#### 2. Xác định dải mạng ASN (Autonomous System Numbers)
- **Thủ công:** [bgp.he.net](http://bgp.he.net)
- **Tự động hóa:**
  - `Metabigor`
  - `ASNLookup`
  - `amass intel -asn [ASN NUMBER]`

#### 3. Tìm kiếm tên miền gốc / hạt giống (Seeds / Root Domains)
- **Reverse WHOIS** (Tự động hóa: `DOMLink`)
- **Mối quan hệ Quảng cáo / Phân tích (Ad/Analytics Relationships):** [builtwith.com](https://builtwith.com) -> Relationships
- **Google Dorks**
- **Shodan**

#### 4. Tìm kiếm Subdomain (**FRAMEWORK BẮT ĐẦU TỪ BƯỚC NÀY**)

##### a. Khai thác liên kết và file JavaScript (Linked and JS Discovery)
- Trích xuất toàn bộ các đường dẫn/link nằm trong mã nguồn client-side của ứng dụng.
- **Với Burp Suite Pro:**
  1. Tắt chế độ quét thụ động (passive scanning) để tối ưu hiệu năng.

**Việc cần làm (To Do):**
- Cron Job - 6 giờ - `kindling.py` - Hoàn thành!
- Cron Job - 24 giờ - `fire_starter.py` - Hoàn thành!
- Cron Job - 1 tuần - `firewood.py` - Hoàn thành!

1. Bổ sung các port mở rộng vào `kindling.py` (`httprobe`) - Hoàn thành!
2. Hiển thị dữ liệu giá trị lên Dashboard - Hoàn thành!
3. Module quét dải IP Cloud (Cloud Ranges Module) - Hoàn thành!
4. Module tạo wordlist tùy chỉnh (Custom Wordlist Module) - Hoàn thành!
5. Module thu thập thông tin Ứng dụng / Máy chủ (Application / Server Info Modules)
6. Sửa lỗi module `Subdomainizer`

##### b. Thu thập thông tin Subdomain từ nguồn dữ liệu ngoài (Subdomain Scraping)
- Cào (scrape) thông tin tên miền từ nhiều nguồn để nhận diện các URL/domain được lưu trong cơ sở dữ liệu của họ:
  - **Nguồn hạ tầng mạng:** Censys, DnsDumpster, WaybackMachine
  - **Nguồn chứng chỉ SSL/TLS (Certificate Transparency):** crt.sh, CertDB, Cert Spotter
  - **Công cụ tìm kiếm:** Google, Yahoo, Baidu
  - **Nguồn dữ liệu bảo mật:** VirusTotal, Rapid7 Project Sonar, SecurityTrails
- **Công cụ:** `Amass` và `Subfinder`
  *(Amass sẽ trả về số ASN. Nếu có ASN mới, quay lại Bước 2 để chạy lại)*
- `github-search` -> `github-subdomains.py` (chạy 5 lần: 4 lần sleep 6 giây và 1 lần sleep 10 giây để tránh rate-limit)
- `shosubgo` (Công cụ phân tích Shodan)
- **Dải IP Cloud (Cloud Ranges):** Kỹ thuật cực kỳ giá trị là giám sát toàn bộ các dải IP của AWS, GCP và Azure cho các website chạy SSL và phân tích chứng chỉ để khớp với mục tiêu (quét port 443).
  *(Bài viết của Daehee Park, bài thuyết trình Defcon của Sam Erb: `tls.bufferover.run/dns?q=[SEARCH_TERM]`)*

##### c. Brute-force Subdomain (Dò quét subdomain hoạt động)
- `amass enum -brute -d [DOMAIN] -rf`
- `shuffleDNS` (bộ bọc - wrapper cho massDNS)
- **Wordlists:**
  - *Wordlist tùy chỉnh riêng (Tailored Wordlists):*
    - TomNomNom
    - Cewl
  - *Wordlist dung lượng lớn (Massive Wordlists):*
    - `all.txt` (JHaddix)
  - [Assetnote Commonspeak2](https://github.com/assetnote/commonspeak2)
- **Biến thể Subdomain (Subdomain Alterations):** (Ví dụ: `www.target.com` -> `ww2.target.com`)

#### 5. Phân tích Cổng (Port Analysis)
- `masscan`: Công cụ siêu nhanh để xác định các cổng mở (cần danh sách IP!) ([Hướng dẫn](https://danielmiessler.com/study/masscan/))
- `dnmasscan`: Bộ bọc giúp phân giải tên miền sang IP rồi chuyển tiếp danh sách IP vào masscan.
- `nmap`: Quét sâu chi tiết các cổng mở vừa phát hiện.
- `Brutespray`: Tự động kiểm tra thông tin đăng nhập mặc định (default creds).

#### 6. GitHub Dorking (Chạy trong lúc chờ quá trình quét subdomain tự động)
- Tìm kiếm các endpoint và subdomain bị lộ.
- Tạo wordlist riêng cho từng mục tiêu dựa trên các công nghệ được phát hiện.
- Hầu hết các tổ chức đều dùng cấu trúc thư mục tương tự nhau trên các trang của họ. Nếu tìm thấy một subdomain đã bị vô hiệu hóa trên GitHub, hãy xây dựng wordlist từ chính cấu trúc thư mục đó.
- Quy ước đặt tên và các mẫu (patterns) là cực kỳ quan trọng!
- Tận dụng thông tin tuyển dụng (Job Postings) của tổ chức để xác định Tech Stack (công nghệ sử dụng).
- Sau khi tìm kiếm, lọc mục "Languages" để tìm các ngôn ngữ scripting:
  - Tìm theo cú pháp ngôn ngữ (Ví dụ: `language:python language:bash`).
- Kiểm tra các repository mới được commit gần đây.
- Đảm bảo repo thực sự thuộc về hoặc liên quan đến công ty mục tiêu.
- Loại bỏ các kết quả không cần thiết bằng từ khóa `NOT` (Ví dụ: `"teslamotors" NOT owner-api.teslamotors.com`).
- Nhận diện các nhân viên làm việc tại tổ chức nhưng không nằm trong danh sách thành viên repo chính của tổ chức (đối chiếu trên LinkedIn để xác nhận).
- Tìm kiếm các tài khoản nhân viên của tổ chức nhưng không được liệt kê chính thức bằng các truy vấn:
  - `"[ORG]" dotfiles`
- Khi làm thủ công, việc tìm ra các nhân viên không được liên kết công khai với tổ chức là quan trọng nhất!

#### 7. Kiểm tra HTTP Probe & Chụp màn hình
- `httprobe` -> `Eyewitness` (hoặc `gowitness`/`aquatone`)

#### 8. Chiếm quyền kiểm soát Subdomain (Subdomain Takeover)
- [EdOverflow/can-i-take-over-xyz](https://github.com/EdOverflow/can-i-take-over-xyz) (Cơ sở dữ liệu định nghĩa và dấu hiệu nhận biết SDT)
- `SubOver`
- `nuclei` (với các template subdomain-takeover)
