- Chuẩn bị Listener: Haỹ cấu hình module exploit/multi/handler để mở tiến trình lắng nghe, sẵn sàng tiếp nhận kết nối trả về tương ứng với payload reverse_https.
    
- Đóng gói Payload: Tạo tệp thực thi shell.exe cấu hình dạng reverse_https nhằm mã hóa toàn bộ lưu lượng điều khiển qua giao thức TLS/HTTPS, hỗ trợ vượt qua các cơ chế giám sát mạng (IDS/IPS) cơ bản.
    
- Phân phối tệp (Delivery): Tận dụng dịch vụ chia sẻ tệp SMB, attacker sử dụng module admin/smb/upload_file để đưa trực tiếp payload.exe lên ổ đĩa của lab
    
- Kích hoạt mã độc (Execution): Tệp payload.exe được thực thi trên môi trường hệ điều hành của lab
    
- Thiết lập phiên điều khiển (C2 Established): Sau khi chạy, payload trên Target chủ động mở kết nối ngược (reverse connection) về listener của attacker qua HTTPS, hoàn tất quá trình bắt tay và khởi tạo phiên Meterpreter toàn quyền trên bài lab 