### So sánh HTTP và HTTPS

Sự khác biệt cốt lõi nằm ở **chữ "S" (Secure)**. Trọng tâm của sự khác biệt này là việc mã hóa dữ liệu trong quá trình truyền tải.

#### Bảng So Sánh Nhanh

| Đặc điểm             | HTTP (HyperText Transfer Protocol)           | HTTPS (HTTP Secure)                             |
| -------------------- | -------------------------------------------- | ----------------------------------------------- |
| **Bảo mật**          | Không (Dữ liệu truyền dưới dạng _Plaintext_) | Có (Dữ liệu được mã hóa thành _Ciphertext_)     |
| **Cổng (Port)**      | 80                                           | 443                                             |
| **Giao thức hỗ trợ** | TCP                                          | TCP + **SSL/TLS**                               |
| **Tốc độ**           | Nhanh hơn (không tốn tài nguyên mã hóa)      | Chậm hơn một chút (do quá trình mã hóa/giải mã) |
| **Chứng chỉ**        | Không yêu cầu                                | Bắt buộc phải có chứng chỉ SSL/TLS              |

#### Ví dụ Thực tế (Real-world Analogy)

- **HTTP giống như gửi một tấm bưu thiếp (Postcard):** Bạn viết nội dung lên đó và gửi đi. Bác đưa thư, người phân loại ở bưu điện, hay bất kỳ ai cầm tấm bưu thiếp đó đều có thể đọc được toàn bộ nội dung bạn viết.
    
- **HTTPS giống như gửi một hộp sắt được khóa bằng mật mã:** Bạn bỏ bức thư vào hộp, khóa lại và gửi đi. Những người trung gian chỉ thấy cái hộp sắt. Chỉ có người nhận (với chiếc chìa khóa tương ứng) mới có thể mở hộp và đọc được thư.
    

### Insights cho Phỏng vấn Web Pentest & SOC

Nhà tuyển dụng sẽ muốn biết bạn hiểu HTTPS tác động thế nào đến việc phòng thủ và tấn công.

#### 1. Góc nhìn Tấn công (Pentest / Red Team)

- **Sniffing & Man-in-the-Middle (MitM):** * Với **HTTP**, nếu bạn cùng mạng LAN với nạn nhân (dùng _ARP Spoofing_), bạn có thể dùng Wireshark bắt trọn gói tin, đọc được ngay Username, Password, và Session Cookies dưới dạng text rõ ràng.
    
    - Với **HTTPS**, bạn bắt được gói tin nhưng chỉ thấy dữ liệu rác đã mã hóa.
        
- **Kỹ thuật Bypassing HTTPS:** Để tấn công HTTPS, Pentester thường dùng kỹ thuật **SSL Stripping** (ép kết nối hạ cấp từ HTTPS xuống HTTP) hoặc tấn công lừa đảo để nạn nhân cài đặt một **Fake Root CA Certificate** của hacker vào máy (thường dùng công cụ như _Burp Suite_ proxy).
    

#### 2. Góc nhìn Phòng thủ & Phân tích (SOC / Blue Team)

- **Mù rọi (Blind Spot) trên hệ thống giám sát:** Các thiết bị IDS/IPS (như Snort, Suricata) không thể đọc được nội dung độc hại (payload) nếu hacker bọc nó trong kết nối HTTPS.
    
- **Giải pháp cho SOC:** Để quét được mã độc trong luồng HTTPS, doanh nghiệp phải triển khai kiến trúc **SSL/TLS Inspection (SSL Decryption)** trên Tường lửa (Firewall) hoặc Web Proxy. Thiết bị này sẽ đứng giữa, giải mã, kiểm tra, sau đó mã hóa lại trước khi gửi đi.
    

#### 3. Lỗ hổng cấu hình (Misconfiguration)

Việc có chữ "HTTPS" không đảm bảo website an toàn 100%. Trong các kỳ pentest, bạn cần kiểm tra xem server có:

- Hỗ trợ các giao thức mã hóa lỗi thời, yếu kém không (như SSLv2, SSLv3, TLS 1.0, TLS 1.1).
    
- Có bị dính các lỗ hổng SSL/TLS kinh điển như **Heartbleed**, **POODLE**, **BEAST** hay không.
    
- Thiếu header **HSTS (HTTP Strict Transport Security)**: HSTS ép trình duyệt luôn dùng HTTPS, ngăn chặn triệt để kỹ thuật hạ cấp SSL Stripping.