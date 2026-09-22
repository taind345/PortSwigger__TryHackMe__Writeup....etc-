# Tiết Lộ Thông Tin Nhạy Cảm (Information Disclosure)

**BÁO CÁO (REPORT):** [HackerOne #1134060](https://hackerone.com/reports/1134060)

**TIÊU ĐỀ (TITLE):** Phát hiện thông tin xác thực trong file cấu hình trên GitHub (Credentials found in config file on github)

**CHƯƠNG TRÌNH (PROGRAM):** BlockFi

**CHỨC NĂNG ỨNG DỤNG (APP FUNCTION):** Nền tảng cho vay tiền mã hóa (Cryptocurrency Lending)

**ENDPOINT:** GitHub Repo

**THAO TÁC CRUD / CHỨC NĂNG BỊ KHAI THÁC (CRUD / EXPLOITED FUNCTION):** Đẩy dữ liệu nhạy cảm lên GitHub (Sensitive Data Pushed to Github)

**TÓM TẮT LỖ HỔNG (SUMMARY):**  
Chuyên gia bảo mật đã tìm thấy thông tin đăng nhập/xác thực (credentials) trong 2 repository GitHub riêng biệt. Các thông tin này thuộc về các người dùng/nhân viên khác nhau.

**MỨC ĐỘ KỸ NĂNG KỸ THUẬT YÊU CẦU (TECHNICAL SKILL REQUIRED):** Cơ bản (Novice)

**BÀI HỌC KINH NGHIỆM RÚT RA (TAKEAWAYS):**  
1. Các file cấu hình định dạng YAML (`.yml`, `.yaml`) rất thường chứa dữ liệu nhạy cảm (API keys, passwords, connection strings).
2. 
3. 

**Ý TƯỞNG TỰ ĐỘNG HÓA PHÁT HIỆN (VULN DISCO AUTOMATION):**  
- `github_brute-dork.py`

---
