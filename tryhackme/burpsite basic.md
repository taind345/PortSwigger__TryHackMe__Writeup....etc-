# 1- tổng quan
```text
Burp_Suite_Vo_Long
├── 1. Setup_va_Giao_dien (Mở hộp đồ nghề)
│   ├── Features_of_Burp_Community (Biết bản Free nó cho xài cái gì)
│   ├── Installation (Cài đặt)
│   ├── Dashboard & Navigation (Xem bảng điều khiển và cách chuyển tab)
│   └── Options (Chỉnh chọt thông số)
│
├── 2. Trai_tim_cua_Burp_Proxy (Học cách đánh chặn)
│   ├── Intro_to_Burp_Proxy (Bản chất proxy đứng giữa là gì)
│   ├── FoxyProxy (Cài extension trên trình duyệt để trỏ traffic sang Burp)
│   ├── Burp_Suite_Browser (Dùng luôn trình duyệt tích hợp sẵn của Burp cho lẹ)
│   └── Proxying_HTTPS (Cài chứng chỉ CA để bắt được gói tin HTTPS mã hóa)
│
├── 3. Khoanh_vung_Muc_tieu (Trinh sát và Ngắm bắn)
│   ├── Site_Map (Vẽ bản đồ cấu trúc website)
│   ├── Issue_Definitions (Đọc hiểu định nghĩa mấy cái lỗ hổng)
│   └── Scoping_and_Targeting (Khoanh vùng mục tiêu để khỏi bắn nhầm domain người khác)
│
└── 4. Thuc_chien (Quẩy)
    └── Example_Attack (Thử nghiệm đấm phát đầu tiên bằng Burp)

```

**Tóm lại mày sẽ học cái gì?**
Phần này dạy mày cách cài đặt Burp Suite, cấu hình trình duyệt để mọi cú click chuột của mày đều phải đi qua Burp (Proxy). Xong rồi nó dạy mày cách nhìn bao quát toàn bộ cấu trúc web (Site Map), khoanh vùng đúng mục tiêu cần đánh (Scope), và cuối cùng là bắt gói tin lại để sửa đổi rồi mới ném lên server (Example Attack). Học chắc phần này thì sau này mới làm trò mèo được!