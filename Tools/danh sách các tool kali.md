Kali Linux nó có hơn 600 tool, bắt liệt kê hết từng cái thì dài như cuốn từ điển và mày cũng chẳng bao giờ xài hết. Nhưng thực tế đi làm pentest hay cày lab/CTF thì chỉ có vài chục con hàng chủ lực ai cũng phải nhẵn mặt.

Tao gom lại cho mày các tool cốt lõi và phổ biến nhất theo từng chuyên môn:

## 1. Thu thập thông tin & Quét mạng

* Nmap: Trùm quét port, phát hiện dịch vụ và hệ điều hành.
* Masscan: Quét port tốc độ cao cho dải IP lớn.
* Amass, theHarvester: Thu thập subdomain, email, thông tin OSINT mục tiêu.
* Wireshark, tcpdump: Bắt và phân tích gói tin mạng theo thời gian thực.

## 2. Tấn công ứng dụng Web

* Burp Suite, OWASP ZAP: Bộ proxy chặn, sửa đổi request và quét lỗ hổng web.
* SQLmap: Tool tự động khai thác lỗi SQL Injection.
* Gobuster, **FFuF:** Quét đường dẫn ẩn, file ẩn, brute-force endpoint cực nhanh.
* WPScan: Chuyên trị quét lỗi trên website chạy mã nguồn WordPress.
* **Nikto:** Quét nhanh các cấu hình lỗi và file rác trên web server.

## 3. Phá mật khẩu & Khai thác mã độc

* Hashcat: Vua phá mật khẩu bằng GPU, hỗ trợ hàng trăm loại hash.
* John the Ripper (JTR): Bẻ khóa hash và trích xuất hash từ các file nén, file hệ thống.
* **Hydra, Medusa:** Tấn công brute-force mật khẩu qua các giao thức mạng như SSH, FTP, RDP.
* **Metasploit Framework:** Nền tảng khai thác lỗ hổng và cắm payload lớn nhất.
* Searchsploit: Tra cứu mã khai thác offline từ cơ sở dữ liệu Exploit-DB.

## 4. Dịch ngược, Điều tra số & Không dây

* Ghidra, Radare2: Dịch ngược mã nguồn và phân tích binary.
* Autopsy, Volatility: Phân tích ảnh đĩa ổ cứng, dump bộ nhớ RAM phục vụ điều tra số.
* Binwalk, Foremost: Trích xuất file ẩn và firmware.
* Aircrack-ng, Wifite: Bộ công cụ bắt gói tin và bẻ khóa mật khẩu Wi-Fi.
* Bettercap, Responder: Đánh chặn, giả mạo mạng nội bộ (MitM, LLMNR/NBT-NS poisoning).

---

Nếu mày muốn xem danh sách chính xác từng gói cài đặt đang có sẵn trên máy của mày, mở terminal gõ thẳng lệnh này:

`dpkg --get-selections | grep -v deinstall`

Muốn xem tài liệu chi tiết của từng tool kèm ví dụ chạy lệnh thì cứ vào trang chủ `kali.org/tools/` mà tra cứu cho chuẩn bài. Mày đang cần cày sâu vào mảng nào trong đống trên?
