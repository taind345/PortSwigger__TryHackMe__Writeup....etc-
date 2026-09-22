# Tấn Công Cross-Site Scripting (XSS)

---

## Báo Cáo 1: Attribute Injection Qua Các Thẻ HTML Lồng Nhau (XSS Trong Ghi Chú)

**BÁO CÁO (REPORT):** [HackerOne #995273](https://hackerone.com/reports/995273)

**TIÊU ĐỀ (TITLE):** XSS - Ghi chú - Chèn thuộc tính thông qua các thẻ chồng chéo (XSS - Notes - Attribute injection through overlapping tags)

**CHƯƠNG TRÌNH (PROGRAM):** Open-Xchange

**CHỨC NĂNG ỨNG DỤNG (APP FUNCTION):** Ứng dụng ghi chú cá nhân (Notes), một phần trong bộ phần mềm cộng tác doanh nghiệp.

**ENDPOINT:** `https://github.com/open-xchange/appsuite-middleware` (Module Notes)

**THAO TÁC CRUD / CHỨC NĂNG BỊ KHAI THÁC (CRUD / EXPLOITED FUNCTION):** Create / Tải lên hình ảnh hoặc tập tin (văn bản được chuyển đổi sang file text thuần)

**TÓM TẮT LỖ HỔNG (SUMMARY):**  
Chuyên gia nghiên cứu nhận thấy ứng dụng ghi chú thực hiện nhiều vòng lọc/thay thế dữ liệu đầu vào bằng phương thức `.replace()` trong JavaScript.  
Bằng cách thêm một URL đầy đủ vào trong thẻ `<img src>`, điều này sẽ kích hoạt lần gọi `.replace()` thứ hai, tự động chèn thêm một thẻ `<a href>` vào bên trong thẻ `<img src>`.  
Việc chèn thêm dấu ngoặc kép sẽ bẻ gãy cấu trúc thẻ HTML (break markup tag), cho phép chèn thêm thuộc tính mới (`onerror` phục vụ thực thi mã JavaScript độc hại).  
Thông thường điều này sẽ làm hỏng thẻ ảnh, nhưng với payload được tinh chỉnh khéo léo, chuyên gia đã vá lỗi cú pháp thành công, dẫn đến mã JavaScript được thực thi trơn tru.

**MỨC ĐỘ KỸ NĂNG KỸ THUẬT YÊU CẦU (TECHNICAL SKILL REQUIRED):** Chuyên gia (Expert)

**BÀI HỌC KINH NGHIỆM RÚT RA (TAKEAWAYS):**  
1. Tìm kiếm các phương thức `.escape()` và `.replace()` trong mã nguồn JavaScript.
2. Tìm kiếm cách ứng dụng phân tích cú pháp ngôn ngữ đánh dấu (markup language) trong mã nguồn.
3. `Underscore` là một thư viện JavaScript rất phổ biến dùng để escape các ký tự đặc biệt (`&` thành `&amp;`).
4. Tìm kiếm các phiên bản lỗi thời của `underscore.js` (dính CVE: `>=1.13.0-0 <1.13.0-2`, `>=1.3.2 <1.12.1`).

**Ý TƯỞNG TỰ ĐỘNG HÓA PHÁT HIỆN (VULN DISCO AUTOMATION):**  
1. Dùng Wappalyzer để nhận diện số phiên bản `underscore.js`.

---

## Báo Cáo 2: Stored XSS Khi Tìm Kiếm Danh Bạ Do Không Escape Chức Vụ

**BÁO CÁO (REPORT):** [HackerOne #993222](https://hackerone.com/reports/993222)

**TIÊU ĐỀ (TITLE):** XSS - Tìm kiếm - Trường chức vụ/nghề nghiệp trong liên hệ không được escape (XSS - Search - Unescaped contact job)

**CHƯƠNG TRÌNH (PROGRAM):** Open-Xchange

**CHỨC NĂNG ỨNG DỤNG (APP FUNCTION):** Đóng vai trò danh bạ liên hệ (Address Book). Có thể thêm danh bạ mới, v.v.

**ENDPOINT:** `https://github.com/open-xchange/appsuite-middleware`

**THAO TÁC CRUD / CHỨC NĂNG BỊ KHAI THÁC (CRUD / EXPLOITED FUNCTION):** Create / Tạo đối tượng liên hệ (Create Object)

**TÓM TẮT LỖ HỔNG (SUMMARY):**  
Chuyên gia phát hiện nội dung nhập vào không hề được lọc/escape khi thêm mới trường Công ty (Company) hoặc Chức vụ (Position).  
Điều này khiến cho bất kỳ người dùng nào tìm kiếm thông tin về kẻ tấn công trong danh bạ đều sẽ bị thực thi mã JavaScript độc hại ngay trên trình duyệt của họ (Stored XSS).

**MỨC ĐỘ KỸ NĂNG KỸ THUẬT YÊU CẦU (TECHNICAL SKILL REQUIRED):** Cơ bản (Novice)

**BÀI HỌC KINH NGHIỆM RÚT RA (TAKEAWAYS):**  
1. Các chức năng cơ bản như lọc dữ liệu/escape mã đầu vào rất dễ bị lập trình viên bỏ quên ở các trường dữ liệu phụ (Job, Title, Bio, Notes).
2. Payload XSS không phải lúc nào cũng được thực thi ngay tại trang được tải sau khi submit (có thể thực thi ở trang thứ ba như trang Admin, trang Search, trang Export).
3. Luôn sử dụng `xsshunter` (hoặc dựng server lắng nghe OOB riêng) cho các payload để bắt các lỗi Blind XSS / Stored XSS.

**Ý TƯỞNG TỰ ĐỘNG HÓA PHÁT HIỆN (VULN DISCO AUTOMATION):** 

---
