# 1- tổng quan
Phần này chủ yếu mở rộng và học thêm các kiến thức đã được học của các lab trước
keyword cần nắm
``` text
1. Đồ nghề trinh sát (Tools)
* curl (-s, -I)
* DevTools (F12 - tab Network)
* Gobuster / ffuf
* Nikto (-nointeractive, -Tuning)

1. Nhận diện & Headers (Fingerprinting)
* Server
* X-Powered-By
* X-Frame-Options (Clickjacking)
* Content-Security-Policy / CSP
* X-Content-Type-Options (nosniff)
* Strict-Transport-Security / HSTS

1. Lỗi cấu hình kinh điển (Misconfigurations)
* Directory Listing / Indexing (Lộ danh sách file)
* Verbose Errors / Stack Trace (Phun mã lỗi chi tiết)
* Debug Endpoints (Lộ API)
* Information Disclosure (Lộ thông tin)
* Exposed Dotfiles / Backups (.env, .bak, .zip)
* Hardcoded secrets (Lộ key trong file tĩnh)

1. Bắt bệnh từng thằng Server
* Apache: Options +Indexes, mod_status (/server-status), .htpasswd, ServerTokens.
* Python HTTP: python3 -m http.server 8000.
* Node.js/Express: NODE_ENV=development, custom error handler, app._router.stack (/api/routes), process.env, express.static().
* Nginx: server_tokens, autoindex, stub_status (/nginx_status).
```


# 2- Xác định websever
Trước khi bắt đầu dò thư mục hay kiểm thử input, bạn phải biết mình đang đối mặt với mục tiêu gì. Phần mềm web server sẽ quyết định xem hệ thống có thể dính những lỗi cấu hình nào, đường dẫn nào đáng để kiểm tra, và dùng công cụ gì thì mới hiệu quả. Việc nhận diện server không phải làm cho có, mà nó ảnh hưởng trực tiếp đến mọi bước tiếp theo của bạn.

Tin vui cho mày là đa số mấy con web server nó toàn tự khai tên bô bô ra. Nhiều dev và admin hay giữ nguyên cấu hình mặc định, mà cấu hình mặc định thì lại thường để lộ rất nhiều thông tin phiên bản. Có mấy chỗ cực kỳ uy tín để mày dòm ngó đây.

### Cái Header Phản Hồi Của Server (Server Response Header)

Tín hiệu trinh sát thẳng mặt nhất chính là cái header `Server` trong phản hồi HTTP. Khi bạn gửi bất kỳ request nào đến web server, nó cũng đính kèm header này vào câu trả lời. Mỗi phần mềm server sẽ đẻ ra một cái format khác nhau, và mấy cái format này đủ chuẩn để mày dùng làm cột mốc nhận dạng luôn.

Dùng lệnh `curl`, thêm cờ `-I` để lấy riêng header mà không cần tải phần thân (body):

```bash
# -s ẩn thanh tiến trình đi
# -I gửi một cái request HEAD, chỉ nôn ra các response headers
curl -sI http://MACHINE_IP:80

```

Mày sẽ thấy nó ói ra một đống kết quả na ná như này cho con Apache đang chạy ở port 80:

```text
ubuntu@tryhackme-2404:~$ curl -sI http://MACHINE_IP:80
HTTP/1.1 200 OK
Date: Wed, 08 Apr 2026 13:59:00 GMT
Server: Apache/2.4.58 (Ubuntu)
Last-Modified: Fri, 03 Apr 2026 18:12:44 GMT
ETag: "29af-64e9243796aa2"
Accept-Ranges: bytes
Content-Length: 10671
Vary: Accept-Encoding
Content-Type: text/html

```

Header `Server` cho biết phần mềm cùng phiên bản cụ thể. Không phải server nào cũng để lộ thông tin rõ ràng thế này. Với các server được cấu hình bảo mật tốt thì nó chỉ trả về chữ `Apache` hoặc giấu hoàn toàn header, nhưng đa số cấu hình mặc định trên các máy Ubuntu thì thường để lộ ra cho bạn xem.

Đây là những gì mà từng con server trong cái bài lab này mặc định sẽ nôn ra:

| Port | Server | Default Server Header |
| --- | --- | --- |
| 80 | Apache2 | Apache/2.4.x (Ubuntu) |
| 8000 | Python HTTP Server | SimpleHTTP/0.6 Python/3.xx.x |
| 3000 | Node.js Express | None (do ứng dụng tự set) |
| 8080 | Nginx | nginx/1.xx.x |

Để ý Node.js Express không có header `Server` nào cả. Vì bản thân Express mặc định không set, và lớp Node.js HTTP bên dưới cũng không set. Dev phải tự tay thêm vào thì mới có. <u>Trên một app Express thực tế, việc không có header `Server` tự nó đã là một tín hiệu nhận biết rồi. </u>C<u>ột mốc uy tín nhất để nhận dạng thằng Express là cái header `X-Powered-By`, cái này thì Express tự động gán,</u> trừ khi dev chủ động gỡ đi.

> [!NOTE] Nhận biết server thông qua header
> - Phần này chỉ cần nhớ cái bảng trên là được
> - còn với thằng express nó không có header server-> đây cũng là một tín hiệu .Thêm nữa là header `X-Powered-By`

### Cái Header X-Powered-By
Có vài thằng framework thích thêm cái header `X-Powered-By` để khai luôn danh tính cái mảng ứng dụng đang núp bóng phía sau server. Thằng Express thì thiết lập giá trị này mặc định:
`X-Powered-By: Express`

Header này không dính dáng gì đến header `Server`, và đối với Express, đây chính là dấu vân tay chuẩn xác nhất của nó. Không giống Apache hay Nginx hiển thị rõ ở header `Server`, Express dùng `X-Powered-By` để nhận diện cội nguồn. Cứ thấy port nào mà mất header `Server` hoặc nhìn chung chung quá thì bạn phải soi ngay `X-Powered-By` này.

### Trinh sát qua DevTools của Trình duyệt

Nếu mày đang mò mẫm trên trình duyệt, thì tab Network trong DevTools hiển thị đầy đủ thông tin header không thiếu chi tiết nào mà không cần dùng tool phức tạp. Mở `http://MACHINE_IP:3000` trên Firefox, nhấp chuột phải bừa vào chỗ nào đó trên web rồi chọn Inspect (hoặc nhấn phím F12) để móc cái Developer Tools lên. Nhảy sang tab Network và f5 lại trang để hốt hết đống request. Chọn cái request chính trong danh sách, mò xuống phần Headers, soi mục Response Headers là mày thấy tuốt tuồn tuột ruột gan con server nó đáp lại những gì.
![[Pasted image 20260818144729.png]]
> [!NOTE] ????
>ơ hóa ra, vậy nãy giờ học cách recon bằng curl công cốc à. Dùng F12 là thấy rõ ràng rồi
>![[Pasted image 20260818145733.png]]
>--> nói chung là curl vẫn bá đạo, cái f12 chỉ là tầng nông thôi

### Mấy Cái Trang Báo Lỗi Mặc Định (Default Error Pages)

Cái cờ `-sI` tao bảo ở trên là gửi request HEAD, nó chỉ trả lại header thôi không có body. Muốn xem được mấy trang báo lỗi mặc định, bạn phải gửi request GET bằng cách giữ cờ `-s` mà bỏ cờ `-I` đi:

```bash
# Gửi HEAD request: chỉ lấy headers, không lấy body
curl -sI http://MACHINE_IP:PORT/

# Gửi GET request: móc luôn cả body, đầy đủ response
curl -s http://MACHINE_IP:PORT/nonexistent-page-xyz

```

Khi bạn cố tình gửi request vào một đường dẫn không tồn tại, đa số server sẽ phản hồi bằng trang lỗi 404 mặc định. Giao diện trang lỗi của từng loại server rất đặc trưng. Thằng Python thì quăng ra cái phản hồi toàn chữ suông. Thằng Nginx thì nhét phiên bản của nó dưới đáy trang HTML. Thằng Apache thì phô luôn tên nó ngay giữa body trang.

Chính những sự khác biệt này giúp bạn nhận diện được danh tính của server kể cả khi nó đã cố tình giấu header `Server`. Bạn cứ test thử qua cả 4 cổng để xem cách mỗi server xử lý và phản hồi khác nhau thế nào trong cùng một tình huống. Đọc xong thì bắt tay vào thực hành nhé!

> [!success] thực hành
> ![[Pasted image 20260818150245.png]]

# 3-python http sever
Thằng Python đẻ ra đã ngậm sẵn một cái HTTP server, dev nào cũng có thể bật lên rất nhanh. Chính vì sự tiện lợi này mà khi đi pentest rất hay gặp nó.

```bash
# This command serves the current working directory over HTTP on port 8000
python3 -m http.server 8000

```

Mấy thằng dev hay xài cái này để share file cho lẹ, test mấy trang web tĩnh, hoặc bắn data giữa 2 máy trong cùng mạng. Nhưng cái rắc rối là tính năng "share file cho lẹ" rất dễ biến thành việc "hớ hênh để lộ ra internet thời gian dài". Nó chạy trên các server public, mạng nội bộ, hay trên cloud chỉ vì ai đó mở port 8000 rồi quên mất. <u>Server này không có phân quyền, không có đăng nhập, cũng không có ghi log gì </u>ngoài mấy dòng log terminal bắt được.

> [!NOTE] Title
> Tức là luôn có một cái http server , dev bật cái này lên là tất cả thư mục và file phía backend nó lộ hết, chỉ cần truy cập vào port (ở ví dụ trên là 8000) thì mình sẽ truy cập được 

### Nó phơi ra những cái đ gì?
Cứ bật lên là nó hiển thị mọi thứ trong thư mục hiện tại ra, kể cả các file ẩn như `.env`. Không có file `.htaccess` để chặn, không có danh sách đen (blocklist), không có file cấu hình ràng buộc gì cả. Cứ có file trong thư mục là bất kỳ ai truy cập vào port 8000 cũng tải được hết. <u>Khác biệt hoàn toàn với Apache hay Nginx </u>(các web server này mặc định cấm xem danh sách thư mục và admin có thể cấu hình chặn từng đường dẫn). <u>Server Python này chỉ có đúng một chế độ: hiển thị hết cho mọi người xem.</u>

### Lộ danh sách thư mục (Directory Listing)
Nếu trong thư mục không có file `index.html`, server Python sẽ tự động tạo ra một trang HTML liệt kê mọi file bên trong. Bạn cứ truy cập thẳng vào thư mục gốc của server bằng lệnh này hoặc mở link `[http://10.48.167.138:8000](http://10.48.167.138:8000)` trên trình duyệt:
![[Pasted image 20260818154830.png]]
```bash
curl -s http://10.48.167.138:8000/

```

Mày sẽ thấy một cái danh sách với tiêu đề chà bá `Directory listing for /`. Mọi thứ trong danh sách đấy đều có thể bấm vào và tải về dễ dàng.

### Móc lốp file ẩn (Dotfiles)
Các file có dấu chấm đứng đầu như `.env` bình thường sẽ bị ẩn đi trên Linux, nhưng server Python này thì không áp dụng quy tắc đấy. Nó hiển thị ra hết như một file bình thường. File `.env` luôn là mục tiêu quan trọng vì dev hay để cấu hình hệ thống vào đấy, mà cấu hình thì kiểu gì cũng chứa mật khẩu hoặc secret key:

```text
root@attackbox:~# curl -s http://10.48.167.138:8000/.env
SECRET_KEY=dev-secret-key-do-not-use
DATABASE_URL=postgresql://webapp:S3cur3DBPass!@localhost/production
DEBUG=True

```

### Tải và soi mấy file nén (Archives)
Nếu thấy file `.zip`, `.tar.gz` hay file nén nào khác trong danh sách, tải về ngay rồi kiểm tra nội dung. Dev thỉnh thoảng để quên file backup nằm trong chính thư mục đang mở server, trong đó rất dễ có mã nguồn, bản sao database (dump), hoặc file cấu hình quan trọng.

```text
root@attackbox:~# curl -s http://10.48.167.138:8000/backup.zip -o backup.zip
root@attackbox:~# unzip backup.zip -d backup-contents/
root@attackbox:~# cat backup-contents/db_dump.sql
Archive:  backup.zip
  inflating: backup-contents/db_dump.sql
-- Database dump for staging environment

CREATE TABLE users (id INTEGER PRIMARY KEY, username VARCHAR(50));
INSERT INTO users VALUES (1, 'admin', 'admin@company.com');
INSERT INTO users VALUES (2, 'jsmith', 'jsmith@company.com');

-- End of dump

```

### Tại sao cái này lại quan trọng?
Tìm ra server Python này là một finding cực kỳ thực tế vì bạn không cần phải exploit phức tạp gì cả. Không cần lỗ hổng mã nguồn để kích hoạt. Bản thân server đang hoạt động đúng như thiết kế. <u>Điểm sơ hở ở đây là nó bị bật ở nơi không nên bật, để lộ những file không nên để lộ.</u> Khi đi làm pentest thật, viết report không phải chỉ nêu là "phát hiện một server mở", mà phải giải thích được nó đang làm lộ những gì và kẻ tấn công có thể khai thác được gì từ đống thông tin đó.

> Chú ý: Nếu trong thư mục mà có file `index.html`, thằng Python sẽ lôi cái file đó ra hiển thị thay vì hiện cái danh sách thư mục. Nếu bạn kiểm tra thư mục gốc mà không thấy danh sách file, thì thử gõ thẳng tên đường dẫn vào hoặc mò sâu xuống mấy thư mục con xem sao.


> [!NOTE] chốt lại
> - sao cái chức năng này lại thiếu an toàn như thế, mà thực tế vẫn có người dùng thật à??
> ![[Pasted image 20260818154938.png]]


> [!success] Thực hành
> ![[Pasted image 20260818155435.png]]
> - file backup bên dưới
> ![[Pasted image 20260818155507.png]]


# 4- Apache 2
Apache là web server rất phổ biến, khi đi kiểm thử kiểu gì bạn cũng sẽ gặp nó. Cấu hình mặc định trên Ubuntu của nó thường để ngỏ một số điểm yếu mà pentester hay nhắm tới. <u>3 lỗi kinh điển nhất là </u>lộ danh sách file (*directory listing*), *lộ trang server-status* và *quên file backup* ở thư mục gốc.

### Lộ hàng phiên bản (Version Disclosure)
Bắt đầu bằng trò cơ bản nhất: soi cái header Server.

```text
root@ip-10-81-64-63:~# curl -SI http://10.48.167.138:80 | grep -i server
Server: Apache/2.4.58 (Ubuntu)

```

Thằng Apache trên Ubuntu hay xài cái cấu hình ServerTokens OS mặc định, nôn mịe cả tên hệ điều hành lẫn số phiên bản ra. <u>Nắm được bản nào là mày tra CVE với soi được con server này làm được trò trống gì.</u>

> [!NOTE] Title
> dễ vậy, thằng apache này hiển thị hết luôn phiên bản trong header của nó luôn rồi

### Lộ thư mục (Directory Listing)
Nếu bạn kiểm tra một thư mục không có file index.html, chỉ thị *Options +Indexes* của Apache sẽ hiển thị danh sách file ra. <u>Dev đôi khi bật tính năng này để share file nội bộ, nhưng lại quên tắt ở những nơi chứa dữ liệu nhạy cảm.</u>
![[Pasted image 20260818160959.png]]
Mày mò vào cái link /files/ thử xem, sẽ thấy nguyên cái bảng HTML "Index of /files". Nó hiện rõ tên file, dung lượng, ngày sửa. Vào đây thì nên kiểm tra từng file một. Người ta thường để quên file excel CSV, tài liệu nội bộ hay file backup ở đây, đây đều là những dữ liệu rất có giá trị.

> [!question] ??
> tao chưa hiểu cái option+index này bật như nào, và chỉ khi bật thì cái /file này mới tồn tại à??
> ![[Pasted image 20260819072739.png]]
> ![[Pasted image 20260819072811.png]]

### Cái trang mod_status
<u>Thằng Apache có sẵn một cái trang trạng thái </u>do module *mod_status* gánh. Đáng lẽ chỉ cho localhost xem,<u> nhưng nếu cấu hình sơ hở </u>"Require all granted" là bất kỳ ai trên mạng cũng vào xem được. Thử gõ */server-status* là thấy.

Trang này làm lộ chi tiết: kết nối nào đang chạy, đang truy cập đường dẫn nào, tổng số request, trạng thái server (rảnh, đang ghi, đang đọc), phiên bản chuẩn và thời gian chạy. Trên server thật, thông tin này để lộ toàn bộ hoạt động của các người dùng khác và danh sách các liên kết nội bộ của hệ thống.
![[Pasted image 20260818161028.png]]
Lưu ý: *mod_status* này bật mặc định trên Ubuntu. Ban đầu có chặn bằng Require local ở file security.conf, nhưng chỉ cần ai đó cấu hình *Require all granted* vào virtual host là nó đè lên luật cũ, mở toang /server-status ra ngoài. Do đó lúc nào cũng nên kiểm tra đường dẫn này kể cả khi server trông có vẻ đã được bảo mật.

> [!NOTE] ok tao có thể hiểu ntn
> cái module nằm ở thư mục */server_status* , nó sẽ lưu hết các trạng thái của server, thông thường người ngoài không đọc được, nhưng nếu mà dev nó cấu hình *Require all granted* thì  tất cả mọi người đều có quyền truy cập

### Cào file ẩn bằng Gobuster
Không phải thông tin giá trị nào cũng hiển thị ở directory listing. Các file backup, file config cũ, file test thường nằm ở thư mục gốc mà không có link nào trỏ tới. Dùng Gobuster để rà quét bằng wordlist.

```bash
# -u là IP mục tiêu
# -w là đường dẫn file wordlist
# -x lệnh cho gobuster chắp thêm mấy cái đuôi này vào mỗi từ để quét
gobuster dir -u http://TARGET_IP:80 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt -x bak,txt,html -t 20

```

Mẹo: <u>Wordlist common.txt khá dài. Nếu thêm 3 đuôi mở rộng (-x bak,txt,html) thì quét sẽ rất lâu.</u> Muốn nhanh thì chạy không có đuôi -x trước để tìm thư mục, sau đó mới thêm -x bak để rà soát các file backup.

```text
root@ip-10-81-64-63:~# gobuster dir -u http://10.48.167.138:80 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt -x bak,txt -t 20
...
/.htpasswd            (Status: 403) [Size: 275]
/backup.bak           (Status: 200) [Size: 178]
/server-status        (Status: 200) [Size: 17539]
...

```

Khi Gobuster quét ra file đuôi .bak thì đây là mục tiêu giá trị, nên tải ngay về để phân tích. File backup thường hay để lộ cấu hình nhạy cảm, mật khẩu hoặc mã nguồn.

Nhớ để mắt tới mấy cái file .htpasswd nữa. Thằng Apache dùng file này để lưu tài khoản và mật khẩu băm cho cái Basic Auth. Vớ được file này thì vác về crack offline, đồng thời mày cũng biết được là trang này có xài xác thực, để sau đó còn biết đường mà đi bruteforce.

```text
root@ip-10-81-64-63:~# curl -s http://10.48.167.138:80/backup.bak
# Apache config backup - DO NOT COMMIT
ServerName company.internal
DocumentRoot /var/www/html
# DB credentials below
# user: dbadmin pass: Backup2024!
# Last updated: 2024-11-15

```

### Chốt hạ

> [!NOTE]
> Đi chọc ngoáy Apache thì cứ thuộc lòng cái quy trình này: *soi header* lấy phiên bản, mò thư mục xem có lộ danh sách file không, soi trang */server-status* và *vác Gobuster ra cào* file ẩn. 4 bước này là húp trọn 99% đống rác mà một con Apache cấu hình lởm phơi ra.


> [!success] Thực hành
> - ban đầu tao sẽ xem web stack dùng là gì
> ![[Pasted image 20260818162743.png]]
> sau đó tao có thể search cve với cái này, nma cái lab này đơn giản thôi, tìm các lỗi directory list hay lỗi cấu hình /file hay lỗi cấu hình của *mod_status* (/server_status)
> ![[Pasted image 20260818162848.png]]
> ![[Pasted image 20260818162904.png]]
> ![[Pasted image 20260818162920.png]]


# 5- NodeJs
Node.js có cách hoạt động rất khác so với các server phục vụ file tĩnh như Apache hay Python. Node.js chạy code trực tiếp, và code đó sẽ toàn quyền quyết định xem request sẽ trả về những gì. Sự linh hoạt này rất mạnh, nhưng cũng tạo ra không ít sơ hở nếu dev lập trình bất cẩn.

Anh em đi săn bug cực khoái móc lốp mấy con app Node.js Express. Tại sao? Vì tụi dev chúa tể lười, lúc code ở máy nhà (*development mode)* thì bật đủ thứ tính năng hỗ trợ, lúc đem lên server thật (*production*) lại không chịu tắt đi, cứ thế để lộ ra. Nào là cổng debug (debug endpoints), nào là cái đống lỗi chi tiết dài ngoằng (verbose error responses), lộ luôn cả biến môi trường. Hậu quả là cái app nó bô bô kể cho mày nghe cách nó được xây lên thế nào, thậm chí nôn luôn cả mật khẩu.

### Framework Fingerprinting
Liếc qua cái header ở port 3000 là mày đọc vị được ngay mục tiêu:

```text
root@ip-10-81-64-63:~# curl -sI http://MACHINE_IP:3000
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 56
ETag: W/"38-K8iCfm09rMr0MV0NsgqdAb94DAk"
Date: Sat, 11 Apr 2026 07:27:28 GMT
Connection: keep-alive
Keep-Alive: timeout=5

```

Thấy cái dòng `X-Powered-By: Express` lù lù ra chưa? Thằng Express mặc định nó sẽ xăm cái dòng này lên, trừ phi thằng dev nó tỉnh táo cạo đi. Thấy chữ này là mày chốt kèo 100% đang chơi với Express, từ đấy biết đường mà móc bài ra đánh.

### Soi version của App
Nhiều con app Express vào thẳng đường dẫn gốc nó sẽ nôn ra một cục JSON thông báo tình trạng sức khỏe:

```text
root@ip-10-81-64-63:~# curl -s http://MACHINE_IP:3000
{"status":"ok","app":"company-portal","version":"1.2.0"}

```

Nhìn thấy cái trường `version` thì húp vội cho tao. Cầm cái version này đi tra xem nó có dính phốt CVE nào không, hay lôi cái changelog của tụi nó ra ngâm cứu xem có hở sườn chỗ nào không.

### Chọc cho nó nôn lỗi (Triggering Verbose Errors)
Thằng Express nó có trò tự động g<u>iấu biến mấy cái dòng báo lỗi chi tiết (stack traces) khi cái biến môi trường `NODE_ENV` bị set là `production`. Còn nếu là `development` thì nó phơi ra hết.</u>

> [!NOTE] 
> *NODE_ENV* là biến môi trường giúp set 'production' hay 'development'. Nếu set chế độ dev, thì server nó sẽ nhả thông báo error, giúp cho dev có thể dễ dàng debug

Tuy nhiên trong thực tế, <u>nhiều dev tự viết hàm xử lý lỗi riêng. Nhưng nếu viết không cẩn thận thì nó cứ thế làm lộ toàn bộ stack traces ra, bất kể `NODE_ENV` đang được cấu hình là gì.</u>
Trong cái bài lab này,<u> con app xài hàm bắt lỗi custom do dev tự chế. </u>Cái cục JSON chà bá mày sắp thấy không phải lỗi mặc định của Express đâu, mà là do thằng dev nặn ra lúc sửa lỗi rồi để nguyên như vậy đưa lên server thật.
*Cả 2 cái trò này* đi test thực tế đầy rẫy: việc bật cờ `NODE_ENV=production` không còn ý nghĩa gì nếu có *hàm xử lý lỗi custom* ghi đè lên. Thử chọc vào cái API này xem:

```json
root@ip-10-81-64-63:~# curl -s http://MACHINE_IP:3000/api/users | python3 -m json.tool
{
    "error": "connect ECONNREFUSED 127.0.0.1:5432",
    "stack": "Error: connect ECONNREFUSED 127.0.0.1:5432\n    at /opt/nodeapp/app.js:16:15\n    at Layer.handle [as handle_request] (/opt/nodeapp/node_modules/express/lib/router/layer.js:95:5)\n...",
    "query": "SELECT * FROM users"
}

```

Thấy server nó giãy nảy báo lỗi 500 (Internal Server Error) từ một cái API là phải sấn vào ngay. Nếu đang ở mode development, nó hay ọc ra nguyên một cục tin nhắn báo lỗi, *stack trace* chằng chịt, và lý do nó tạch. Cái đống stack trace này là cục vàng đấy, nó leak cho mày cấu trúc thư mục giấu bên trong, tên mấy cái module nó xài, thậm chí có cả cái câu query database làm nó sập. Đống thông tin này nếu chỉ nhìn mã lỗi 500 thông thường thì không bao giờ thấy được.

> [!NOTE] 
> Đại khái là gì? Tức là mặc dù cái *NODE_ENV*=poduction ; nhưng thằng dev đã đè 1 cái hàm debug tự viết ; mà sau đó lại quên up luôn lên production.Lúc này mặc dù đang ở mod production thì cái server nó vẫn nhả lỗi đều đều do cái hàm debug tự chế vẫn đang hoạt động

> [!NOTE] chốt lại
> đại khái ở đây đè cập đến 2 cái lỗi cấu hình do dev làm
>  - _NODE_ENV_=poduction
>  - hàm bắt lỗi dev tự viết, nhưng quên ko tắt đi khi đưa lên production

### Moi móc toàn bộ đường dẫn qua cổng Debug
Một trong những trò hớ hênh nực cười nhất của app Express là nó tự nguyện liệt kê hết đường đi lối lại cho mày. <u>Dev lúc phát triển hay viết API debug hiển thị danh sách router để test cho dễ, lên production lại quên không xóa.</u>

```text
root@ip-10-81-64-63:~# curl -s http://MACHINE_IP:3000/api/routes
[{"method":"GET","path":"/"},{"method":"GET","path":"/api/users"},{"method":"GET","path":"/api/routes"},{"method":"GET","path":"/api/debug/env"}]

```

Lấy được danh sách này thì rất tiện lợi. Bạn không cần dùng Gobuster quét mất thời gian nữa, nó chỉ rõ từng đường dẫn để kiểm thử luôn.

> [!NOTE] 
> ok tức là cái /api/routes trên nó là api do dev tự viết để lôi hết thư mục ra, cái này giúp debug.Nma lúc mang lên production thì dev lại ko xóa cái api này đi

> Lưu ý: Cái đường dẫn `/api/routes` kia móc data từ cái property ngầm `app._router.stack` của Express. Nhưng cái trò này chỉ mượt trên Express 4 thôi. Lên bản Express 5 tụi nó đổi lại lõi router, nên nếu chọc vào API này mà thấy nó báo lỗi hoặc nôn ra format lạ hoắc, thì tức là con server nó đang xài phiên bản Express khác, không giống lab này.

### Lộ Biến Môi Trường (Environment Variables)
Mấy cái <u>biến môi trường trong Node.js toàn là nơi lưu thông tin nhạy cảm: mật khẩu database, key API, cờ cấu hình các kiểu.</u> Vớ được endpoint trả về `process.env` là đã phát hiện lỗ hổng nghiêm trọng:

```text
root@ip-10-81-64-63:~# curl -s http://MACHINE_IP:3000/api/debug/env
{"NODE_ENV":"development","DB_PASSWORD":"NodeDBPass2024!","PORT":"3000","DB_HOST":"localhost:5432","APP_NAME":"company-portal"}

```

Thấy chữ nào có mùi như `DB_PASSWORD`, `SECRET_KEY` thì nhặt hết về viết report. Cái cờ `NODE_ENV=development` lù lù kia cũng là lời tự thú rằng server này không hề được gia cố bảo mật cẩn thận.

### Soi File Tĩnh (Static File Serving)
Express hay xài cái middleware `express.static()` để quăng mấy cái file giao diện (như JS, CSS, config) cho người dùng. Khi cái router file tĩnh này được mở, nó quăng nguyên cái thư mục ra ngoài. Lũ file Javascript phía trình duyệt đôi khi bị tụi dev ngáo nhét cứng mẹ luôn mấy cái đường link API, tên miền nội bộ, hay mấy cái cờ debug vào trong đó.

> [!NOTE] Lại là một lỗi cấu hình à? ko có gì mới hơn à?
> - nói chung là tao cần nhớ `express.static()`. Nhưng tao ko hiểu nó là gì?
> - thư mục /static bên dưới , nó chỉ có khi nào??
> ![[Pasted image 20260818174655.png]]

Có được list router từ bước trên rồi, check thử xem thư mục tĩnh nó có gì:

```text
root@ip-10-81-64-63:~# curl -s http://MACHINE_IP:3000/static/config.js
// Client-side configuration
const API_BASE = 'http://internal-api.company.local:8080';
const DEBUG = true;
const VERSION = '1.2.0';

```

Mấy cái file config dạng tĩnh này rất hay bị anh em bỏ qua, vì theo lý thì nó phải public cho trình duyệt đọc mới chạy được. Tuy nhiên, "bắt buộc phải public" khác với việc "chỉ chứa thông tin an toàn".

> [!question] 
> cái file /static này chứa những gì ,tao vẫn chưa rõ? Nhưng có lẽ t có thể hiểu được cái đoạn này là public và lỗi sinh ra do dev sơ ý ném dữ liệu nhạy cảm vào đây? vậy thì tao nghĩ nó chỉ tồn tại ở lý thuyết thôi chứ nó ko phải là một lỗi hay gặp ở thực tế
> ![[Pasted image 20260818174720.png]]



> Chú ý: `express.static()` tự động chặn và báo lỗi 404 với tất cả các file ẩn (bắt đầu bằng dấu chấm), không hiển thị trực tiếp như server Python. Do đó nếu bạn dò tìm file `.env` qua router tĩnh mà bị trả về 404, thì không có nghĩa là file đó không tồn tại, chỉ là bị middleware chặn lại thôi. Muốn tiếp cận thì phải tìm hướng khác hoặc lấy được quyền shell.*==> hmmmmmmm*

### Tổng hợp lại đòn đánh

> [!NOTE] tổng kết lại
> Ngồi nhìn lại xem tao với mày vừa làm gì. Mở màn bằng trò ngửi header để *nhận dạng framework.* Xong *chọc cho app nó báo lỗi* để móc ruột gan ra xem. Mò mẫm cổng debug để *moi danh sách đường dẫn*. Hốt sạch mật khẩu từ *biến môi trường*, rồi lần theo dấu vết tìm đến kho file tĩnh để bú cờ.
> 
> Chuỗi kỹ thuật này rất hiệu quả vì bước trước dọn đường cho bước sau: Header nhận diện framework, lỗi làm lộ cấu trúc nội bộ, endpoint debug chỉ ra danh sách router, file tĩnh phản ánh sự bất cẩn trong cấu hình. Ghi nhớ quy trình này để áp dụng khi thực chiến nhé.

### một tỷ câu hỏi 
> [!success] Thực hành
> - thực tế là phải dùng gobuster mới moi được cái enpoint ra để test, 
> ![[Pasted image 20260818182745.png]]
>![[Pasted image 20260818182717.png]]
![[Pasted image 20260818182345.png]]
![[Pasted image 20260818182528.png]]


> [!question]
> - tại sao trỏ vào /api/user thì nó phun ra lỗi, nhưng khi trỏ vào /api thì không thấy gì?. Vậy cái lớp debug nó nằm ở đâu?
> ![[Pasted image 20260818180839.png]]
> - nhưng mà lạ vậy, sao cái api này lại có thể trigger bởi client được, hay chính vì không có quyền nên nó mới trả về mã lỗi (do dev viết hàm debug, chứ không là nó trả về 404 à)
> ![[Pasted image 20260818180750.png]]
> -> ok tao có thể hiểu sương sương như này
> tại /api/user , có đoạn code kiểu như try , catch{hàm in ra lỗi}. Khi client truy cập tới enpoint này, code trong /api/user chạy nhưng ko có quyền thực thi phía server ,nên nó kích hoạt caí catch kia, nhả ra lỗi


# 6- Ngix
Sau thằng Apache với Node.js, giờ nhảy sang ngâm cứu con Nginx. <u>Kịch bản soi mói thì y chang, nhưng thằng này xài một bộ từ vựng cấu hình riêng.</u> Mấy cái trò lố quen thuộc như lộ hàng phiên bản, lộ danh sách thư mục, hay tơ hơ trang status lại xuất hiện, chỉ khác cái tên lệnh với đường dẫn thôi.

Thằng Nginx nó ngồi ở cái mâm khác so với Apache hay Node.js. Bọn dev toàn xài nó làm reverse proxy (đứng mũi chịu sào), load balancer (chia chác tải), hoặc làm cái máy bơm file tĩnh siêu tốc. Lúc đem lên production, Nginx toàn ngồi ghế đầu chặn cổng, hứng hết luồng traffic từ giang hồ mạng đổ vào trước khi nhả cho con app server đằng sau. Ch<u>ỗ ngồi trọng yếu thế nên cấu hình của nó cực kỳ quan trọng</u>. <u>Cấu hình sai sót là để lộ luôn cấu trúc mạng nội bộ, lộ thông tin vận hành, hoặc mở toang cửa cho người ngoài tải các file đáng lẽ phải giấu kín.</u>![[Pasted image 20260819075812.png]]
> Thông não: Trong bài lab này, Nginx chạy ở port 8080 thay vì 80 như bình thường. Đơn giản vì Apache đã chiếm cổng 80 trên máy này rồi. Còn đi làm thật, *Nginx mặc định chạy ở cổng 80 (HTTP) hoặc 443 (HTTPS).*


> [!NOTE] 
>Ngix mặc định ở cổng 80 hoặc 443

### Lộ hàng phiên bản (Version Disclosure)
Bài cũ soạn lại: cào cái header Server trước.

```text
root@ip-10-81-64-63:~# curl -sI http://MACHINE_IP:8080 | grep -i server
Server: nginx/1.24.0 (Ubuntu)

```

Nginx mặc định cũng để phiên bản vào header Server. Khác với Apache dùng chỉ thị `ServerTokens`, Nginx dùng chỉ thị `server_tokens`. Mặc định là `on`, tức là hiển thị đầy đủ version ra ngoài. Nắm được version chuẩn này thì tra cứu changelog xem có CVE nào liên quan không, hoặc ít nhất cũng có thông tin để đưa vào báo cáo kiểm thử.

Cái lệnh `server_tokens` này nó một tay che trời, kiểm soát luôn cả header Server lẫn cái dòng version trên trang báo lỗi; set `server_tokens off` phát là nó câm nín ở cả 2 chỗ. Nếu mày thấy header Server bị giấu, thử gửi request vào một đường dẫn không tồn tại xem trang lỗi 404 trả về nội dung gì, từ đó kiểm tra xem chỉ thị `server_tokens` đã thực sự được tắt hay chưa:

```html
root@ip-10-81-64-63:~# curl -s http://MACHINE_IP:8080/nonexistent-path
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.24.0 (Ubuntu)</center>
</body>
</html>

```

### Lộ thư mục với Autoindex
Nginx mặc định KHÔNG BẬT liệt kê thư mục. Khi thằng dev ngứa tay muốn phơi cái list file ra, nó sẽ chêm cái lệnh `autoindex` vào trong cái block `location` của file cấu hình:

```nginx
location /files/ {
    autoindex on;
    root /var/www/nginx/;
}

```

Đây là tính năng chuẩn dùng để chia sẻ file. Sai sót cấu hình ở đây là áp dụng nó vào đường dẫn chứa dữ liệu nhạy cảm, hoặc để lộ trên production mà không kiểm soát truy cập. Thử truy cập vào link `http://MACHINE_IP/files/` mà xem:

```html
root@ip-10-81-64-63:~# curl -s http://MACHINE_IP:8080/files/
<html>
<head><title>Index of /files/</title></head>
<body>
<h1>Index of /files/</h1><hr><pre><a href="../">../</a>
<a href="deploy-notes.txt">deploy-notes.txt</a>                                   03-Apr-2026 18:23                 148
<a href="old-backup.tar.gz">old-backup.tar.gz</a>                                  03-Apr-2026 18:23                 236
<a href="server-config.txt">server-config.txt</a>                                  03-Apr-2026 18:23                 135
</pre><hr></body>
</html>

```

Cái định dạng HTML autoindex của Nginx nó tọng ra một cái bảng phèn ỉa gồm tên file, ngày sửa và dung lượng. Mò thấy cái này thì đè từng file ra mà húp. Mấy cái chỗ lộ list thư mục của Nginx toàn là do dev xài làm kho chứa file chung, xong lười vứt cả đống data nhạy cảm của hệ thống vào đấy.

### Chọc ngoáy cổng nginx_status
Thằng module `stub_status` của Nginx chuyên nôn ra số liệu kết nối real-time ở một cái URL do dev tự set. Cấu hình khôn thì chỉ cho phép mỗi localhost được xem. Còn cấu hình sơ hở thì mở cửa cho bất kỳ IP nào cũng vào xem được:
*=> lại là do lỗi cấu hình thôi*
```nginx
location /nginx_status {
    stub_status;
    allow all;  # Đáng nhẽ phải là: allow 127.0.0.1; deny all;
}

```

Thử gửi request xem nó trả về cái gì:
```text
root@ip-10-81-64-63:~# curl -s http://MACHINE_IP:8080/nginx_status
Active connections: 1 
server accepts handled requests
 15 15 15 
Reading: 0 Writing: 1 Waiting: 0 

```

Cái đống output này trông thì gọn nhưng dòng thứ hai không có nhãn chú thích gì cả, lần đầu nhìn khá khó hiểu.
Ba cái con số vô tri ở dòng thứ 2 theo thứ tự là: tổng số kết nối đã nhận, tổng số kết nối đã xử lý, và tổng số request tính từ lúc bật server. Cái dòng thứ 3 thì bóc tách trạng thái của các kết nối đang active.

Lượng dữ liệu này tuy không thể dùng để exploit chiếm quyền server ngay, nhưng nó làm lộ nhiều thông tin vận hành như tải server hay thói quen sử dụng. Đi pentest thực tế, phát hiện `/nginx_status` này cũng tính là một finding có giá trị, vì nó chứng tỏ hệ thống giám sát nội bộ đang bị hở, và khả năng cao là còn các cổng giám sát khác cũng đang bị để lộ.

### Chốt hạ

> [!NOTE]
> Quá trình soi con Nginx này đi theo cái sườn y đúc bài Apache. Check header lấy version, lội thư mục dò autoindex, và ngửi xem có cái cổng status nào lộ ra không. Tên đường dẫn với câu lệnh cấu hình thì khác bọt tí thôi, chứ tư duy bới rác thì vẫn y chang.
> keyword:
>  - *autoindex*
>  - */ngix_status*
>  - */etc/ngix/*

> Mẹo: F<u>ile cấu hình Nginx trên Ubuntu rúc ở</u> `/etc/nginx/`. Nếu mốt mày có quyền nhảy shell vào một con server chạy Nginx, đè ngay mấy cái file config trong thư mục `/etc/nginx/sites-available/` ra mà đọc, mày sẽ thấy mười mươi thư mục nào đang phơi ra ngoài và cái module nào đang bật.

> [!success] thực hành
>![[Pasted image 20260819085311.png]]


# 7-
Tiếp tục đào sâu thêm nhé. Chúng ta vừa tìm hiểu qua 4 loại web server khác nhau, mỗi loại có cách cấu hình và những điểm yếu đặc trưng riêng. Không có hệ thống nào hoàn hảo, có những lỗi cơ bản mà server nào cũng có thể gặp phải. Hai điểm thường thấy nhất là không cấu hình Security Headers và các lỗi cấu hình cơ bản mà các công cụ scan tự động quét qua là phát hiện ra ngay.
### Security Headers (Mấy cái cờ bảo mật)
Security headers thực chất là mấy cái response header để ra lệnh cho trình duyệt web biết phải xử lý nội dung trang như thế nào. Bọn nó sinh ra để làm lá chắn chống lại một mớ các đòn tấn công từ phía người dùng (client-side), kiểu như clickjacking (bấm nhầm link độc), sniffing (đọc trộm nội dung), hay XSS (chèn mã độc). Trong bài lab này, không có server nào được cấu hình để gửi các header bảo mật này, vì đó là trạng thái mặc định của cả 4 server.

Dưới đây là mấy cái header phổ biến nhất và tác dụng của từng thằng:

| Header                      | Chống lại kiểu tấn công nào                                                                             | Ví dụ minh họa                     |
| --------------------------- | ------------------------------------------------------------------------------------------------------- | ---------------------------------- |
| `X-Frame-Options`           | Chống Clickjacking (ngăn không cho web khác nhúng trang của mình vào thẻ iframe)                         | `DENY` hoặc `SAMEORIGIN`           |
| `X-Content-Type-Options`    | Chống MIME sniffing (cấm trình duyệt cầm đèn chạy trước ô tô, tự đoán định dạng file)                   | `nosniff`                          |
| `Content-Security-Policy`   | Giới hạn nghiêm ngặt nguồn tải script, CSS và các tài nguyên khác                                       | `default-src 'self'`               |
| `Referrer-Policy`           | Kiểm soát cái header Referer nôn ra những gì khi user bấm link sang trang khác                          | `no-referrer` hoặc `strict-origin` |
| `Strict-Transport-Security` | Ép buộc trình duyệt phải dùng HTTPS cho các lần truy cập sau (chỉ có tác dụng nếu server đã chạy HTTPS) | `max-age=31536000`                 |

> Thông não: Về mặt kỹ thuật thì `X-Frame-Options` đã được thay thế nhiều bởi `Content-Security-Policy: frame-ancestors` (vì CSP kiểm soát chi tiết hơn rất nhiều). Các trang web hiện đại thường dùng CSP để chống clickjacking và loại bỏ `X-Frame-Options`. Do đó khi kiểm thử và viết báo cáo, hãy kiểm tra kỹ cả 2 cơ chế.

Quét thử từng con server bằng `curl` xem:
```text
root@ip-10-81-64-63:~# for port in 80 8000 3000 8080; do echo "=== Port $port ==="; curl -sI http://10.48.149.51:$port/ | grep -iE "x-frame-options|x-content-type|content-security-policy|strict-transport|referrer-policy" || echo "(no security headers found)"; done
=== Port 80 ===
(no security headers found)
=== Port 8000 ===
(no security headers found)
=== Port 3000 ===
(no security headers found)
=== Port 8080 ===
(no security headers found)

```

Khi lệnh `grep` không trả về kết quả nào, tức là server không có header bảo mật nào cả. Thử qua cả 4 port sẽ thấy cấu hình mặc định của cả 4 loại server đều thiếu sót ở điểm này. Security headers không tự động xuất hiện, nếu dev hoặc admin không tự tay cấu hình thì sẽ không bao giờ có.

> Lưu ý: `Strict-Transport-Security` chỉ có ý nghĩa khi chạy trên HTTPS. Lab này đang chạy HTTP thường nên không có nó là chuyện bình thường. Nhắc ở đây cho đầy đủ chứ không tính là một lỗi cấu hình sai.

### Càn quét tự động bằng Nikto
Nikto là công cụ chuyên quét web server để tìm các lỗi cấu hình phổ biến, phần mềm lỗi thời, lộ cổng quản trị, hay thiếu Security headers. Công cụ này hoạt động không hề âm thầm; nó quét tạo ra lượng traffic lớn và rất dễ bị phát hiện. Vì thế nó chỉ phù hợp khi kiểm thử đã được cấp phép rõ ràng, chứ không phù hợp cho các hoạt động cần ẩn mình (stealth). Quét thử vào Apache xem:

```text
root@ip-10-81-64-63:~# nikto -h http://10.48.149.51:80 -nointeractive
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          10.48.149.51
+ Target Hostname:    10.48.149.51
+ Target Port:        80
+ Start Time:         2026-04-11 09:16:09 (GMT1)
---------------------------------------------------------------------------
+ Server: Apache/2.4.58 (Ubuntu)
+ Server leaks inodes via ETags, header found with file /, fields: 0x29af 0x64e9243796aa2 
+ The anti-clickjacking X-Frame-Options header is not present.
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Allowed HTTP Methods: HEAD, GET, POST, OPTIONS 
+ OSVDB-561: /server-status: This reveals Apache information. Comment out appropriate line in httpd.conf or restrict access to allowed hosts.
+ OSVDB-3268: /files/: Directory indexing found.
+ OSVDB-3092: /files/: This might be interesting...
+ 6544 items checked: 0 error(s) and 6 item(s) reported on remote host
+ End Time:           2026-04-11 09:16:18 (GMT1) (9 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

```

Cờ `-nointeractive` giúp công cụ tự động quét liên tục mà không dừng lại yêu cầu người dùng xác nhận. Thằng Nikto sẽ cào hàng trăm cái đường dẫn và quy tắc phổ biến. Chú ý nhìn mấy cái dòng bắt đầu bằng dấu `+` trong kết quả, đấy là đồ ăn được đấy.

Trên một server Apache để cấu hình sơ hở như này, Nikto sẽ nhanh chóng phát hiện trang `/server-status` mở công khai, file `backup.bak` để ở thư mục gốc, lỗi liệt kê danh sách file ở `/files/`, cùng hàng loạt cảnh báo thiếu security headers.

> Mẹo: Output của Nikto thỉnh thoảng rất dài dòng. Bạn có thể dùng tùy chọn `-Tuning` để chỉ định quét một số loại lỗi cụ thể. Muốn nhanh gọn thì chạy lệnh `nikto -h TARGET -Tuning 123`, nó sẽ gom các finding phổ biến nhất mà không cần quét toàn bộ từ điển. Nhớ là các mã số tuning này viết liền nhau, không có dấu phẩy ngăn cách.

### Những sai sót cấu hình phổ biến (Ở đâu cũng gặp)

Nhìn lại cả 4 server, bạn sẽ thấy những điểm sơ hở cấu hình này lặp đi lặp lại tương tự nhau:

| Lỗi Cấu Hình | Apache | Python HTTP | Node.js | Nginx |
| --- | --- | --- | --- | --- |
| Lộ version ở Header | Có | Có | Một phần | Có |
| Phơi danh sách thư mục | `/files/` | Đường dẫn gốc | Không có | `/files/` |
| Lộ trang status/debug | `/server-status` | Không có | `/api/debug/env`, `/api/routes` | `/nginx_status` |
| Lộ file nhạy cảm | `backup.bak`, `internal-notes.txt` | `.env`, `notes.txt`, `backup.zip` | `config.js` | `server-config.txt`, `deploy-notes.txt` |
| Không có Security Headers | Tất cả | Tất cả | Tất cả | Tất cả |

Điểm chung cốt lõi ở đây là: các cấu hình mặc định luôn ưu tiên sự tiện lợi hơn là bảo mật. Lộ version, lộ danh sách thư mục, hay mở trang status đều được bật mặc định để dễ theo dõi hệ thống và giảm bớt gánh nặng quản trị ban đầu. Muốn tắt hoặc siết chặt lại thì phải có người trực tiếp thao tác. Trong thực tế, các lỗi này thường không phải do quản trị viên cố ý tạo ra, mà đơn giản là do không ai rà soát lại các thiết lập mặc định từ lúc cài đặt ban đầu.
