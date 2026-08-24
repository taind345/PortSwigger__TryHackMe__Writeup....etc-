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

| Lỗ hổng                            | Mức độ       | Cách sửa                                                                             |
| ---------------------------------- | ------------ | ------------------------------------------------------------------------------------ |
| **IDOR** (profile & API)           | Cao          | Kiểm tra quyền server-side: người dùng chỉ xem được dữ liệu của chính họ.            |
| **Token reset hiển thị trên HTTP** | Nghiêm trọng | Chỉ gửi token qua email; dùng token ngẫu nhiên ít nhất 32 ký tự.                     |
| **Upload filter không chặt**       | Nghiêm trọng | Dùng danh sách cho phép (allowlist), kiểm tra cả MIME type; lưu file ngoài web root. |
| **Lộ cấu trúc API**                | Trung bình   | Ẩn hoặc giới hạn endpoint index, chỉ cho admin đã xác thực.                          |

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
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=
```
%%