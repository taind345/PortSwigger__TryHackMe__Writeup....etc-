### 📘 VirusTotal – Công cụ quét mã độc đa nguồn
VirusTotal tổng hợp kết quả từ **hơn 70 trình diệt virus và quét website**. Bạn gửi file, URL, tên miền hoặc hash – nó sẽ cho biết có engine nào gắn cờ độc hại hay không.
Dù không tuyệt đối, đây là nguồn tham khảo phổ biến với **Blue Team** để:
- Xác minh nhanh file/link đáng ngờ.
- Thu thập thông tin về mối đe dọa mới.
### 🧪 Ví dụ
- Nhận email lạ đính kèm `invoice.pdf.exe`. Tải lên VirusTotal.
- Kết quả: 45/70 engine báo **Trojan.Generic**, chi tiết hành vi, domain liên hệ C2.
![[Pasted image 20260729145824.png|510]]
### 📝 Bài tập nhỏ
1. Vào [VirusTotal.com](https://www.virustotal.com), chọn tab **URL**.
2. Nhập `http://example.com/malware` (URL giả định) và quét.
3. Xem chi tiết: có engine nào phát hiện không? Kiểm tra tab **Relations** xem domain liên kết tới IP/file nào.
4. Tìm một hash độc trên mạng (ví dụ: `d41d8cd98f00b204e9800998ecf8427e` là MD5 của file rỗng, nhưng thử tìm hash khác), tra trên VirusTotal xem đã bị phát hiện chưa.
👉 Hoàn thành và cho thầy biết em khám phá được gì mới.