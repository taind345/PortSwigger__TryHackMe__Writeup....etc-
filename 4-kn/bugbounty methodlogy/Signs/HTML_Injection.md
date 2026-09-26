# Chèn Mã HTML (HTML Injection)

**BÁO CÁO (REPORT):** [HackerOne #381553](https://hackerone.com/reports/381553)

**TIÊU ĐỀ (TITLE):** Chèn mã HTML dẫn tới khả năng thực thi XSS (HTML Injection with XSS possible)

**CHƯƠNG TRÌNH (PROGRAM):** Imgur

**CHỨC NĂNG ỨNG DỤNG (APP FUNCTION):** Tải lên và xem hình ảnh (Upload and view pictures)

**ENDPOINT:** `https://[USERNAME].imgur.com`

**THAO TÁC CRUD / CHỨC NĂNG BỊ KHAI THÁC (CRUD / EXPLOITED FUNCTION):** Create and Update / Tạo đối tượng Album ảnh (Create Object - Album)

**TÓM TẮT LỖ HỔNG (SUMMARY):**  
Chuyên gia bảo mật đã chèn mã HTML thành công và nâng cấp thành payload XSS bằng cách đặt payload làm tên của Album được tạo mới.  
Payload tải lên là: `#"></div><a href=javaSCRIPT&colon;alert(/XSS/)>XSS</a>`, thực hiện thành công nhờ kỹ thuật vượt qua WAF.

**MỨC ĐỘ KỸ NĂNG KỸ THUẬT YÊU CẦU (TECHNICAL SKILL REQUIRED):** Cơ bản (Novice)

**BÀI HỌC KINH NGHIỆM RÚT RA (TAKEAWAYS):**  
1. Luôn luôn kiểm tra lỗi HTML Injection trước, sau đó mới tìm cách nâng cấp lên XSS.
2. Thực thể mã hóa `&colon;` có thể được sử dụng để thay thế cho dấu hai chấm `:` nhằm bypass các bộ lọc/WAF.
3. 

**Ý TƯỞNG TỰ ĐỘNG HÓA PHÁT HIỆN (VULN DISCO AUTOMATION):**  
1. Fuzzing tự động phát hiện HTML Injection.

---
