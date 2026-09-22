## 1-intro
cyber killchain gồm:
- **Reconnaissance (Trinh sát):** Thu thập thông tin mục tiêu.
    
- **Weaponisation (Vũ khí hóa):** Tạo mã độc/payload khớp với lỗ hổng của mục tiêu.
    
- **Delivery (Phát tán):** Gửi payload đến nạn nhân (phishing email, web độc hại...).
    
- **Exploitation (Khai thác):** Kích hoạt mã độc để thực thi lỗi trên hệ thống.
    
- **Installation (Cài đặt):** Cài backdoor/malware để duy trì quyền truy cập lâu dài.
    
- **Command & Control / C2 (Điều khiển):** Thiết lập kênh điều khiển hệ thống bị chiếm quyền từ xa.
    
- **Actions on Objectives (Thực hiện mục tiêu):** Tiến hành mục đích cuối cùng (đánh cắp dữ liệu, phá hoại, tấn công sang hệ thống khác...).

## 2-reconise
<u> 1. Phân loại Trinh sát</u>
- **Passive (Thụ động):** Thu thập thông tin không tương tác trực tiếp, không tạo tiếng ồn (noise).
    - _Ví dụ:_ Tra cứu *WHOIS, DNS, Google Dorking,* cào dữ liệu website, khai thác mạng xã hội.
- **Active (Chủ động):** Tương tác trực tiếp với mục tiêu, dễ bị hệ thống phát hiện.
    - _Ví dụ:_ Quét cổng mạng (Port scan), quét lỗ hổng, kỹ thuật xã hội (Social engineering), trinh sát thực địa.

<u> 2. Biện pháp Phòng thủ</u>
- **Tối thiểu hóa thông tin công khai:** Bật ẩn thông tin WHOIS, hạn chế chi tiết nhạy cảm trên DNS, website và mạng xã hội.
- **Giám sát & Phân tích:** Theo dõi lưu lượng mạng và kiểm tra log dịch vụ thường xuyên để phát hiện hành vi quét cổng/lỗ hổng.
 
## 3-wepionisation
Tóm tắt giai đoạn Weaponisation (Vũ khí hóa):
1.<u> Khái niệm & Kỹ thuật</u>
- Bản chất: Đóng gói mã khai thác (exploit) thành "vũ khí mạng" hoàn chỉnh dựa trên thông tin thu thập từ bước Trinh sát.
- Kỹ thuật: Dùng Obfuscation (làm mờ code), mã hóa hoặc ẩn mã độc vào file thông dụng (Word, PDF, USB) để né tránh kiểm tra.
- Mục tiêu: Tạo ra một file/payload độc hại hoàn chỉnh sẵn sàng cho khâu phát tán.
*--> bước là tạo payload; sau đó tìm cách che giáu nó , để có thể tuồn vào mục tiêu*

<u> 2. Biện pháp Phòng thủ</u>
- Đào tạo người dùng: Hướng dẫn kiểm tra nguồn gốc email, cảnh giác với file đính kèm hoặc file ZIP nén có mật khẩu.
- Thắt chặt cấu hình (Hardening):
    - Vô hiệu hóa Macro trong Office (hoặc chỉ cho phép nguồn tin cậy) qua Group Policy.
    - Tắt/gỡ bỏ các plugin, phần mềm và tính năng không cần thiết để thu hẹp diện tấn công (_attack surface_).

## 4-Delivery
 1-hình thức phát tán
- **email lừa đảo (phishing / spear phishing):** giả mạo người quen hoặc cấp trên, kèm file đính kèm mã độc (ví dụ `invoice.pdf.exe`) hoặc link lừa đảo.
- **liên kết độc hại:** dùng web giả mạo, rút gọn url hoặc quảng cáo độc hại (malvertising) để chuyển hướng người dùng.
- **nền tảng chia sẻ file:** tải file chứa mã độc lên các dịch vụ lưu trữ phổ biến nhằm lợi dụng sự tin tưởng.
- **sms (smishing) & cuộc gọi (vishing):** lừa nạn nhân bấm vào link hoặc tải phần mềm qua tin nhắn/cuộc gọi.
- **phương tiện vật lý:** bỏ quên usb chứa mã độc ở nơi công cộng hoặc gửi đĩa dvd giả mạo catalogue.

 2-biện pháp phòng thủ
- **đào tạo nhận thức:** huấn luyện người dùng nhận biết email lừa đảo và thói quen duyệt web an toàn.
- **bộ lọc email & web:** tự động quét và chặn các file, link độc hại từ email và lưu lượng web.
- **trang bị waf:** sử dụng tường lửa ứng dụng web để ngăn chặn mã độc phát tán qua web.
- **giám sát & quản lý bản vá:** theo dõi lưu lượng mạng liên tục và cập nhật bản vá hệ thống kịp thời.

## 5-exploitation
tóm tắt giai đoạn **exploitation (khai thác)**:
 1- hình thức khai thác
- **tấn công xác thực:** đoán mật khẩu yếu/mặc định hoặc lừa lấy thông tin đăng nhập của người dùng.
- **lỗ hổng phần mềm:** khai thác lỗi ứng dụng/dịch vụ mạng (injection, buffer overflow...) hoặc sử dụng zero-day exploit.
- **cấu hình sai:** lợi dụng sơ hở trong thiết lập hệ thống để chiếm quyền truy cập mà không cần tài khoản.

 2- biện pháp phòng thủ
- **xác thực mạnh:** bắt buộc chính sách mật khẩu phức tạp và triển khai mfa (xác thực nhiều yếu tố).
- **quản lý bản vá & rà quét:** cập nhật bản vá hệ thống thường xuyên và quét lỗ hổng định kỳ.
- **trang bị ips & waf:** sử dụng ips để chặn lưu lượng chứa mã khai thác và waf để chống các cuộc tấn công ứng dụng web (sqli, xss, csrf).

## 6-installation
tóm tắt giai đoạn **installation (cài đặt / duy trì thâm nhập)**:
 1- hình thức duy trì
- **bản chất:** đảm bảo duy trì quyền truy cập lâu dài (**persistence**) để không cần khai thác lại.
- **tạo tác vụ định kỳ:** dùng <u>scheduled task</u> (windows) hoặc <u>cron job</u> (linux).
- **cài đặt dịch vụ:** tạo service/daemon mới hoặc chỉnh sửa script khởi động hệ thống.
- **công cụ độc hại:** cài <u>malware, backdoor, rootkit </u>hoặc tận dụng các công cụ hợp lệ có sẵn trên hệ thống (lolbins).
- **web shell:** triển khai script độc hại để điều khiển qua giao diện web (thường chạy qua https để ngụy trang).

 2- biện pháp phòng thủ
- **giám sát tiến trình:** theo dõi tiến trình/dịch vụ mới và triển khai edr để phát hiện bất thường trên thiết bị đầu cuối.
- **kiểm toán hệ thống:** rà soát định kỳ dựa trên cấu hình chuẩn (baseline) để phát hiện tài khoản hoặc dịch vụ lạ.
- **allowlisting:** áp dụng danh sách cho phép (application allowlisting) để ngăn phần mềm không rõ nguồn gốc khởi chạy.

## 7-command & control / c2
tóm tắt giai đoạn command & control / c2 (điều khiển & khống chế):
1- hình thức & kỹ thuật
*reverse shell cũng là 1 trong số đó*
- bản chất: thiết lập kênh liên lạc ẩn (covert channel) giữa máy nạn nhân và hạ tầng của kẻ tấn công để truyền lệnh và dữ liệu.
- lợi dụng giao thức chuẩn: sử dụng http, https, dns, icmp hoặc k*ỹ thuật dns tunneling* để trộn lẫn lưu lượng độc hại vào lưu lượng mạng hợp lệ.
- lợi dụng dịch vụ uy tín: dùng tin nhắn mạng xã hội (như x/twitter) hoặc dịch vụ cloud (google docs, dropbox) để gửi lệnh và rút dữ liệu.
- <u>kỹ thuật chống bị gỡ bỏ:</u>
    - *dga* (domain generation algorithm): dùng thuật toán sinh ra hàng chục nghìn tên miền động, giúp mã độc tự đổi domain khi domain cũ bị chặn.
    - f*ast flux:* gán hàng nghìn ip (từ các máy zombie/iot) cho một tên miền và 
    - thay đổi ip liên tục sau mỗi vài phút để tránh bị chặn ip.
![[Pasted image 20260806193240.png|546]]
![[Pasted image 20260806194332.png|607]]![[Pasted image 20260806194346.png|551]]
2- biện pháp phòng thủ
- giám sát lưu lượng mạng: dùng firewall, ids/ips để phát hiện các kết nối bất thường hoặc kết nối tới ip/domain độc hại.
- phân tích truy vấn dns: phát hiện dns tunneling bằng cách theo dõi các request dns có độ dài bất thường hoặc tần suất truy cập cao tới domain lạ.
- giải mã mã hóa (encryption inspection): bóc tách lưu lượng https để kiểm tra nội dung dữ liệu c2 ẩn bên trong.
- triển khai honeypot: dựng hệ thống mồi bẫy để dụ, ghi lại và phân tích hành vi kết nối c2.

3- khác gì so với SSRF
![[Pasted image 20260806193457.png|643]]
[[blind ssrf]]![[Pasted image 20260806193512.png]]
## 8-actions on objectives
tóm tắt giai đoạn actions on objectives (thực hiện mục tiêu):
1- hình thức & mục đích tấn công
- tác động phá hoại: xóa hoặc làm sai lệch dữ liệu để gây gián đoạn hoạt động của hệ thống.
- mục tiêu tài chính: mã hóa dữ liệu đòi tiền chuộc (ransomware), chuyển tiền trái phép hoặc thực hiện các giao dịch gian lận.
- gián điệp & trộm dữ liệu: đánh cắp thông tin nhạy cảm (data exfiltration) để phục vụ gián điệp công nghiệp hoặc chính trị.
- di chuyển ngang (lateral movement): lợi dụng máy đã chiếm quyền để tiếp tục xâm nhập sâu hơn vào các máy khác trong mạng nội bộ.
- thao túng hệ thống công nghiệp: can thiệp vào các hệ thống ics (industrial control systems) hoặc nằm vùng lâu dài chờ thời điểm hành động.


2- biện pháp phòng thủ
- chống thất thoát dữ liệu (dlp): triển khai giải pháp dlp để ngăn chặn hành vi tuồn dữ liệu nhạy cảm ra ngoài.
- sao lưu & phục hồi: duy trì kế hoạch backup dữ liệu định kỳ để ứng phó với ransomware và các đợt tấn công phá hoại.
- phân vùng mạng & phân quyền: áp dụng phân vùng mạng (network segmentation) để ngăn kẻ tấn công di chuyển ngang; tuân thủ nguyên tắc quyền tối thiểu (least privilege).
- giám sát hành vi & edr: theo dõi hoạt động bất thường của người dùng và dùng edr để phát hiện kịp thời các tiến trình độc hại (mã hóa file, can thiệp dữ liệu nhạy cảm...).