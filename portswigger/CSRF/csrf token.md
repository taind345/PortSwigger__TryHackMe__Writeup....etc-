# 1-ví dụ thực tế
**Ví dụ thực tế: Chức năng đổi email trên một trang web ngân hàng hoặc mạng xã hội.**
![[Pasted image 20260926215600.png]]
**1. Phía Server (Khi tải trang):**
Server tạo một token ngẫu nhiên (ví dụ: `a1b2c3d4e5f6`) và lưu nó vào session của người dùng. Sau đó, nó <u>nhúng token này vào form HTML trả về cho trình duyệt:</u>
```html
<form action="/change-email" method="POST">
    <input type="hidden" name="csrf_token" value="a1b2c3d4e5f6"> <=='đoạn này'
    <input type="email" name="email" value="user@example.com">
    <button type="submit">Cập nhật</button>
</form>
```

> [!NOTE]
> tức là cái button "cập nhật" bản thân nó đã có cái csrs token
> --> bấm vô là nó gửi request kèm cái csrs token luôn

**2. Phía Client (Khi người dùng bấm Cập nhật):**
Trình duyệt gửi request kèm theo token:
```http
POST /change-email HTTP/1.1
Host: nganhang.com
Cookie: session=xyz123

email=newemail@gmail.com&csrf_token=a1b2c3d4e5f6
==> cái email nó được đính kèm với csrs token
```

**3. Phía Server (Kiểm tra):**
Server so sánh `csrf_token` trong request với token lưu trong session `xyz123`.
- Nếu khớp → Đổi email thành công.
- Nếu sai hoặc thiếu → Trả về lỗi `403 Forbidden`.

**4. Kẻ tấn công (CSRF):**
Kẻ tấn công tạo một trang web độc hại có form:
```html
<form action="https://nganhang.com/change-email" method="POST">
    <input type="hidden" name="email" value="hacker@evil.com">
</form>
<script>document.forms[0].submit();</script>
```
Khi nạn nhân truy cập, trình duyệt tự động gửi request kèm cookie session. **Tuy nhiên, request này thiếu `csrf_token`** (vì kẻ tấn công không biết token). Server kiểm tra thấy thiếu → Từ chối request. Cuộc tấn công CSRF thất bại.

**Trong thực tế lập trình:**
Các framework hiện đại (Django, Laravel, Spring Security, ASP.NET) đều tự động sinh và kiểm tra CSRF token. Lập trình viên thường chỉ cần thêm một dòng như `{% csrf_token %}` (Django) hoặc `@csrf` (Laravel) vào form là đã được bảo vệ.

