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
Trước khi mày đâm đầu vào việc dò thư mục hay test input, mày phải biết mày đang đánh nhau với con cặc gì đã. Cái phần mềm web server nó sẽ quyết định xem hệ thống có thể dính những cái lỗi cấu hình ngu nào, đường dẫn nào đáng để chọc, và xài tool gì thì mới ăn thua. Cái việc nhận diện server đéo phải làm cho có đâu con ạ. Nó ảnh hưởng trực tiếp đến mọi đường đi nước bước tiếp theo của mày.

Tin vui cho mày là đa số mấy con web server nó toàn tự khai tên bô bô ra. Bọn dev với admin thì hay lười vãi lz, toàn bê nguyên cái cấu hình mặc định để đấy, mà cấu hình mặc định thì lại hay phun ra cả đống thông tin phiên bản. Có mấy chỗ cực kỳ uy tín để mày dòm ngó đây.

### Cái Header Phản Hồi Của Server (Server Response Header)

Tín hiệu trinh sát thẳng mặt nhất chính là cái header `Server` trong phản hồi HTTP. Khi mày gửi bất kỳ cái request đéo nào đến web server, nó cũng nhét cái header này vào câu trả lời. Mỗi phần mềm server sẽ đẻ ra một cái format khác nhau, và mấy cái format này đủ chuẩn để mày dùng làm cột mốc nhận dạng luôn.

Xách thằng lệnh `curl` lên, thêm cái cờ `-I` vào để yêu cầu nôn header ra thôi, đéo thèm lấy phần thân (body) làm mẹ gì:

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

Cái header `Server` nó khai toẹt mẹ luôn cho mày phần mềm với phiên bản là gì. Đéo phải server nào cũng lộ hàng tơ hơ thế này đâu. Gặp mấy con server cấu hình bảo mật xịn thì nó chỉ phun ra chữ `Apache` hoặc giấu nhẹm mẹ luôn cái header, nhưng đa số cấu hình mặc định trên mấy con Ubuntu thì cứ phơi hết hàng ra cho mày xem.

Đây là những gì mà từng con server trong cái bài lab này mặc định sẽ nôn ra:

| Port | Server | Default Server Header |
| --- | --- | --- |
| 80 | Apache2 | Apache/2.4.x (Ubuntu) |
| 8000 | Python HTTP Server | SimpleHTTP/0.6 Python/3.xx.x |
| 3000 | Node.js Express | None (do ứng dụng tự set) |
| 8080 | Nginx | nginx/1.xx.x |

Để ý con Node.js Express đéo có cái header `Server` nào cả. Vì bản thân thằng Express mặc định đéo thèm set, và cái lớp Node.js HTTP ở dưới đít nó cũng đéo set nốt. Bọn dev phải tự tay chêm vào thì mới có. <u>Trên một con app Express thực tế, việc đéo có cái header `Server` tự nó đã là một cái tín hiệu rồi. </u>C<u>ột mốc uy tín nhất để nhận dạng thằng Express là cái header `X-Powered-By`, cái này thì Express tự động gán,</u> trừ khi thằng dev nó ngứa tay gỡ đi.

> [!NOTE] Nhận biết server thông qua header
> - Phần này chỉ cần nhớ cái bảng trên là được
> - còn với thằng express nó không có header server-> đây cũng là một tín hiệu .Thêm nữa là header `X-Powered-By`

### Cái Header X-Powered-By
Có vài thằng framework thích thêm cái header `X-Powered-By` để khai luôn danh tính cái mảng ứng dụng đang núp bóng phía sau server. Thằng Express thì set cái này mặc định cmnr:
`X-Powered-By: Express`

Cái header này nó đéo dính dáng gì đến header `Server`, và đối với thằng Express, đây chính là cái dấu vân tay chuẩn mẹ nhất của nó. Đéo giống tụi Apache hay Nginx cứ rống lên báo danh ở header `Server`, thằng Express dùng `X-Powered-By` để nhận diện cội nguồn. Cứ thấy port đéo nào mà mất cái header `Server` hoặc nhìn nó chung chung quá thì mày phải soi ngay cái `X-Powered-By` này.

### Trinh sát qua DevTools của Trình duyệt

Nếu mày đang mò mẫm trên trình duyệt, thì cái tab Network trong DevTools nôn ra đủ mẹ thông tin header đéo thiếu chữ nào mà khỏi cần xài tool lằng nhằng. Mở `http://MACHINE_IP:3000` trên Firefox, nhấp chuột phải bừa vào chỗ nào đó trên web rồi chọn Inspect (hoặc vã mẹ nút F12) để móc cái Developer Tools lên. Nhảy sang tab Network và f5 lại trang để hốt hết đống request. Chọn cái request chính trong danh sách, mò xuống phần Headers, soi mục Response Headers là mày thấy tuốt tuồn tuột ruột gan con server nó đáp lại những gì.
![[Pasted image 20260818144729.png]]
> [!NOTE] ????
>ơ cái dcm, vậy nãy giờ học cách recon bằng curl công cốc à. Tại dùng mẹ f12 nó lù lù ra đấy rồi
>![[Pasted image 20260818145733.png]]
>--> nói chung là curl vẫn bá đạo, cái f12 chỉ là tầng nông thôi

### Mấy Cái Trang Báo Lỗi Mặc Định (Default Error Pages)

Cái cờ `-sI` tao bảo ở trên là gửi request HEAD, nó chỉ ném lại header thôi đéo có body. Muốn xem được mấy cái trang báo lỗi mặc định, mày phải quất cái request GET bằng cách giữ cờ `-s` mà vứt cụ cái `-I` đi:

```bash
# Gửi HEAD request: chỉ lấy headers, đéo lấy body
curl -sI http://MACHINE_IP:PORT/

# Gửi GET request: móc luôn cả body, đầy đủ response
curl -s http://MACHINE_IP:PORT/nonexistent-page-xyz

```

Khi mày cố tình chọc vào một cái đường dẫn đéo tồn tại, đa số server sẽ chửi mày bằng cái trang lỗi 404 mặc định. Giao diện trang lỗi của từng con server nó đặc trưng vãi cứt. Thằng Python thì quăng ra cái phản hồi toàn chữ suông. Thằng Nginx thì nhét phiên bản của nó dưới đáy trang HTML. Thằng Apache thì phô luôn tên nó ngay giữa body trang.

Chính mấy cái sự khác bọt này giúp mày bóc phốt được danh tính con server kể cả khi nó đã cố tình giấu cái header `Server`. Mày cứ test thử chiêu này qua cả 4 cổng mà xem cách mỗi con server nó xử lý chửi mày khác nhau thế nào trong cùng một tình huống. Đọc xong thì đi thực hành đi con chó!

> [!success] thực hành
> ![[Pasted image 20260818150245.png]]

# 3-python http sever
Thằng Python đẻ ra đã ngậm sẵn một cái HTTP server, dev đéo nào cũng có thể bật lên chỉ bằng một nốt nhạc. Chính vì cái sự tiện vãi l này mà anh em đi pentest cực kỳ hay gặp nó.

```bash
# This command serves the current working directory over HTTP on port 8000
python3 -m http.server 8000

```

Mấy thằng dev hay xài cái này để share file cho lẹ, test mấy trang web tĩnh, hoặc bắn data giữa 2 máy trong cùng mạng. Nhưng cái rắc rối là lúc đéo nào cái trò "share file cho lẹ" cũng biến thành "hớ hênh phơi mẹ ra internet nửa năm trời". Nó chạy phây phây trên mấy con server public, mạng nội bộ, hay trên cloud chỉ vì có thằng ngáo đá nào đó mở port 8000 rồi quên cmn mất. <u>Cái server này đéo có phân quyền, đéo có đăng nhập, cũng đéo có ghi log mẹ gì </u>ngoài mấy cái rác rưởi terminal chộp được.

> [!NOTE] Title
> Tức là luôn có một cái http server , dev bật cái này lên là tất cả thư mục và file phía backend nó lộ hết, chỉ cần truy cập vào port (ở ví dụ trên là 8000) thì mình sẽ truy cập được 

### Nó phơi ra những cái đ gì?
Cứ bật lên là nó nôn sạch sành sanh mọi thứ trong thư mục hiện tại ra, kể cả mấy cái file ẩn giấu kỹ như `.env`. Đéo có file `.htaccess` để chặn, đéo có danh sách đen (blocklist), đéo có file cấu hình mẹ gì sất. Cứ có file trong thư mục là bất kỳ thằng l nào mò được vào port 8000 cũng tải được hết. <u>Khác bọt hoàn toàn với Apache hay Nginx </u>(bọn này mặc định cấm xem danh sách thư mục và admin có thể khóa mõm từng đường dẫn). <u>Thằng Python này chỉ có đúng một chế độ: phơi hết cho thiên hạ xem.</u>

### Lộ danh sách thư mục (Directory Listing)
Nếu trong thư mục đéo có file `index.html`, con server Python sẽ tự động nặn ra một cái trang HTML liệt kê sạch mọi file mà nó dòm thấy. Mày cứ chọc thẳng vào thư mục gốc của server bằng lệnh này hoặc mở mẹ link `[http://10.48.167.138:8000](http://10.48.167.138:8000)` trên trình duyệt:
![[Pasted image 20260818154830.png]]
```bash
curl -s http://10.48.167.138:8000/

```

Mày sẽ thấy một cái danh sách với tiêu đề chà bá `Directory listing for /`. Đm, mọi thứ trong cái list đấy đều có thể bấm vào và tải về mượt mà.

### Móc lốp file ẩn (Dotfiles)
Mấy cái file có dấu chấm đứng đầu như `.env` bình thường sẽ bị giấu tịt đi trên Linux, nhưng con server Python này thì đéo thèm quan tâm quy tắc đấy. Nó phơi ra hết như một file bình thường. Cái file `.env` luôn là miếng mồi ngon vì tụi dev hay nhét cấu hình hệ thống vào đấy, mà cấu hình thì kiểu đéo gì chả dính mật khẩu với key bí mật:

```text
root@attackbox:~# curl -s http://10.48.167.138:8000/.env
SECRET_KEY=dev-secret-key-do-not-use
DATABASE_URL=postgresql://webapp:S3cur3DBPass!@localhost/production
DEBUG=True

```

### Tải và soi mấy file nén (Archives)
Nếu mày liếc thấy mấy cục `.zip`, `.tar.gz` hay file nén đéo nào khác trong danh sách, hốt ngay về rồi soi ruột nó. Bọn dev thỉnh thoảng lười để mẹ file backup nằm hớ hênh trong chính cái thư mục đang mở server, trong đó rất dễ có mã nguồn, bản sao database (dump), hoặc file cấu hình xịn.

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
Săn được con server Python này là một quả finding cực kỳ thực tế vì mày đéo cần phải hack (exploit) cái mả mẹ gì cả. Đéo có lỗ hổng nào để kích hoạt. Bản thân con server nó đang hoạt động đúng y như thiết kế. <u>Cái ngu ở đây là nó bị bật ở cái nơi đéo nên bật, phơi ra những cái file đéo nên phơi.</u> Khi đi làm pentest thật, viết report đéo phải chỉ kêu là "ê tao thấy có cái server", mà mày phải giải thích được nó đang làm lộ cái gì và một thằng hacker có thể bú được cái gì từ đống thông tin đấy.

> Chú ý: Nếu trong thư mục mà có file `index.html`, thằng Python sẽ lôi cái file đó ra hiển thị thay vì hiện cái danh sách thư mục. Nếu mày chọc vào thư mục gốc mà đéo thấy danh sách file, thì thử gõ thẳng tên đường dẫn vào hoặc mò sâu xuống mấy thư mục con xem sao.


> [!NOTE] chốt lại
> - sao cái chức năng này nó ngu l thế , mà thực tế vẫn có thằng dùng thật à??
> ![[Pasted image 20260818154938.png]]


> [!success] Thực hành
> ![[Pasted image 20260818155435.png]]
> - file backup bên dưới
> ![[Pasted image 20260818155507.png]]


# 4- Apache 2
Apache là con web server phổ biến nhất thế giới, đi test dạo kiểu đéo gì mày cũng va phải nó. Cấu hình mặc định trên Ubuntu của nó toàn mở toang mấy cái lỗ hổng mà anh em pentester hay bú. <u>3 cái lỗi kinh điển nhất là </u>lộ danh sách file (*directory listing*), *lộ trang server-status* và *vứt quên file backup* ở thư mục gốc.

### Lộ hàng phiên bản (Version Disclosure)
Bắt đầu bằng trò cơ bản nhất: soi cái header Server.

```text
root@ip-10-81-64-63:~# curl -SI http://10.48.167.138:80 | grep -i server
Server: Apache/2.4.58 (Ubuntu)

```

Thằng Apache trên Ubuntu hay xài cái cấu hình ServerTokens OS mặc định, nôn mịe cả tên hệ điều hành lẫn số phiên bản ra. <u>Nắm được bản nào là mày tra CVE với soi được con server này làm được trò trống gì.</u>

> [!NOTE] Title
> dễ vl vậy , thằng apache này nôn hết luôn phiên bản trong header của nó luôn r

### Lộ thư mục (Directory Listing)
Nếu mày dòm vào một thư mục đéo có file index.html, cái lệnh *Options +Indexes* của Apache sẽ phơi sạch danh sách file ra cho mày xem. <u>Bọn dev hay cố tình bật cái này để share file nội bộ, nhưng toàn não cá vàng để quên ở mấy chỗ chứa data nhạy cảm.</u>
![[Pasted image 20260818160959.png]]
Mày mò vào cái link /files/ thử xem, sẽ thấy nguyên cái bảng HTML "Index of /files". Nó hiện rõ tên file, dung lượng, ngày sửa. Đm, vào đây thì cứ đè từng file ra mà đọc. Mấy thằng lười toàn vứt file excel CSV, tài liệu nội bộ hay file backup ở đây. Nhìn thấy thì húp hết cho tao.

> [!question] ??
> tao chưa hiểu cái option+index này bật như nào, và chỉ khi bật thì cái /file này mới tồn tại à??
> ![[Pasted image 20260819072739.png]]
> ![[Pasted image 20260819072811.png]]

### Cái trang mod_status
<u>Thằng Apache có sẵn một cái trang trạng thái </u>do module *mod_status* gánh. Đáng lẽ chỉ cho localhost xem,<u> nhưng cứ cấu hình ngu </u>"Require all granted" là thằng ất ơ nào trên mạng cũng vào xem được. Mày gõ thử */server-status* là thấy.

Trang này nó leak sạch sành sanh: kết nối nào đang chạy, đang truy cập link đéo nào, tổng số request, trạng thái server (rảnh, đang ghi, đang đọc), phiên bản chuẩn và thời gian chạy. Trên server thật, cái này leak mẹ hết thông tin xem người dùng khác đang làm cái l gì và hệ thống có mấy cái link nội bộ nào.
![[Pasted image 20260818161028.png]]
Lưu ý: Cái *mod_status* này bật mặc định trên Ubuntu. Bọn nó có chặn bằng Require local ở file security.conf, nhưng đm chỉ cần thằng dev ngáo chó nào phang cái *Require all granted* vào file cấu hình virtual host là nó đè luôn luật cũ, mở toang cái /server-status cho cả thế giới. Nên lúc đéo nào cũng phải check cái link này kể cả khi server trông có vẻ bảo mật.

> [!NOTE] ok tao có thể hiểu ntn
> cái module nằm ở thư mục */server_status* , nó sẽ lưu hết các trạng thái của server, cái này người thường đ đọc được, nhưng nếu mà dev nó cấu hình *Require all granted* thì  tất cả mọi người đều có quyền truy cập

### Cào file ẩn bằng Gobuster
Đéo phải cục mồi nào cũng phơi ra ở directory listing. Mấy cái file backup, file config cũ, file test toàn nằm chết dí ở thư mục gốc mà đéo có link nào trỏ tới. Vác con Gobuster ra mà càn quét bằng wordlist.

```bash
# -u là IP mục tiêu
# -w là đường dẫn file wordlist
# -x lệnh cho gobuster chắp thêm mấy cái đuôi này vào mỗi từ để quét
gobuster dir -u http://TARGET_IP:80 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt -x bak,txt,html -t 20

```

Mẹo: <u>Cái list common.txt dài vãi cứt. Mày mà nã thêm 3 cái đuôi (-x bak,txt,html) thì quét lâu vãi lồn.</u> Muốn nhanh thì chạy không có đuôi -x trước để húp thư mục, xong mới thêm -x bak để vét máng mấy file backup.

```text
root@ip-10-81-64-63:~# gobuster dir -u http://10.48.167.138:80 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt -x bak,txt -t 20
...
/.htpasswd            (Status: 403) [Size: 275]
/backup.bak           (Status: 200) [Size: 178]
/server-status        (Status: 200) [Size: 17539]
...

```

Khi con Gobuster khạc ra file đuôi .bak thì mày trúng mánh cmnr, tải ngay về. File backup hay giấu mấy đoạn config bẩn, mật khẩu hoặc source code.

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
Đm con hàng Node.js này nó khệnh vãi l, cách nó chạy đéo giống mấy con server vứt file tĩnh kiểu Apache hay Python đâu. Bọn Node.js nó chạy code mẹ luôn, và đoạn code đấy sẽ nắm quyền sinh sát xem request này thì nôn ra cái gì. Sự linh hoạt này đỉnh thì có đỉnh, nhưng đéo thiếu chỗ cho mấy thằng dev ngáo đá tự bóp dái.

Anh em đi săn bug cực khoái móc lốp mấy con app Node.js Express. Tại sao? Vì tụi dev chúa tể lười, lúc code ở máy nhà (*development mode)* thì bật đủ thứ tính năng hỗ trợ, lúc đem lên server thật (*production*) lại đéo chịu tắt đi, cứ thế phơi nguyên xi ra. Nào là cổng debug (debug endpoints), nào là cái đống lỗi chi tiết dài ngoằng (verbose error responses), lộ luôn cả biến môi trường. Hậu quả là cái app nó bô bô kể cho mày nghe cách nó được xây lên thế nào, thậm chí nôn luôn cả mật khẩu.

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

Nhưng đời đéo như mơ, <u>mấy thằng dev hay thích thể hiện, tự viết cái hàm bắt lỗi riêng. Mà viết ngu thì nó cứ thế xì khói tung tóe hết cái stack traces ra, đéo cần biết `NODE_ENV` đang là cái lồn gì.</u>
Trong cái bài lab này,<u> con app xài hàm bắt lỗi custom do dev tự chế. </u>Cái cục JSON chà bá mày sắp thấy đéo phải lỗi mặc định của Express đâu, mà là do thằng dev nặn ra lúc sửa lỗi rồi vứt mẹ đấy lên server thật.
*Cả 2 cái trò này* đi test thực tế đầy rẫy: cái vụ gạt cờ `NODE_ENV=production` đéo có ý nghĩa con cặc gì nếu có cái *hàm bắt lỗi custom* nó đè lên. Thử chọc vào cái API này xem:

```json
root@ip-10-81-64-63:~# curl -s http://MACHINE_IP:3000/api/users | python3 -m json.tool
{
    "error": "connect ECONNREFUSED 127.0.0.1:5432",
    "stack": "Error: connect ECONNREFUSED 127.0.0.1:5432\n    at /opt/nodeapp/app.js:16:15\n    at Layer.handle [as handle_request] (/opt/nodeapp/node_modules/express/lib/router/layer.js:95:5)\n...",
    "query": "SELECT * FROM users"
}

```

Thấy server nó giãy nảy báo lỗi 500 (Internal Server Error) từ một cái API là phải sấn vào ngay. Nếu đang ở mode development, nó hay ọc ra nguyên một cục tin nhắn báo lỗi, *stack trace* chằng chịt, và lý do nó tạch. Cái đống stack trace này là cục vàng đấy, nó leak cho mày cấu trúc thư mục giấu bên trong, tên mấy cái module nó xài, thậm chí có cả cái câu query database làm nó sập. Đống này mày nhìn mã lỗi 500 chay đéo bao giờ thấy được đâu.

> [!NOTE] 
> Đại khái là gì? Tức là mặc dù cái *NODE_ENV*=poduction ; nhưng thằng dev đã đè 1 cái hàm debug tự viết ; mà sau đó lại quên up luôn lên production.Lúc này mặc dù đang ở mod production thì cái server nó vẫn nhả lỗi đều đều do cái hàm debug tự chế vẫn đang hoạt động

> [!NOTE] chốt lại
> đại khái ở đây đè cập đến 2 cái lỗi cấu hình do dev làm
>  - _NODE_ENV_=poduction
>  - hàm bắt lỗi dev tự viết, nhưng quên ko tắt đi khi đưa lên production

### Moi móc toàn bộ đường dẫn qua cổng Debug
Một trong những trò hớ hênh nực cười nhất của app Express là nó tự nguyện liệt kê hết đường đi lối lại cho mày. <u>Bọn dev lúc làm hay viết cái API debug lôi hết mẹ danh sách router ra để test cho dễ, lên production lười đéo xóa.</u>

```text
root@ip-10-81-64-63:~# curl -s http://MACHINE_IP:3000/api/routes
[{"method":"GET","path":"/"},{"method":"GET","path":"/api/users"},{"method":"GET","path":"/api/routes"},{"method":"GET","path":"/api/debug/env"}]

```

Bú được cái list này thì sướng vãi lồn. Mày đéo cần vác Gobuster ra cào bừa tốn thời gian nữa, nó chỉ tay chỉ đường cho mày đi luôn.

> [!NOTE] 
> ok tức là cái /api/routes trên nó là api do dev tự viết để lôi hết thư mục ra, cái này giúp debug.Nma lúc mang lên production thì dev lại ko xóa cái api này đi

> Lưu ý: Cái đường dẫn `/api/routes` kia móc data từ cái property ngầm `app._router.stack` của Express. Nhưng cái trò này chỉ mượt trên Express 4 thôi. Lên bản Express 5 tụi nó đổi lại lõi router, nên nếu chọc vào API này mà thấy nó báo lỗi hoặc nôn ra format lạ hoắc, thì tức là con server nó đang xài phiên bản Express khác đéo giống cái lab này.

### Lộ cmn Biến Môi Trường (Environment Variables)
Mấy cái <u>biến môi trường trong Node.js toàn là ổ giấu hàng nóng: mật khẩu database, key API, cờ cấu hình các kiểu.</u> Vớ được cái endpoint nôn ra `process.env` là trúng số cmnr:

```text
root@ip-10-81-64-63:~# curl -s http://MACHINE_IP:3000/api/debug/env
{"NODE_ENV":"development","DB_PASSWORD":"NodeDBPass2024!","PORT":"3000","DB_HOST":"localhost:5432","APP_NAME":"company-portal"}

```

Thấy chữ nào có mùi như `DB_PASSWORD`, `SECRET_KEY` thì nhặt hết về viết report. Cái cờ `NODE_ENV=development` lù lù kia cũng là lời tự thú rằng con server này đéo hề được gia cố bảo mật cẩn thận.

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

Mấy cái file config dạng tĩnh này rất hay bị anh em bỏ qua, vì theo lý thì nó phải public cho trình duyệt đọc mới chạy được. Nhưng đm, "bắt buộc phải public" khác với việc "chỉ chứa thông tin an toàn".

> [!question] 
> cái file /static này chứa những gì ,tao vẫn chưa rõ? Nhưng có lẽ t có thể hiểu được cái đoạn này là public và lỗi sinh ra do dev sơ ý ném dữ liệu nhạy cảm vào đây? vậy thì tao nghĩ nó chỉ tồn tại ở lý thuyết thôi chứ nó ko phải là một lỗi hay gặp ở thực tế
> ![[Pasted image 20260818174720.png]]



> Cú lừa: Cái thằng `express.static()` khôn hơn mày tưởng, nó tự động chặn mõm báo lỗi 404 với tất cả mấy cái file ẩn (bắt đầu bằng dấu chấm). Đéo phơi hàng thô bỉ như con server Python đâu. Nên nếu mày dò tìm file `.env` qua cái router tĩnh mà bị trả về 404, thì đéo có nghĩa là file đó không tồn tại đâu, chỉ là bị cái middleware nó chặn mặt thôi. Muốn húp thì phải kiếm đường khác hoặc lấy được quyền shell.*==> hmmmmmmm*

### Tổng hợp lại đòn đánh

> [!NOTE] tổng kết lại
> Ngồi nhìn lại xem tao với mày vừa làm gì. Mở màn bằng trò ngửi header để *nhận dạng framework.* Xong *chọc cho app nó báo lỗi* để móc ruột gan ra xem. Mò mẫm cổng debug để *moi danh sách đường dẫn*. Hốt sạch mật khẩu từ *biến môi trường*, rồi lần theo dấu vết tìm đến kho file tĩnh để bú cờ.
> 
> Cái chuỗi đòn này ăn tiền vì bước trước dọn đường cho bước sau. Header chỉ điểm framework. Lỗi thì leak cấu trúc ngầm. Debug thì nôn router. File tĩnh thì vạch mặt cái sự bất cẩn của tụi dev. Khắc cốt ghi tâm cái quy trình này đi con chó.

### một tỷ câu hỏi 
> [!success] Thực hành
> - thực tế là phải dùng gobuster mới moi được cái enpoint ra để test, 
> ![[Pasted image 20260818182745.png]]
>![[Pasted image 20260818182717.png]]
![[Pasted image 20260818182345.png]]
![[Pasted image 20260818182528.png]]


> [!question]
> - tại sao trỏ vào /api/user thì nó phun ra lỗi, nhưng khi tao trỏ vào /api thì đéo thấy gì?. Vậy cái lớp debug nó nằm ở đâu?
> ![[Pasted image 20260818180839.png]]
> - nhưng mà óc chó vậy, sao cái api này lại có thể trigger bởi client được, hay chính vì đéo có quyền nên nó mới trả về mã lỗi(do thằng dev viết hàm debug, chứ ko là nó trả về 404 à)
> ![[Pasted image 20260818180750.png]]
> -> ok tao có thể hiểu sương sương như này
> tại /api/user , có đoạn code kiểu như try , catch{hàm in ra lỗi}. Khi client truy cập tới enpoint này, code trong /api/user chạy nhưng ko có quyền thực thi phía server ,nên nó kích hoạt caí catch kia, nhả ra lỗi


# 6- Ngix
Sau thằng Apache với Node.js, giờ nhảy sang ngâm cứu con Nginx. <u>Kịch bản soi mói thì y chang, nhưng thằng này xài một bộ từ vựng cấu hình riêng.</u> Mấy cái trò lố quen thuộc như lộ hàng phiên bản, lộ danh sách thư mục, hay tơ hơ trang status lại xuất hiện, chỉ khác cái tên lệnh với đường dẫn thôi.

Thằng Nginx nó ngồi ở cái mâm khác so với Apache hay Node.js. Bọn dev toàn xài nó làm reverse proxy (đứng mũi chịu sào), load balancer (chia chác tải), hoặc làm cái máy bơm file tĩnh siêu tốc. Lúc đem lên production, Nginx toàn ngồi ghế đầu chặn cổng, hứng hết luồng traffic từ giang hồ mạng đổ vào trước khi nhả cho con app server đằng sau. Ch<u>ỗ ngồi VIP thế nên cấu hình của nó quan trọng vcl</u>. <u>Cấu hình ngu phát là nó phơi mẹ luôn cái cấu trúc mạng nội bộ, lộ mẹ thông tin vận hành, hoặc mở toang cửa cho thiên hạ tải mấy file đáng lẽ phải giấu kín.</u>![[Pasted image 20260819075812.png]]
> Thông não: Trong cái bài lab củ lz này, con Nginx nó cắm chốt ở port 8080 thay vì 80 như bình thường. Đơn giản vì thằng Apache đã chiếm mẹ cái slot 80 trên con máy này rồi. Chứ đi làm thật, *Nginx nó mặc định bú cổng 80 (HTTP) hoặc 443 (HTTPS) nhé.*


> [!NOTE] 
>Ngix mặc định ở cổng 80 hoặc 443

### Lộ hàng phiên bản (Version Disclosure)
Bài cũ soạn lại: cào cái header Server trước.

```text
root@ip-10-81-64-63:~# curl -sI http://MACHINE_IP:8080 | grep -i server
Server: nginx/1.24.0 (Ubuntu)

```

Thằng Nginx mặc định cũng nhét cmn phiên bản vào header Server. Đéo như Apache xài cái lệnh `ServerTokens`, thằng này xài lệnh `server_tokens`. Mặc định của nó là `on`, tức là khoe mẹ hết version ra. Bớ được cái version chuẩn này thì vác đi tra changelog xem có húp được cái CVE đéo nào không, hoặc bét nhất cũng có cái điền vào report chém gió lấy tiền.

Cái lệnh `server_tokens` này nó một tay che trời, kiểm soát luôn cả header Server lẫn cái dòng version trên trang báo lỗi; set `server_tokens off` phát là nó câm nín ở cả 2 chỗ. Nếu mày thấy header Server bị giấu, thử chọc vào một cái đường dẫn đéo tồn tại xem trang lỗi 404 nó nôn ra cái gì, từ đó check xem lệnh `server_tokens` đã thực sự bị tắt hay chỉ là bị thằng admin cấu hình mõm:

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
Nginx mặc định ĐÉO BẬT liệt kê thư mục. Khi thằng dev ngứa tay muốn phơi cái list file ra, nó sẽ chêm cái lệnh `autoindex` vào trong cái block `location` của file cấu hình:

```nginx
location /files/ {
    autoindex on;
    root /var/www/nginx/;
}

```

Đây là tính năng xịn xò có trong sách vở đàng hoàng, chuyên dùng để share file. Cái cấu hình ngu ở đây là phang nó vào cái đường dẫn chứa data nhạy cảm, hoặc vứt hớ hênh trên production mà đéo thèm kiểm soát truy cập. Mò thử vào cái link `http://MACHINE_IP/files/` mà xem:

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
Thằng module `stub_status` của Nginx chuyên nôn ra số liệu kết nối real-time ở một cái URL do dev tự set. Cấu hình khôn thì chỉ cho phép mỗi localhost được xem. Còn cấu hình óc chó thì mở cửa cho cả thế giới vác IP vào coi:
*=> lại là do lỗi cấu hình thôi*
```nginx
location /nginx_status {
    stub_status;
    allow all;  # Đáng nhẽ phải là: allow 127.0.0.1; deny all;
}

```

Chọc thử xem nó ọc ra cái lz gì:
```text
root@ip-10-81-64-63:~# curl -s http://MACHINE_IP:8080/nginx_status
Active connections: 1 
server accepts handled requests
 15 15 15 
Reading: 0 Writing: 1 Waiting: 0 

```

Cái đống output này trông thì gọn nhưng dòng thứ hai đéo có lable gì cả, lần đầu nhìn lú cmn luôn.
Ba cái con số vô tri ở dòng thứ 2 theo thứ tự là: tổng số kết nối đã nhận, tổng số kết nối đã xử lý, và tổng số request tính từ lúc bật server. Cái dòng thứ 3 thì bóc tách trạng thái của các kết nối đang active.

Chỗ data này mày đéo thể đâm exploit nổ server được, nhưng nó leak ra đủ thứ thông tin vận hành như tải server hay thói quen sử dụng. Đi pentest thực tế, mò ra cái `/nginx_status` này cũng tính là một con finding ăn tiền, vì nó chứng tỏ hệ thống giám sát nội bộ của tụi nó đang hở sườn, và khả năng cao là còn mấy cái cổng giám sát khác cũng đang bị phơi dái ra.

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
Đm, tiếp tục thông não nào con chó. Tao với mày vừa cày xong 4 con web server khác nhau, mỗi con có một kiểu cấu hình và một rổ những cái ngu đặc trưng riêng. Nhưng đéo có gì là hoàn hảo, có những cái lỗi cơ bản mà server đéo nào cũng dính. Hai cái thứ lặp đi lặp lại nhiều nhất là đéo thèm cấu hình Security Headers và những cái lỗi ngớ ngẩn mà mấy con tool scan tự động quẹt phát là ra ngay.
### Security Headers (Mấy cái cờ bảo mật)
Security headers thực chất là mấy cái response header để ra lệnh cho trình duyệt web biết phải xử lý nội dung trang như thế nào. Bọn nó sinh ra để làm lá chắn chống lại một mớ các đòn tấn công từ phía người dùng (client-side), kiểu như clickjacking (bấm nhầm link độc), sniffing (đọc trộm nội dung), hay XSS (chèn mã độc). Trong cái bài lab lz này, đéo có con server nào được cấu hình để gửi mấy cái header này cả, vì đấy mẹ nó là trạng thái mặc định của cả 4 thằng rồi.

Dưới đây là mấy cái header phổ biến nhất và tác dụng của từng thằng:

| Header                      | Chống lại cái lz gì                                                                                     | Ví dụ minh họa                     |
| --------------------------- | ------------------------------------------------------------------------------------------------------- | ---------------------------------- |
| `X-Frame-Options`           | Chống Clickjacking (ngăn đéo cho web khác nhúng trang của mình vào thẻ iframe)                          | `DENY` hoặc `SAMEORIGIN`           |
| `X-Content-Type-Options`    | Chống MIME sniffing (cấm trình duyệt cầm đèn chạy trước ô tô, tự đoán định dạng file)                   | `nosniff`                          |
| `Content-Security-Policy`   | Giới hạn nghiêm ngặt nguồn tải script, CSS và các tài nguyên khác                                       | `default-src 'self'`               |
| `Referrer-Policy`           | Kiểm soát cái header Referer nôn ra những gì khi user bấm link sang trang khác                          | `no-referrer` hoặc `strict-origin` |
| `Strict-Transport-Security` | Ép buộc trình duyệt phải dùng HTTPS cho các lần truy cập sau (chỉ có tác dụng nếu server đã chạy HTTPS) | `max-age=31536000`                 |

> Thông não: Về mặt kỹ thuật thì thằng `X-Frame-Options` đã bị cho ra rìa bởi lệnh `Content-Security-Policy: frame-ancestors` (vì thằng CSP này kiểm soát chi tiết vcl hơn). Mấy trang web thời nay bảo mật tận răng thường chỉ xài mỗi CSP để chống clickjacking và cố tình vứt cmn thằng `X-Frame-Options` đi. Nên lúc đi test và viết report, nhớ check kỹ cả 2 thằng.

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

Khi thằng `grep` đéo nôn ra chữ nào, tức là con server đéo có cái header bảo mật nào sất. Mày cứ phang thử qua cả 4 port mà xem, cấu hình mặc định của cả 4 loại server đều hở sườn chỗ này. Security headers đéo bao giờ từ trên trời rơi xuống, dev hay admin đéo tự tay cấu hình thì đéo bao giờ có.

> Lưu ý: Cái `Strict-Transport-Security` chỉ có ý nghĩa khi chạy trên HTTPS. Cái lab củ lz này đang chạy HTTP thường, nên đéo có nó là chuyện đương nhiên. Tao nhắc ở đây cho đủ bộ chứ đéo tính là một lỗi cấu hình ngu đâu.

### Càn quét tự động bằng Nikto
Nikto là con tool chuyên cào web server để bới móc mấy cái lỗi cấu hình kinh điển, phần mềm đồ cổ, lộ cổng admin, hay thiếu Security headers. Con hàng này đéo biết nín thở nằm im đâu; nó càn quét ồn ào vãi lz, xả rác traffic ầm ầm và cực kỳ dễ bị tóm. Thế nên nó chỉ hợp mang đi test khi đã xin phép đàng hoàng, chứ đéo hợp để đi đêm ngấm ngầm (stealth). Nã nó thẳng vào con Apache xem:

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

Cái cờ `-nointeractive` là để cấm con tool lải nhải hỏi han, cứ thế mà quét đéo cần mày phải gõ phím xác nhận gì cả. Thằng Nikto sẽ cào hàng trăm cái đường dẫn và quy tắc phổ biến. Chú ý nhìn mấy cái dòng bắt đầu bằng dấu `+` trong kết quả, đấy là đồ ăn được đấy.

Trên một con Apache cấu hình ngu như này, mười mươi là con Nikto sẽ vạch mặt trang `/server-status` tơ hơ, cái file `backup.bak` vứt trong thư mục gốc, cái lỗi phơi danh sách file ở `/files/`, và cả mớ cảnh báo đéo có security headers.

> Mẹo: Output của Nikto thỉnh thoảng dài dòng vãi lz. Mày xài tùy chọn `-Tuning` để ra lệnh cho nó chỉ quét một số lỗi cụ thể thôi. Muốn nhanh gọn thì vã lệnh `nikto -h TARGET -Tuning 123`, nó sẽ hốt hết mấy cái finding phổ biến nhất mà đéo cần cày lại cả bộ từ điển. Nhớ là mấy cái mã số tuning này viết liền tù tì vào nhau, đéo có dấu phẩy cạch mặt đâu.

### Những cái Ngu mang tính Thời Đại (Ở đâu cũng gặp)

Ngó lại cả 4 con server, mày sẽ thấy những cái mảng cấu hình óc chó này lặp đi lặp lại y hệt nhau:

| Lỗi Cấu Hình | Apache | Python HTTP | Node.js | Nginx |
| --- | --- | --- | --- | --- |
| Khai mẹ version ở Header | Có | Có | Một phần | Có |
| Phơi danh sách thư mục | `/files/` | Đường dẫn gốc | Đéo có | `/files/` |
| Lộ trang status/debug | `/server-status` | Đéo có | `/api/debug/env`, `/api/routes` | `/nginx_status` |
| Lộ file nhạy cảm | `backup.bak`, `internal-notes.txt` | `.env`, `notes.txt`, `backup.zip` | `config.js` | `server-config.txt`, `deploy-notes.txt` |
| Đéo có Security Headers | Tất cả | Tất cả | Tất cả | Tất cả |

Cái sợi dây xuyên suốt ở đây là đm, những cái cấu hình mặc định luôn ưu tiên sự "nhanh gọn lẹ" hơn là bảo mật. Lộ version, lộ list thư mục, hay phơi trang status đều được bật mặc định để dễ bắt bệnh hệ thống. Tụi nó làm thế để mấy thằng admin đỡ đau đầu. Muốn gỡ hoặc khóa mõm tụi nó lại thì phải có người tự tay thao tác. Lúc đi làm thực tế, bới ra mấy cái lỗi này đéo hẳn là do thằng admin nó phá hoại, mà đơn giản là do đéo có thằng lz nào thèm đọc lại cái đống cấu hình mặc định từ lúc setup máy đến giờ.

