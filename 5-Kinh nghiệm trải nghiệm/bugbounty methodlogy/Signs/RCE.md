# Thực Thi Mã Từ Xa (Remote Code Execution - RCE)

---

## Báo Cáo 1: MobileIron Unauthenticated RCE Trên QIWI (Vượt Qua WAF)

**BÁO CÁO (REPORT):** [HackerOne #983548](https://hackerone.com/reports/983548)

**TIÊU ĐỀ (TITLE):** Thực thi mã từ xa không cần xác thực trên mdm.qiwi.com thông qua MobileIron và bypass WAF (MobileIron Unauthenticated RCE on mdm.qiwi.com with WAF bypass)

**CHƯƠNG TRÌNH (PROGRAM):** QIWI

**CHỨC NĂNG ỨNG DỤNG (APP FUNCTION):** Ứng dụng tài chính, hệ thống bán lẻ (PoS) và giao nhận hàng hóa.

**ENDPOINT:** `*.qiwi.com/mifs/.;/services/LogService`

**THAO TÁC CRUD / CHỨC NĂNG BỊ KHAI THÁC (CRUD / EXPLOITED FUNCTION):** 

**TÓM TẮT LỖ HỔNG (SUMMARY):**  
Lỗ hổng này nằm trong phần mềm MobileIron MDM (dùng để quản lý thiết bị cá nhân của nhân viên - BYOD).  
Ba mã CVE liên quan đã được kết hợp và khai thác bằng mã exploit công khai.  
Chuyên gia bảo mật đã viết một lệnh kiểm tra đơn giản dùng `curl` chuyển tiếp sang `grep` để kiểm tra sự tồn tại của endpoint bị lỗi.

**MỨC ĐỘ KỸ NĂNG KỸ THUẬT YÊU CẦU (TECHNICAL SKILL REQUIRED):** Cơ bản (Novice)

**BÀI HỌC KINH NGHIỆM RÚT RA (TAKEAWAYS):**  
1. Hãy nghiên cứu các mã CVE mới ngay khi vừa được công bố, sau đó viết script kiểm tra (check) và bắt đầu rà quét toàn bộ các subdomain của mục tiêu.
2. WAF đã chặn một Java class đã biết (`java.lang.test`). Dấu hiệu này cho thấy hệ thống có WAF và ít nhất một phần ứng dụng được viết bằng Java.
3. Dù CVE/exploit nằm trong phần mềm của một nhà cung cấp bên thứ ba, nhưng miễn là nó chạy trên máy chủ của công ty mục tiêu thì vẫn nằm trong phạm vi (In-Scope) được thưởng. *(Cần kiểm tra kỹ xem sản phẩm đó có phải dạng SaaS do bên thứ 3 host không)*.
4. Quét tìm kiếm cổng `9997`.

**Ý TƯỞNG TỰ ĐỘNG HÓA PHÁT HIỆN (VULN DISCO AUTOMATION):**  
1. Quét template Nuclei.

---

## Báo Cáo 2: RCE Trên GitLab Thông Qua Tùy Chọn Kramdown Không An Toàn

**BÁO CÁO (REPORT):** [HackerOne #1125425](https://hackerone.com/reports/1125425)

**TIÊU ĐỀ (TITLE):** RCE thông qua các tùy chọn Kramdown nội tuyến không an toàn khi hiển thị trang Wiki (RCE via unsafe inline Kramdown options when rendering certain Wiki pages)

**CHƯƠNG TRÌNH (PROGRAM):** GitLab

**CHỨC NĂNG ỨNG DỤNG (APP FUNCTION):** Nền tảng DevOps CI/CD.

**ENDPOINT:** Instance GitLab cá nhân (Personal GitLab instance)

**THAO TÁC CRUD / CHỨC NĂNG BỊ KHAI THÁC (CRUD / EXPLOITED FUNCTION):** Create / Tải lên hình ảnh hoặc tập tin (Image or File Upload)

**TÓM TẮT LỖ HỔNG (SUMMARY):**  
Chuyên gia bảo mật phát hiện khi một trang wiki được tải lên GitLab, có một điều kiện kiểm tra định dạng tập tin theo tên. Nếu tập tin có định dạng khác thường, hàm `other_markup_unsafe` sẽ được gọi để làm sạch văn bản trong file.  
Kramdown và Rouge được sử dụng để làm sạch nội dung. Bằng cách thiết lập tùy chọn `formatter`, chuyên gia đã khởi tạo một đối tượng Redis và gán giá trị của `driver` thành một payload dạng Directory Traversal.  
Giá trị `driver` này sau đó được truyền vào hàm `eval()`, do đó bất kỳ file nào có sẵn trên hệ thống được đặt làm giá trị driver đều sẽ bị thực thi. Cuối cùng, chuyên gia gọi một file đính kèm kèm theo file ban đầu với đường dẫn có thể đoán trước. Bằng cách trỏ driver tới file đính kèm, mã độc trong file đính kèm được thực thi hoàn toàn.

**MỨC ĐỘ KỸ NĂNG KỸ THUẬT YÊU CẦU (TECHNICAL SKILL REQUIRED):** Chuyên gia (Expert)

**BÀI HỌC KINH NGHIỆM RÚT RA (TAKEAWAYS):**  
1. Các đối tượng Redis có biến chuỗi `driver` có thể lợi dụng cho tấn công LFI.
2. Khi mục tiêu là ứng dụng Ruby on Rails, hãy cố gắng tìm phiên bản Kramdown đang dùng.
3. Formatter của Rouge (`syntax_highlighter_opt="{formatter: [OBJECT]}`) có thể bị lạm dụng để khởi tạo đối tượng tùy ý.

**Ý TƯỞNG TỰ ĐỘNG HÓA PHÁT HIỆN (VULN DISCO AUTOMATION):**  
1. Quét nhận diện các ứng dụng Ruby on Rails.
2. Quét các repo GitHub tìm kiếm `Rouge` và `driver`.

---

## Báo Cáo 3: Truy Cập Trái Phép Mail Server Dẫn Đến Chiếm WordPress Và RCE

**BÁO CÁO (REPORT):** [HackerOne #1067547](https://hackerone.com/reports/1067547)

**TIÊU ĐỀ (TITLE):** Truy cập không xác thực vào webmail tại maildev.happytools.dev dẫn đến chiếm quyền trang WordPress api.happytools.dev [RCE]

**CHƯƠNG TRÌNH (PROGRAM):** Automattic (WordPress CMS)

**CHỨC NĂNG ỨNG DỤNG (APP FUNCTION):** WordPress là Hệ quản trị nội dung (CMS) dùng để xây dựng và quản trị website.

**ENDPOINT:** `/wp-login?action=lostpassword` VÀ `https://maildev.happytools.dev/`

**THAO TÁC CRUD / CHỨC NĂNG BỊ KHAI THÁC (CRUD / EXPLOITED FUNCTION):** Update / Đặt lại mật khẩu (Reset Password)

**TÓM TẮT LỖ HỔNG (SUMMARY):**  
Chuyên gia phát hiện bảng đăng nhập WordPress bị lộ ra ngoài và có tính năng reset password.  
Đồng thời, chuyên gia cũng truy cập được vào máy chủ mail thử nghiệm (MailDev) nơi email đặt lại mật khẩu được gửi tới.  
Bằng cách kết hợp 2 endpoint bị lộ này, chuyên gia đã đặt lại mật khẩu tài khoản Admin, sau đó tải lên plugin độc hại để chiếm quyền RCE máy chủ.

**MỨC ĐỘ KỸ NĂNG KỸ THUẬT YÊU CẦU (TECHNICAL SKILL REQUIRED):** Cơ bản (Novice)

**BÀI HỌC KINH NGHIỆM RÚT RA (TAKEAWAYS):**  
1. Khi chứng chỉ SSL hết hạn, hãy thử lùi ngày giờ của request về trước thời điểm hết hạn để xem có tính năng nào khác biệt hoạt động hay không (`tls-scan`).
2. Có sẵn template quét Nuclei trong thư mục `exposed-panels` để tìm các trang login WordPress bị lộ.
3. `api` có thể là một username mặc định trên WordPress.

**Ý TƯỞNG TỰ ĐỘNG HÓA PHÁT HIỆN (VULN DISCO AUTOMATION):**  
- Quét Nuclei tự động.

---
