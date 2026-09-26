### Bắt Tay 3 Bước (TCP Three-Way Handshake)

Quá trình này đảm bảo hai bên (Client và Server) đồng bộ trạng thái trước khi thực sự trao đổi dữ liệu.

#### Cơ chế hoạt động ngắn gọn

1. **SYN (Client $\rightarrow$ Server):** "Tôi muốn kết nối. Số thứ tự (Sequence) của tôi là X."
    
2. **SYN-ACK (Server $\rightarrow$ Client):** "Xác nhận (ACK = X+1). Tôi cũng muốn kết nối. Số thứ tự của tôi là Y."
    
3. **ACK (Client $\rightarrow$ Server):** "Xác nhận (ACK = Y+1). Mọi thứ ok, bắt đầu gửi dữ liệu."
    

### Insights cho Phỏng vấn Web Pentest & Security

Trong các buổi phỏng vấn bảo mật (Pentest hoặc SOC), nhà tuyển dụng thường không chỉ hỏi định nghĩa mà sẽ khai thác cách bạn ứng dụng kiến thức này vào thực tế. Dưới đây là các điểm "ăn tiền" bạn cần nắm vững:

#### 1. Kỹ thuật Port Scanning (Nmap)

Khi thực hiện trinh sát (Reconnaissance) mục tiêu web, cách công cụ quét cổng tương tác với TCP Handshake rất quan trọng:

- **TCP Connect Scan (`-sT`):** Hoàn thành đầy đủ 3 bước (SYN $\rightarrow$ SYN-ACK $\rightarrow$ ACK).
    
    - _Nhược điểm:_ Chắc chắn bị lưu log ở tầng ứng dụng (Web server log, Application log) vì hệ điều hành đã thiết lập kết nối thành công.
        
- **TCP SYN Scan / Stealth Scan (`-sS`):** Chỉ gửi `SYN`, nhận `SYN-ACK` từ server, nhưng sau đó Client lập tức gửi `RST` (Reset) thay vì `ACK` để ngắt kết nối.
    
    - _Ưu điểm:_ Vượt qua việc bị ghi log ở tầng ứng dụng vì kết nối chưa bao giờ hoàn thành (half-open), giúp pentester ẩn mình tốt hơn trước hệ thống giám sát.
        

#### 2. Tấn công Từ chối dịch vụ (SYN Flood)

- **Cơ chế:** Kẻ tấn công gửi hàng loạt gói `SYN` đến Web Server nhưng **không bao giờ gửi gói `ACK` cuối cùng** (hoặc dùng IP giả mạo).
    
- **Hậu quả:** Server phải cấp phát bộ nhớ để duy trì các "kết nối đang mở một nửa" (half-open connections) chờ gói `ACK`. Khi hàng đợi này đầy, server treo và từ chối người dùng hợp lệ.
    
- **Giải pháp (Dành cho góc nhìn phòng thủ):** Bật tính năng _SYN Cookies_ trên server hoặc cấu hình Firewall/WAF giới hạn rate limit.
    

#### 3. Vượt mặt Firewall (Firewall Evasion)

- **Stateful Firewall:** Các tường lửa hiện đại theo dõi trạng thái kết nối. Nếu một gói tin mang cờ `ACK` bay tới mà không có quá trình `SYN` báo trước, nó sẽ bị drop.
    
- **Ứng dụng:** Pentester có thể gửi các gói tin TCP với các cờ (flags) bất thường (như NULL, FIN, XMAS scan) để xem cách Firewall phản hồi, từ đó vẽ bản đồ các rule đang được cấu hình chặn/mở cổng nào phía sau WAF.
    

Bạn có muốn đi sâu vào cách đọc và phân tích các luồng gói tin TCP này (PCAP) thông qua Wireshark để chuẩn bị cho các câu hỏi tình huống thực hành không?