# Bộ câu hỏi ôn tập phỏng vấn (Bản mở rộng) — Vị trí Web Pentester / Penetration Tester

> Dựa trên CV của Nguyễn Đức Tài. Trọng tâm ôn tập: **PortSwigger Web Security Academy (70%+ lab đã hoàn thành), TryHackMe, và các cuộc thi CTF (PTIT CTF, picoCTF)** — vì đây là phần nhà tuyển dụng nhiều khả năng xoáy sâu nhất để kiểm tra thực lực.

---

# PHẦN A — CÂU HỎI CHUNG (NGẮN GỌN)

## A1. Giới thiệu bản thân & định hướng nghề nghiệp
1. Giới thiệu bản thân trong 2-3 phút, tập trung vào phần liên quan đến vị trí ứng tuyển.
2. Vì sao bạn chọn Penetration Testing thay vì SOC/DevSecOps/Forensics chuyên sâu?
3. Vì sao bạn muốn làm ở môi trường ISP? Pentest hạ tầng ISP khác gì so với web application thông thường theo bạn?
4. Bạn biết gì về NetNam / công ty ứng tuyển?
5. Mục tiêu 1 năm, 3 năm tới của bạn là gì?
6. Bạn cân bằng thời gian giữa học năm 3, thực tập và tự học bảo mật (PortSwigger, THM, CTF) như thế nào?

## A2. Kinh nghiệm thực tập tại G-Innovations Việt Nam
1. Mô tả công việc hằng ngày khi làm Web Pentester Intern.
2. Kể một lỗ hổng cụ thể bạn từng phát hiện (Authentication/Session/Access control) — cách phát hiện, mức độ nghiêm trọng, cách fix đề xuất.
3. Quy trình kiểm thử web/API của bạn từ đầu đến cuối (recon → scan → exploit → report) diễn ra thế nào?
4. Bạn đánh giá mức độ rủi ro dựa trên tiêu chí gì? Có biết CVSS không, tính điểm CVSS cơ bản ra sao?
5. Báo cáo kỹ thuật của bạn gồm những phần nào để đội dev dễ hiểu và khắc phục?
6. Bạn dùng Nmap quét hạ tầng — các kỹ thuật/tham số quét bạn hay dùng (SYN scan, service/version detection, script scan, timing)?
7. Khó khăn lớn nhất trong 7 tháng thực tập là gì, xử lý ra sao?
8. Sự khác biệt lớn nhất giữa pentest thực tế (production/staging) và luyện tập trên lab/CTF là gì?

---

# PHẦN B — TRỌNG TÂM: ÔN TẬP THEO TỪNG MODULE PORTSWIGGER WEB SECURITY ACADEMY

> Đây là phần quan trọng nhất. Với mỗi chủ đề, nhà tuyển dụng thường hỏi theo 4 lớp: **(1) Định nghĩa/bản chất → (2) Cách phát hiện → (3) Cách khai thác/leo thang → (4) Cách phòng chống.** Hãy tự trả lời đủ 4 lớp cho từng mục dưới đây.

## B1. SQL Injection
1. Phân biệt In-band (Union-based, Error-based), Blind (Boolean-based, Time-based), và Out-of-band SQLi. **
2. Cách xác định số lượng cột trong UNION attack khi không biết cấu trúc bảng (`ORDER BY`, `UNION SELECT NULL,NULL...`)? *ok*
3. Khi ứng dụng không trả lỗi và không có sự khác biệt nội dung, bạn xác định blind SQLi bằng cách nào (time-based: `SLEEP()`, `WAITFOR DELAY`)? 
4. SQLi trong các ngữ cảnh khác ngoài câu lệnh SELECT (trong ORDER BY, trong LIMIT, trong INSERT/UPDATE) khai thác khác nhau ra sao?
5. Second-order SQL Injection là gì, vì sao khó phát hiện hơn first-order?
6. Cách khai thác SQLi để đọc file hệ thống hoặc ghi file (nếu DB có quyền, ví dụ `INTO OUTFILE` trên MySQL)?
7. Cách bypass WAF cơ bản khi test SQLi (encoding, comment injection, case manipulation)?
8. Phòng chống SQLi hiệu quả nhất: Parameterized query/Prepared statement, ORM, least privilege DB account — giải thích vì sao input sanitization/blacklist không đủ an toàn.

## B2. Cross-Site Scripting (XSS)
1. Phân biệt Reflected, Stored, DOM-based XSS — cho ví dụ luồng dữ liệu (source → sink) của từng loại.
2. DOM-based XSS: các "sink" nguy hiểm phổ biến là gì (`innerHTML`, `document.write`, `eval`, `location`)?
3. Cách phát hiện XSS khi input bị lọc/encode một phần — kỹ thuật bypass filter cơ bản (đổi tag, đổi event handler, encoding).
4. CSP (Content Security Policy) hoạt động ra sao, và những cách CSP có thể bị bypass (unsafe-inline, JSONP endpoint, whitelisted domain bị lợi dụng)?
5. XSS trong các ngữ cảnh khác nhau (trong thuộc tính HTML, trong JavaScript string, trong URL) đòi hỏi payload khác nhau thế nào?
6. Từ một lỗ hổng XSS, bạn có thể leo thang tấn công gì (đánh cắp cookie/session, keylogging, CSRF token theft, chiếm quyền tài khoản)?
7. Vì sao `HttpOnly` và `Secure` cookie flag lại giảm thiểu rủi ro từ XSS?

## B3. CSRF (Cross-Site Request Forgery)
1. Điều kiện để một request dễ bị tấn công CSRF là gì?
2. Cơ chế CSRF token hoạt động ra sao, và những lỗi hay gặp khi triển khai token khiến vẫn bị bypass (token không gắn với session, token có thể tái sử dụng, thiếu kiểm tra ở backend)?
3. SameSite cookie (Strict/Lax/None) giúp giảm CSRF như thế nào? Hạn chế của nó là gì?
4. CSRF trên request dùng JSON body có dễ khai thác như form thường không? Vì sao?

## B4. Access Control Vulnerabilities (Broken Access Control / IDOR)
1. Phân biệt Horizontal Privilege Escalation và Vertical Privilege Escalation.
2. IDOR là gì? Cách bạn dò IDOR có hệ thống khi test API (thay đổi ID tuần tự, UUID, thay đổi tham số ẩn)?
3. "URL-matching discrepancies" trong access control (ví dụ path case-sensitivity, thêm ký tự `/./`, `..;/`) có thể dẫn tới bypass như thế nào?
4. Multi-step process với access control — vì sao chỉ kiểm tra quyền ở bước đầu mà không kiểm tra lại ở các bước sau là một lỗ hổng?
5. Access control dựa trên request method (ví dụ chỉ chặn GET mà không chặn POST) có rủi ro gì?

## B5. Authentication Vulnerabilities
1. Kỹ thuật brute-force cơ bản và cách xác định cơ chế chống brute-force (rate limiting, account lockout) có "flawed" hay không (ví dụ reset counter theo IP, không khóa khi dùng đúng username sai password nhiều lần).
2. Username enumeration xảy ra như thế nào (thông báo lỗi khác nhau giữa "sai username" và "sai password", thời gian phản hồi khác nhau)?
3. Flawed 2FA — các lỗi triển khai phổ biến (không kiểm tra lại session sau khi nhập đúng mật khẩu bước 1, mã OTP có thể brute-force, có thể bỏ qua bước 2FA bằng cách sửa response/status)?
4. Broken logic trong luồng "quên mật khẩu" (password reset token không random đủ mạnh, token không hết hạn, token lộ qua Referer header)?
5. JWT trong authentication: các lỗ hổng phổ biến — `alg=none`, thuật toán confusion (RS256 → HS256), secret yếu để brute-force, header `kid` bị inject để đọc file/SQLi.
6. Session sau khi đăng nhập — những sai sót phổ biến (session không đổi ID sau login → session fixation, session không hết hạn khi logout)?

## B6. File Upload Vulnerabilities
1. Các kỹ thuật bypass kiểm tra extension (double extension `.php.jpg`, null byte `.php%00.jpg`, viết hoa/thường, thêm khoảng trắng/dấu chấm cuối tên file)?
2. Bypass kiểm tra Content-Type/MIME type như thế nào (chỉnh sửa header trong Burp Repeater)?
3. Bypass kiểm tra "magic bytes" (thêm header ảnh hợp lệ trước payload PHP) hoạt động ra sao?
4. Sau khi upload file webshell thành công, làm sao xác định đường dẫn để truy cập và thực thi?
5. Nếu server không cho thực thi file trong thư mục upload, còn cách khai thác nào khác (upload file `.htaccess` để đổi cấu hình, path traversal khi upload để ghi đè file khác)?

## B7. SSRF (Server-Side Request Forgery)
1. SSRF là gì? Phân biệt SSRF cơ bản và <u>blind SSRF</u>.
2. Kịch bản khai thác thực tế: truy cập cloud metadata endpoint (`169.254.169.254`) để lấy credentials — vì sao nguy hiểm?
3. Cách bypass whitelist/blacklist domain trong SSRF (dùng redirect, DNS rebinding, IP dạng số thập phân/octal/hex, `localhost` vs `127.0.0.1` vs `0.0.0.0`)?
4. SSRF qua các tính năng gián tiếp (webhook, PDF generator, import URL, image fetch từ URL) khai thác khác gì so với tham số URL trực tiếp?
5. Cách phòng chống SSRF hiệu quả (network-level whitelist, disable redirect follow, không dùng blacklist).

## B8. XXE (XML External Entity) Injection
1. Điều kiện để một endpoint xử lý XML dễ bị XXE?
2. Payload XXE cơ bản để đọc file hệ thống (`<!ENTITY xxe SYSTEM "file:///etc/passwd">`)?
3. XXE có thể dẫn tới SSRF như thế nào?
4. Blind XXE (không có phản hồi trực tiếp) khai thác qua Out-of-band (OOB) hoặc error-based ra sao?
5. Cách phòng chống XXE (disable external entity/DTD processing trong XML parser)?

## B9. SSTI (Server-Side Template Injection)
1. SSTI khác XSS ở điểm nào — vì sao nó nguy hiểm hơn (thực thi ở server, có thể dẫn tới RCE)?
2. Cách nhận biết một endpoint dễ bị SSTI — payload polyglot cơ bản để test (`{{7*7}}`, `${7*7}`, `<%= 7*7 %>`) tùy engine.
3. Với mỗi template engine phổ biến (Jinja2, Twig, FreeMarker, Velocity...), payload leo thang lên RCE khác nhau thế nào? Bạn từng thử engine nào?
4. Cách xác định template engine đang dùng khi không biết trước (dựa vào lỗi trả về, dựa vào cú pháp phản hồi)?

## B10. Command Injection & Path Traversal
1. OS Command Injection: cách chèn thêm lệnh vào input (`; ls`, `| whoami`, `` `id` ``, `$(id)`) tùy hệ điều hành/shell.
2. Blind command injection phát hiện qua kỹ thuật nào (time delay, out-of-band DNS/HTTP callback)?
3. Path Traversal: payload cơ bản (`../../../etc/passwd`), cách bypass filter (encoding `%2e%2e/`, double encoding, null byte, absolute path)?

## B11. Business Logic Vulnerabilities
1. Business logic vulnerability khác lỗi kỹ thuật (SQLi, XSS...) ở điểm nào? Cho ví dụ bạn từng gặp hoặc học trong lab.
2. Ví dụ về lỗi logic phổ biến: bỏ qua bước xác thực trong quy trình nhiều bước, thao túng giá trị (giá sản phẩm, số lượng âm), race condition trong quy trình thanh toán/đổi điểm.

## B12. Information Disclosure
1. Các nguồn thông tin nhạy cảm dễ bị lộ (comment trong HTML/JS, file backup `.bak`/`.old`, thông báo lỗi verbose, endpoint debug, `robots.txt`, version control `.git` lộ ra web root)?
2. Cách bạn dò tìm information disclosure có hệ thống trong một bài test (Dirsearch/Gobuster, xem source, xem response header)?

## B13. Race Conditions
1. Race condition trong web là gì, vì sao nó khó phát hiện và khai thác?
2. Ví dụ kịch bản khai thác (redeem coupon nhiều lần, chuyển tiền vượt số dư) bằng cách gửi request đồng thời (Burp Turbo Intruder)?

## B14. Web Cache Poisoning
1. Web cache poisoning là gì? "Cache key" và "unkeyed input" liên quan thế nào đến lỗ hổng này?
2. Cách khai thác qua header không được đưa vào cache key (ví dụ `X-Forwarded-Host`) để chèn nội dung độc hại vào cache dùng chung cho nhiều người dùng?

## B15. Insecure Deserialization
1. Vì sao Insecure Deserialization nguy hiểm (có thể dẫn tới RCE)?
2. Với PHP object injection / Java deserialization (nếu bạn có tìm hiểu), cách nhận biết endpoint nhận dữ liệu serialized và hướng khai thác cơ bản là gì (gadget chain)?

## B16. CORS Misconfiguration
1. CORS là gì, khác gì với Same-Origin Policy?
2. Cấu hình CORS sai phổ biến (`Access-Control-Allow-Origin: *` kèm `Access-Control-Allow-Credentials: true`, reflect Origin header không kiểm tra) dẫn tới rủi ro gì?

## B17. Clickjacking
1. Clickjacking là gì, khai thác bằng cách nào (iframe ẩn)?
2. Cách phòng chống (`X-Frame-Options`, CSP `frame-ancestors`)?

## B18. HTTP Request Smuggling & Host Header Attacks
1. Request smuggling là gì? Phân biệt CL.TE và TE.CL.
2. Vì sao nó nguy hiểm trong kiến trúc có front-end/back-end server hoặc load balancer + origin server?
3. HTTP Host header injection có thể bị lợi dụng cho những tấn công gì (password reset poisoning, cache poisoning, routing-based SSRF)?

## B19. GraphQL & API-specific Vulnerabilities
1. Những rủi ro đặc thù của GraphQL API (introspection bị bật ở production, excessive data exposure, batching attack cho brute force)?
2. Khi test REST API, ngoài OWASP Top 10 thông thường bạn còn kiểm tra thêm gì (rate limiting, mass assignment, versioning cũ còn lỗ hổng)?

## B20. OAuth Authentication
1. Luồng OAuth Authorization Code cơ bản là gì?
2. Các lỗi triển khai OAuth phổ biến (redirect_uri không được validate chặt, thiếu kiểm tra `state` parameter dẫn tới CSRF, token bị lộ qua Referer)?

## B21. WebSockets
1. Kiểm thử bảo mật WebSocket khác HTTP thường ở điểm nào?
2. Cross-Site WebSocket Hijacking là gì, điều kiện để khai thác được?

---

# PHẦN C — TRYHACKME: ÔN TẬP THEO KỸ NĂNG

> Nhà tuyển dụng có thể hỏi theo hướng thực hành/tư duy hơn là lý thuyết thuần túy, vì TryHackMe thiên về mô phỏng máy thật.

## C1. Nền tảng & Recon
1. Quy trình enumeration ban đầu khi được giao một target mới (thường bắt đầu từ đâu: Nmap → xác định service → tra CVE theo version)?
2. Bạn dùng Nmap để phát hiện các service/port mở, sau đó ưu tiên kiểm tra port/service nào trước và vì sao?
3. Gobuster/Dirsearch/ffuf dùng để làm gì, khi nào bạn chọn wordlist nào (common.txt, SecLists, custom theo ngữ cảnh)?
4. Bạn từng dùng Hydra/Burp Intruder để brute-force đăng nhập chưa? Khi nào brute-force là hướng đi hợp lý trong một bài CTF/THM?

## C2. Web Fundamentals trên TryHackMe
1. Sự khác biệt giữa việc pentest một trang trên THM (thường có 1-2 lỗ hổng "chủ đích") và một ứng dụng thực tế nhiều lớp phòng thủ?
2. Bạn từng gặp phòng nào yêu cầu kết hợp nhiều lỗ hổng liên tiếp (chain vulnerabilities) để đạt được root/admin chưa? Kể lại luồng tấn công.

## C3. Privilege Escalation cơ bản (Linux/Windows)
1. Sau khi có shell với quyền thấp trên Linux, bạn thường kiểm tra gì để leo thang đặc quyền (sudo -l, SUID binaries, cron jobs, kernel version, PATH hijacking)?
2. Trên Windows, các hướng leo thang đặc quyền cơ bản bạn biết là gì (misconfigured service, unquoted service path, scheduled task, AlwaysInstallElevated)?
3. GTFOBins là gì và bạn dùng nó như thế nào?

## C4. Networking & Hệ điều hành thực hành
1. Reverse shell và bind shell khác nhau thế nào, khi nào bạn chọn loại nào?
2. Bạn setup listener (netcat) và catch reverse shell như thế nào? Có gặp vấn đề shell không ổn định (không có TTY đầy đủ) không, cách xử lý (`python -c 'import pty;pty.spawn("/bin/bash")'`)?
3. Bạn hiểu gì về pivoting/port forwarding cơ bản khi target chỉ truy cập được từ một máy trung gian?

---

# PHẦN D — CTF (PTIT CTF & PICOCTF): ÔN TẬP THEO CATEGORY

## D1. Web Exploitation (trọng tâm chính của bạn)
1. Trong CTF, làm sao bạn nhanh chóng xác định loại lỗ hổng "chủ đích" của một challenge Web (dựa vào source code được cho, dựa vào hint, dựa vào cấu trúc ứng dụng)?
2. Kể lại chi tiết một challenge Web bạn từng giải ở PTIT CTF vòng chung kết — luồng suy luận từ lúc đọc đề đến lúc lấy được flag.
3. Khi được cấp source code (white-box) thay vì black-box, chiến lược tìm lỗ hổng của bạn thay đổi thế nào?
4. Bạn có kinh nghiệm với challenge kết hợp nhiều lỗ hổng (ví dụ SSTI → RCE → đọc flag, hoặc SQLi → lấy credentials → login admin → IDOR) không?

## D2. Digital Forensics
1. Với một file pcap (network capture) trong CTF, bạn dùng Wireshark như thế nào để tìm flag (follow TCP/HTTP stream, filter theo protocol, export objects)?
2. Với memory dump, quy trình phân tích của bạn là gì (Volatility hoặc công cụ tương tự: xác định profile, liệt kê process, dump process nghi ngờ)?
3. Phân tích file log để tìm dấu vết tấn công — bạn tìm kiếm pattern gì (request bất thường, user-agent lạ, mã lỗi lặp lại, timestamp bất thường)?
4. Steganography cơ bản — bạn kiểm tra file ảnh/audio nghi có giấu dữ liệu bằng công cụ/kỹ thuật nào (exiftool, binwalk, strings, LSB analysis)?
5. Phân tích file không rõ định dạng, bạn xác định loại file thật sự bằng cách nào (magic bytes, `file` command, hex editor)?

## D3. Cryptography (cơ bản, thường có trong picoCTF)
1. Bạn phân biệt được mã hóa đối xứng và bất đối xứng không, cho ví dụ thuật toán?
2. Với cipher cổ điển (Caesar, XOR, Base encoding nhiều lớp), cách bạn nhận diện và giải mã?
3. Bạn từng gặp challenge khai thác lỗi triển khai crypto (weak key, reuse nonce, ECB mode pattern) chưa?

## D4. Reverse Engineering & Binary/Pwn (nếu có tiếp xúc qua picoCTF)
1. Bạn có kinh nghiệm dùng công cụ nào để phân tích binary không (Ghidra, IDA, strings, file, checksec)?
2. Buffer overflow cơ bản là gì, bạn hiểu ở mức khái niệm hay đã thực hành exploit đơn giản chưa?
3. Nếu chưa mạnh mảng này, hãy chuẩn bị câu trả lời trung thực: bạn tự đánh giá thế mạnh của mình nằm ở Web/Forensics, và đang có kế hoạch học thêm Reverse/Pwn như thế nào?

## D5. OSINT & Misc
1. Bạn từng giải challenge OSINT (tìm thông tin từ ảnh, mạng xã hội, metadata) chưa? Quy trình ra sao?
2. Trong các challenge "Misc", kỹ năng tổng hợp nào giúp bạn giải quyết vấn đề nhanh (đọc kỹ đề, thử nhiều công cụ, tra cứu theo từ khóa lạ)?

## D6. Tư duy & phương pháp thi CTF nói chung
1. Khi bí một challenge, bạn có quy trình xử lý thế nào (đổi hướng tiếp cận, tạm bỏ qua rồi quay lại, tra cứu writeup tương tự)?
2. Làm việc nhóm trong CTF — bạn phân chia category giữa các thành viên như thế nào?
3. Sau khi thi xong, bạn có thói quen đọc writeup của những challenge chưa giải được không? Cho ví dụ một bài học rút ra gần đây.
4. Bạn quản lý thời gian trong một trận CTF dài (nhiều giờ/nhiều ngày) như thế nào để không bị "mắc kẹt" quá lâu ở một câu?

---

# PHẦN E — CÔNG CỤ KIỂM THỬ (ĐÀO SÂU)

1. Burp Suite: bạn dùng Repeater, Intruder (4 loại attack: Sniper, Battering ram, Pitchfork, Cluster bomb), Sequencer, Decoder, Comparer như thế nào? Cho ví dụ tình huống cụ thể dùng Intruder.
2. Burp Extension nào bạn từng dùng (nếu có): Turbo Intruder, Autorize, Param Miner...?
3. So sánh Burp Suite Community vs Pro — bạn thấy giới hạn nào của bản Community khi làm việc thực tế?
4. OWASP ZAP: bạn dùng Automated Scan hay Manual Explore nhiều hơn? So sánh với Burp?
5. Dirsearch/Gobuster/ffuf: sự khác biệt về tốc độ, tính năng (ffuf hỗ trợ fuzz nhiều vị trí cùng lúc)?
6. SQLMap: các tham số bạn hay dùng (`--dbs`, `--tables`, `--dump`, `--risk`, `--level`, `--tamper` để bypass WAF)?
7. Nessus: quy trình bạn chạy một bản scan và đọc report như thế nào, phân biệt false positive?
8. Metasploit: bạn đã dùng module exploit có sẵn hay tự viết chưa? Kể một lần dùng msfconsole trong lab/CTF.
9. Wireshark: filter cơ bản bạn hay dùng (`http`, `tcp.port==`, `ip.addr==`, follow stream)?
10. Postman: bạn dùng để test API như thế nào trong một buổi pentest (import Swagger/OpenAPI, test theo collection, kiểm tra auth header)?

---

# PHẦN F — MẠNG & HỆ ĐIỀU HÀNH (ĐÀO SÂU)

1. Trình bày mô hình TCP/IP, so sánh với OSI 7 tầng.
2. TLS handshake diễn ra qua các bước nào? Vì sao HTTPS an toàn hơn HTTP?
3. DNS hoạt động ra sao, các kiểu tấn công liên quan (DNS spoofing, cache poisoning, subdomain takeover)?
4. TCP vs UDP khác nhau thế nào, ví dụ giao thức dùng mỗi loại?
5. Bạn thao tác trên Kali Linux/Ubuntu hằng ngày như thế nào trong công việc pentest? Lệnh/kỹ thuật bạn hay dùng?
6. Kiến thức về Windows/Active Directory bạn có (nếu có): SMB enumeration, kiểm tra user/group, các lỗ hổng AD cơ bản (nếu từng tìm hiểu)?
7. Sự khác biệt giữa quét mạng nội bộ (internal) và quét từ Internet (external) về phạm vi và rủi ro?

---

# PHẦN G — DIGITAL FORENSICS (ĐÀO SÂU THÊM)

1. Chain of custody trong forensics là gì, vì sao quan trọng kể cả trong bối cảnh CTF/học thuật?
2. Bạn phân biệt volatile data và non-volatile data như thế nào, thứ tự thu thập bằng chứng ưu tiên ra sao (order of volatility)?
3. Với một memory dump, những cấu trúc dữ liệu nào bạn quan tâm nhất (process list, network connections, loaded DLLs/modules, command history)?
4. Khi phân tích log server (access log/error log), bạn tìm dấu hiệu tấn công web như thế nào (payload trong query string, status code bất thường, user-agent của scanner tự động)?

---

# PHẦN H — DỰ ÁN CÁ NHÂN

**Trang web PHP chứa lỗ hổng (file upload, Insecure Design, Authentication Bypass)**
1. Vì sao bạn xây dựng dự án này, mục đích học tập cụ thể là gì?
2. Bạn thiết kế từng lỗ hổng như thế nào để mô phỏng đúng bản chất lỗi thực tế (không chỉ "cho vui")?
3. Bạn có viết writeup/tài liệu hướng dẫn khai thác cho từng lỗ hổng không? Có publish lên GitHub không?

**Web quản lý bán hàng (Python)**
1. Kiến trúc dự án (framework, database) ra sao?
2. Bạn có áp dụng biện pháp bảo mật nào khi xây dựng (input validation, hashing password, parameterized query) không?

**Pomodoro timer cho Linux (Python)**
1. Dự án giải quyết vấn đề gì, dùng thư viện/công nghệ nào?

**Deepfake detector (Python)**
1. Bạn dùng kỹ thuật/mô hình gì để phát hiện deepfake?
2. Dự án này có liên hệ gì tới định hướng bảo mật của bạn không, hay đơn thuần là dự án học máy cá nhân?

---

# PHẦN I — TÌNH HUỐNG THỰC TẾ / CASE STUDY

1. Cho một đoạn HTTP request, chỉ ra những điểm nghi ngờ có thể tồn tại lỗ hổng.
2. Nếu phát hiện lỗ hổng nghiêm trọng (RCE) khi đang test trên môi trường production, bạn xử lý ra sao (dừng khai thác, báo cáo khẩn cấp, không tự ý leo thang thêm)?
3. Đội dev không đồng ý với đánh giá rủi ro của bạn — bạn thuyết phục họ thế nào (dẫn chứng CVSS, PoC, tài liệu OWASP)?
4. Chỉ có 2 giờ để test một ứng dụng hoàn toàn mới, bạn ưu tiên kiểm tra gì trước?
5. Công cụ tự động (Burp/ZAP/Nessus) báo lỗi nhưng bạn nghi là false positive — bạn xác minh lại bằng cách nào?
6. Khách hàng yêu cầu bạn test nhưng phạm vi (scope) không rõ ràng — bạn xử lý thế nào trước khi bắt đầu?

---

# PHẦN J — CÂU HỎI HÀNH VI (BEHAVIORAL)

1. Kể một lần bạn gặp bế tắc trong học tập/CTF/thực tập và cách vượt qua.
2. Bạn học kiến thức mới bằng cách nào (blog, lab, cộng đồng)? Kể tên nguồn bạn hay theo dõi (PortSwigger blog, HackTricks, Twitter/X security researchers...).
3. Làm việc nhóm trong CTF/dự án — bạn đóng vai trò gì, xử lý bất đồng ra sao?
4. Bạn ưu tiên công việc thế nào khi có nhiều deadline cùng lúc?
5. Điểm mạnh, điểm yếu của bạn là gì? Bạn đang cải thiện điểm yếu ra sao?
6. Bạn phản ứng thế nào khi bị chỉ ra lỗi sai trong báo cáo kỹ thuật của mình?

---

# PHẦN K — CÂU HỎI VỀ MONG MUỐN & CÂU HỎI NGƯỢC LẠI

1. Mức lương/chế độ thực tập bạn mong muốn?
2. Bạn có thể làm full-time/part-time trong khung thời gian nào, có ảnh hưởng lịch học không?
3. Chuẩn bị sẵn 2-3 câu hỏi ngược lại cho nhà tuyển dụng, ví dụ:
   - Quy trình pentest nội bộ của công ty hiện tại như thế nào?
   - Công ty có hỗ trợ học/thi chứng chỉ (OSCP, eJPT, CEH...) không?
   - Lộ trình phát triển của một Pentester Intern sau khi kết thúc thời gian thử việc là gì?
   - Team Security của công ty làm việc phối hợp với team Network/Infra (đặc thù ISP) như thế nào?

---

# GỢI Ý ÔN TẬP NHANH TRƯỚC NGÀY PHỎNG VẤN

- **PortSwigger (ưu tiên số 1):** Với mỗi chủ đề ở Phần B, tự viết ra giấy: định nghĩa 1 câu, payload ví dụ, 1 cách bypass, 1 cách phòng chống. Nếu có lab nào làm rồi nhưng quên chi tiết, nên vào lại portswigger.net làm nhanh lại trong 1-2 ngày trước phỏng vấn.
- **TryHackMe:** Ôn lại các room bạn từng làm thuộc path Web Fundamentals/Jr Penetration Tester — chú ý phần bạn dùng Burp, phần privilege escalation cơ bản.
- **CTF:** Chuẩn bị sẵn 2-3 câu chuyện chi tiết theo mô hình STAR (Situation-Task-Action-Result) từ PTIT CTF — đây thường là phần bạn tự tin nhất nên khai thác triệt để khi phỏng vấn.
- **Mạng cơ bản:** Vì ứng tuyển vào ISP, hãy chắc chắn nắm vững TCP/IP, HTTP/HTTPS, DNS, TLS — nhiều khả năng được hỏi sâu hơn so với công ty phần mềm thông thường.
- **GitHub cá nhân:** Xem lại README/code tại github.com/taind345, sẵn sàng giải thích bất kỳ dự án nào bất kỳ lúc nào.
- **Luyện nói to thành tiếng:** Tránh học thuộc lòng — nhà tuyển dụng thường hỏi xoáy sâu (follow-up) vào một câu trả lời để kiểm tra mức độ hiểu bản chất, không phải hiểu bề mặt.
