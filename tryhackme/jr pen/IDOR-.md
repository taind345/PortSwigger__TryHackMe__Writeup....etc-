![[Pasted image 20260731151320.png]]
###  Lý thuyết: IDOR
- **IDOR** (Insecure Direct Object Reference) xảy ra khi ứng dụng dùng tham số dự đoán được (vd: `id=6`) để truy cập đối tượng, nhưng **không kiểm tra quyền** của người dùng hiện tại.
- Ví von: Số phòng khách sạn là chìa khoá. Biết số là vào được phòng khác.
### 🧪 Ví dụ thực tế (từ RecruitX)
- **Web**: `/profile.php?id=6` hiển thị profile của bạn. Thay `id=1` → thấy thông tin **Sarah Mitchell** (admin) dù đang login user thường.
- **API**: `/api/user?id=1` trả JSON có email, vai trò, **không cần session cookie** – còn nguy hiểm hơn.
```bash
curl -s "http://MACHINE_IP/api/user?id=1"
# {"id":1,"name":"Sarah Mitchell","email":"s.mitchell@recruitx.thm","role":"administrator"}
```
### 📝 Bài tập
1. Dùng tài khoản `testuser` đã đăng nhập, đổi `id` từ 1 đến 5, ghi lại email và vai trò của từng người.
2. Kiểm tra API với `curl` không kèm cookie, xem có lấy được dữ liệu không.
3. Trả lời: Làm thế nào để vá lỗi này đơn giản nhất? (gợi ý: kiểm tra session hiện tại có `id` trùng với tham số không, hoặc cấm truy cập nếu không phải admin)
👉 Làm xong báo thầy kết quả.