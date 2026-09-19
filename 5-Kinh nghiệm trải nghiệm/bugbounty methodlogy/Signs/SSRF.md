# Tấn Công Giả Mạo Yêu Cầu Phía Máy Chủ (SSRF)

---

## Báo Cáo: Lỗ Hổng SSRF Do Không Kiểm Tra Snippet ID Khi Phân Phối Tập Tin

**BÁO CÁO (REPORT):** [HackerOne #997926](https://hackerone.com/reports/997926)

**TIÊU ĐỀ (TITLE):** SSRF - Không kiểm tra ID của Snippet đối với các tập tin phân tán (SSRF - Unchecked Snippet IDs for distributed files)

**CHƯƠNG TRÌNH (PROGRAM):** Open-Xchange

**ỨNG DỤNG MỤC TIÊU (TARGET APP):** [open-xchange/appsuite-middleware](https://github.com/open-xchange/appsuite-middleware)

**CHỨC NĂNG ỨNG DỤNG (APP FUNCTION):**  
Phần mềm nhắn tin, cộng tác và năng suất văn phòng:
- **Dovecot:** Máy chủ gửi nhận email IMAP, POP3 và Submission.
- **App Suite:** Giao diện web truy cập email, lịch làm việc, lưu trữ đám mây và chỉnh sửa tài liệu.
- **PowerDNS:** Máy chủ DNS cung cấp khả năng phân giải tên miền và các tính năng bảo mật mạng.

**ENDPOINT:** `/ajax/snippet?action=new`

**THAO TÁC CRUD / CHỨC NĂNG BỊ KHAI THÁC (CRUD / EXPLOITED FUNCTION):** Create / Dịch vụ thư điện tử (Mail Service)

**TÓM TẮT LỖ HỔNG (SUMMARY):**  
Bằng cách sử dụng phương pháp phân tích mã nguồn tĩnh (static code analysis), chuyên gia bảo mật đã tìm ra cách làm cho ứng dụng bị đầu độc URL, URL độc hại này sau đó được máy chủ sử dụng để thực hiện một request nội bộ.  
Kỹ thuật này đòi hỏi việc sửa một giá trị boolean từ `false` thành `true` trên client để mở khóa thêm chức năng phụ. Bất kỳ người dùng nào cũng có thể thực hiện thao tác này ngay trên máy tính của họ, cho phép gửi đi một URL độc hại để máy chủ thực thi cuộc tấn công SSRF.

**MỨC ĐỘ KỸ NĂNG KỸ THUẬT YÊU CẦU (TECHNICAL SKILL REQUIRED):** Chuyên gia (Expert)

**BÀI HỌC KINH NGHIỆM RÚT RA (TAKEAWAYS):**  
1. Các ràng buộc / giới hạn nghiệp vụ hoàn toàn có thể bị gỡ bỏ bằng cách chỉnh sửa code client nếu ứng dụng có thành phần cài đặt cục bộ (ứng dụng Android, Desktop Client, Electron app).
2. 
3. 

**Ý TƯỞNG TỰ ĐỘNG HÓA PHÁT HIỆN (VULN DISCO AUTOMATION):**  
Không có (None). Mã khai thác này yêu cầu khai thác một Java class rất đặc thù của riêng ứng dụng này, gần như không tái sử dụng được trên các ứng dụng khác.

---

## Mẫu Báo Cáo Phân Tích SSRF

**BÁO CÁO (REPORT):** 

**TIÊU ĐỀ (TITLE):** 

**CHƯƠNG TRÌNH (PROGRAM):** 

**CHỨC NĂNG ỨNG DỤNG (APP FUNCTION):** 

**ENDPOINT:** 

**THAO TÁC CRUD / CHỨC NĂNG BỊ KHAI THÁC (CRUD / EXPLOITED FUNCTION):** 

**TÓM TẮT LỖ HỔNG (SUMMARY):** 

**MỨC ĐỘ KỸ NĂNG KỸ THUẬT YÊU CẦU (TECHNICAL SKILL REQUIRED):** 

**BÀI HỌC KINH NGHIỆM RÚT RA (TAKEAWAYS):**  
1. 
2. 
3. 

**Ý TƯỞNG TỰ ĐỘNG HÓA PHÁT HIỆN (VULN DISCO AUTOMATION):** 

---
