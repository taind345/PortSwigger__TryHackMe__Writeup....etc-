Payload dùng ở **bước khai thác (exploitation)**.

Khi mày chạy một module exploit trong Metasploit, nó thường đi kèm một payload mặc định (hoặc mày chọn bằng `set PAYLOAD`). Lúc mày gõ `exploit` hoặc `run`, exploit sẽ **tấn công lỗ hổng** rồi **gửi payload** vào máy nạn nhân. Payload là thứ chạy trên máy đó sau khi cánh cửa đã được mở.

Ví dụ:

- Exploit `ms17_010_eternalblue` khai thác lỗi SMB.
    
- Payload `windows/x64/meterpreter/reverse_tcp` chạy trên máy Windows, kết nối ngược về máy mày, tạo phiên Meterpreter.
    
Ngoài ra, payload cũng có thể được tạo riêng bằng `msfvenom` (phòng sau) để dùng trong các kịch bản như file đính kèm, web shell, hoặc tiêm qua lỗi upload. Nhưng trong quy trình Metasploit cơ bản, payload luôn nằm trong bước exploit. 😎
![[Pasted image 20260905165243.png]]