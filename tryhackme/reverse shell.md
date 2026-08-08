---

excalidraw-plugin: parsed
tags: [excalidraw]

---
==⚠  Switch to EXCALIDRAW VIEW in the MORE OPTIONS menu of this document. ⚠== You can decompress Drawing data with the command palette: 'Decompress current Excalidraw file'. For more info check in plugin settings under 'Saving'



# attack chain

### 📘 Tổng kết chuỗi tấn công & Biện pháp khắc phục

#### 🧪 Chuỗi tấn công từ đầu đến cuối:
1. **Trinh sát** – Phát hiện Apache, PHP, MySQL; tìm được `/api`, `/admin`, `/reset.php`, `/uploads`.
2. **IDOR** – Dùng `id` trên profile và API lấy được email admin `s.mitchell@recruitx.thm`.
3. **Reset password yếu** – Token 6 chữ số hiện ngay trên màn hình, cho phép đặt lại mật khẩu admin.
4. **Truy cập Admin Panel** – Chiếm tài khoản admin, vào chức năng upload file.
5. **RCE** – Upload shell `.phtml` vượt qua bộ lọc, thực thi lệnh trên server, lấy reverse shell.

➡️ Không lỗ hổng đơn lẻ nào gây ra toàn bộ sự cố; chính sự kết hợp các điểm yếu đã dẫn đến chiếm toàn bộ máy chủ.

#### 🛡️ Khắc phục (tóm tắt)

| Lỗ hổng | Mức độ | Cách sửa |
|--------|--------|----------|
| **IDOR** (profile & API) | Cao | Kiểm tra quyền server-side: người dùng chỉ xem được dữ liệu của chính họ. |
| **Token reset hiển thị trên HTTP** | Nghiêm trọng | Chỉ gửi token qua email; dùng token ngẫu nhiên ít nhất 32 ký tự. |
| **Upload filter không chặt** | Nghiêm trọng | Dùng danh sách cho phép (allowlist), kiểm tra cả MIME type; lưu file ngoài web root. |
| **Lộ cấu trúc API** | Trung bình | Ẩn hoặc giới hạn endpoint index, chỉ cho admin đã xác thực. |

---

### 📝 Bài tập củng cố
1. Sắp xếp các lỗ hổng theo mức độ ưu tiên sửa (từ cao xuống thấp) và giải thích lý do.
2. Nếu em không có quyền truy cập vào email của admin, có cách nào khác để lấy token reset không? (Gợi ý: brute‑force khoảng 1 triệu số)
3. Đề xuất thêm một biện pháp phòng thủ cho chức năng upload file ngoài allowlist và kiểm tra MIME.

👉 Làm xong báo thầy. Chúng ta sẽ chuyển sang chủ đề mới: **Leo quyền Linux cơ bản**.


# Excalidraw Data

## Text Elements
đọc phần này trước ^ae4GAUsR

[[netcat]] ^VE2zGgQG

attack chain
 ^K68GvSJ6

## Element Links
zR6TlSxq: [[tryhackme/reverse shell.md#attack chain]]

%%
## Drawing
```compressed-json
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
```
%%