### 📘 Shodan – Công cụ tìm kiếm thiết bị kết nối Internet
Shodan không chỉ *tìm web* mà *quét mọi thiết bị công khai:* *camera*, *server*, *hệ thống công nghiệp*... Nó cho biết **cái gì đang chạy, ở đâu, phiên bản nào**.
### 🧪 Ví dụ
Tìm server Apache phiên bản cũ:
```
apache 2.4.1
```
Kết quả liệt kê IP, quốc gia, cổng. Kết hợp CVE đã biết → đánh giá rủi ro nhanh.
### 🔎 Bộ lọc hữu ích
| Lọc | Ý nghĩa | Ví dụ |
|-----|--------|-------|
| `country:` | Giới hạn quốc gia | `country:VN` |
| `port:` | Lọc theo cổng | `port:22` (SSH) |
| `org:` | Theo tổ chức/ASN | `org:"Google"` hoặc `AS15169` |
| `hostname:` | Tìm theo tên miền | `hostname:example.com` |
### 📝 Bài tập thực hành
Dùng Shodan (web hoặc CLI), thực hiện:
1. Tìm tất cả thiết bị **MongoDB** ở Việt Nam không có xác thực (cổng 27017).
2. Lọc xem có bao nhiêu IP, ghi lại một địa chỉ.
3. Tra CVE liên quan đến phiên bản MongoDB tìm được.


