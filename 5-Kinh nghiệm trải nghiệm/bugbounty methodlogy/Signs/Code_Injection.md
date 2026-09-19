# Chèn Mã Thực Thi (Code Injection)

**BÁO CÁO (REPORT):** [HackerOne #1051192](https://hackerone.com/reports/1051192)

**TIÊU ĐỀ (TITLE):** Chèn mã thông qua việc sử dụng hàm yaml.load không an toàn (Code Injection via Insecure Yaml.load)

**CHƯƠNG TRÌNH (PROGRAM):** Kubernetes

**CHỨC NĂNG ỨNG DỤNG (APP FUNCTION):** Tập hợp các công cụ tự động hóa kiểm thử trong Kubernetes (hệ thống điều phối container).

**ENDPOINT:** `https://github.com/kubernetes/test-infra`

**THAO TÁC CRUD / CHỨC NĂNG BỊ KHAI THÁC (CRUD / EXPLOITED FUNCTION):** Create and Update / Script kiểm thử (Test Scripts)

**TÓM TẮT LỖ HỔNG (SUMMARY):**  
Khi phân tích mã nguồn repo `test-infra`, chuyên gia bảo mật phát hiện hàm `yaml.load()` được dùng để cập nhật file cấu hình, dẫn đến lỗ hổng chèn mã (code injection).  
Kỹ thuật này hoạt động hiệu quả (trong hầu hết các trường hợp) nếu công cụ đang sử dụng phiên bản `PyYAML < 5.1`.

**MỨC ĐỘ KỸ NĂNG KỸ THUẬT YÊU CẦU (TECHNICAL SKILL REQUIRED):** Trung bình (Moderate)

**BÀI HỌC KINH NGHIỆM RÚT RA (TAKEAWAYS):**  
1. **Phân biệt:** Code Injection có nghĩa là kẻ tấn công có thể chèn thêm mã của riêng mình để ứng dụng thực thi sau đó. Trong khi Command Injection là khi kẻ tấn công khiến máy chủ mục tiêu trực tiếp thực thi các lệnh hệ thống (OS commands) từ xa.
2. Kiểm tra phiên bản thư viện `PyYAML` xem có nằm trong danh sách các phiên bản dính lỗi không (`< 5.1` không sử dụng `SafeLoader`).
3. 

**Ý TƯỞNG TỰ ĐỘNG HÓA PHÁT HIỆN (VULN DISCO AUTOMATION):**  
1. Quét mã nguồn GitHub để tìm kiếm lệnh `yaml.load()`.
2. Quét nhận diện các ứng dụng Python (Django / Flask).
3. Tìm kiếm trên GitHub từ khóa `PyYAML` trong các file có tên chứa `"requirements"` (như `requirements.txt`), sau đó kiểm tra thủ công phiên bản.

---
