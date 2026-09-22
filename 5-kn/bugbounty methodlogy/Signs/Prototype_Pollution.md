# Prototype Pollution (Gây Ô Nhiễm Prototype)

**BÁO CÁO (REPORT):** [HackerOne #968355](https://hackerone.com/reports/968355)

**TIÊU ĐỀ (TITLE):** [i18next] Tấn công Prototype Pollution ([i18next] Prototype pollution attack)

**CHƯƠNG TRÌNH (PROGRAM):** Node.js third-party modules

**CHỨC NĂNG ỨNG DỤNG (APP FUNCTION):** Mã JavaScript chạy phía máy chủ (Server-side JavaScript)

**ENDPOINT:** Thư viện `i18next`

**THAO TÁC CRUD / CHỨC NĂNG BỊ KHAI THÁC (CRUD / EXPLOITED FUNCTION):** Hàm `addResourceBundle`

**TÓM TẮT LỖ HỔNG (SUMMARY):**  
Chuyên gia bảo mật đã sử dụng kỹ thuật phân tích mã nguồn tĩnh (static code analysis) để phát hiện một module Node.js phổ biến chỉ lọc/làm sạch các thuộc tính `__proto__`, nhưng lại bỏ quên thuộc tính `constructor`.

**MỨC ĐỘ KỸ NĂNG KỸ THUẬT YÊU CẦU (TECHNICAL SKILL REQUIRED):** Trung bình (Moderate)

**BÀI HỌC KINH NGHIỆM RÚT RA (TAKEAWAYS):**  
1. Các cuộc tấn công Prototype Pollution (PPA) có thể nhắm vào cả `__proto__` lẫn `constructor.prototype`.
2. Quét các file JavaScript tìm kiếm `__proto__` để xem quy tắc kiểm tra/lọc dữ liệu của lập trình viên (ví dụ: `if (prop !== '__proto__')` -> có thể bypass bằng `constructor`).
3. Luôn kiểm tra các thư viện JavaScript lỗi thời / phiên bản cũ.

**Ý TƯỞNG TỰ ĐỘNG HÓA PHÁT HIỆN (VULN DISCO AUTOMATION):** 

---
