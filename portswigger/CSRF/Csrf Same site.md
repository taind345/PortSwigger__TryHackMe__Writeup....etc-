
# 1- Lý thuyết về samesite
### 1. Phân biệt "Site" và "Origin"

**Origin** = Scheme + Domain + Port (cực kỳ nghiêm ngặt).
**Site** = TLD + 1 (ít nghiêm ngặt hơn).

**Ví dụ:**
- `https://app.example.com` và `https://intranet.example.com`:
  - **Cùng Site?** Có (vì cùng `example.com`).
  - **Cùng Origin?** Không (vì khác subdomain).
- `http://example.com` và `https://example.com`:
  - **Cùng Site?** Không (khác scheme).
  - **Cùng Origin?** Không.

 **Tại sao điều này quan trọng?** SameSite bảo vệ dựa trên **Site**, không phải **Origin**. Nghĩa là nếu kẻ tấn công có XSS trên `app.example.com`, hắn có thể tấn công `intranet.example.com` vì trình duyệt coi chúng là **same-site**.


### 2. Ba mức độ SameSite (Strict, Lax, None)

Giả sử bạn đang đăng nhập vào `facebuk.com` (có cookie `session`). Kẻ tấn công dụ bạn vào `evil.com`.

####  SameSite=Strict (Nghiêm ngặt nhất)
- **Kịch bản:** `evil.com` có link `<a href="https://facebuk.com/settings">Click here</a>`.
- **Kết quả:** Khi bạn click, trình duyệt **KHÔNG gửi cookie** `session` của `facebuk.com`. Bạn bị đăng xuất hoặc yêu cầu đăng nhập lại.
- **Ứng dụng:** Dùng cho các hành động nhạy cảm như đổi mật khẩu, chuyển tiền.

####  SameSite=Lax (Mặc định của Chrome)
- **Kịch bản 1 (GET - Top-level navigation):** `evil.com` có link `<a href="https://facebuk.com/change-email?email=hacker@evil.com">Click</a>`.
  - **Kết quả:** Cookie `session` **ĐƯỢC GỬI**. Vì đây là GET và là top-level navigation (click link).
  - → **Lỗ hổng:** Nếu server cho phép đổi email bằng GET, CSRF thành công.
- **Kịch bản 2 (POST):** `evil.com` có form ẩn submit POST đến `facebuk.com/change-email`.
  - **Kết quả:** Cookie `session` **KHÔNG ĐƯỢC GỬI**. Lax chặn cross-site POST.
- **Kịch bản 3 (Background request):** `evil.com` có `<img src="facebuk.com/change-email?email=hacker@evil.com">`.
  - **Kết quả:** Cookie `session` **KHÔNG ĐƯỢC GỬI**. Lax chặn request ngầm (iframe, ảnh, script).

####  SameSite=None (Tắt bảo vệ)
- **Kịch bản:** `facebuk.com` nhúng iframe từ `tracker.com`. `tracker.com` cần cookie để theo dõi bạn.
- **Kết quả:** Cookie **LUÔN ĐƯỢC GỬI** trong mọi ngữ cảnh, kể cả từ `evil.com`.
- **Lưu ý:** Phải đi kèm `Secure` (chỉ gửi qua HTTPS).
- **Rủi ro:** Nếu cookie session được set `SameSite=None`, CSRF có thể xảy ra từ bất kỳ đâu.


> [!NOTE] note
> - [ ] samesite: cờ này trong request giúp chỉ chấp nhận request từ chính cái site đó ==> nó sử dụng cơ chế gắn cookie
> 	- [ ] samesite=strict -> nghiêm ngặt, chỉ chấp nhận request từ chính cái site đó. Ko đính kèm cookie vào những request ko cùng site
> 	- [ ] samesite=lax  -> Lax nó sẽ cho phép GET , nhưng ko cho phép POST từ những website khác site
> 	- [ ] samesite = none -> cho phép tất cả 

### 3. Bypass SameSite=Lax bằng GET

Vì Lax cho phép cookie trong **GET top-level navigation**, kẻ tấn công có thể lợi dụng nếu server chấp nhận GET để thay đổi trạng thái.

**Ví dụ 1: Dùng JavaScript chuyển hướng**
```html
<script>
    document.location = 'https://facebuk.com/change-email?email=hacker@evil.com';
</script>
```
→ Trình duyệt coi đây là top-level navigation (GET) → Gửi cookie `session` → CSRF thành công.

**Ví dụ 2: Dùng Method Override (Framework như Symfony)**
Một số framework cho phép ghi đè method bằng tham số `_method`:
```html
<form action="https://facebuk.com/change-email" method="GET">
    <input type="hidden" name="_method" value="POST">
    <input type="hidden" name="email" value="hacker@evil.com">
</form>
<script>document.forms[0].submit();</script>
```
→ Trình duyệt gửi request **GET** (nên cookie được gửi), nhưng server đọc `_method=POST` và thực hiện hành động như POST.
→ **Bypass thành công SameSite=Lax.**


###  Tóm tắt thực chiến:
| Kịch bản                         | SameSite=Strict    | SameSite=Lax       | SameSite=None |
| -------------------------------- | ------------------ | ------------------ | ------------- |
| Click link (GET) từ site khác    | ❌ Không gửi cookie | ✅ Gửi cookie       | ✅ Gửi cookie  |
| Form POST từ site khác           | ❌ Không gửi cookie | ❌ Không gửi cookie | ✅ Gửi cookie  |
| `<img>`, `<iframe>` từ site khác | ❌ Không gửi cookie | ❌ Không gửi cookie | ✅ Gửi cookie  |

**Khi test CSRF:**
1. Kiểm tra cookie `session` có `SameSite` gì.
2. Nếu là `Lax` → Thử đổi POST sang GET, dùng `document.location` hoặc `_method`.
3. Nếu là `Strict` → Cần tìm lỗ hổng XSS hoặc subdomain cùng site.
4. Nếu là `None` → CSRF dễ dàng như không có bảo vệ.


# 2- Nhìn vào đâu để phát hiện 1 trang  web có samesite và same origin
Để kiểm tra một trang web có **SameSite** và **Same Origin** hay không,cần nhìn vào **2 nơi khác nhau**: Server (Response) và Trình duyệt (DevTools). Dưới đây là hướng dẫn chi tiết.
### 1. Cách phát hiện SameSite (Cookie)

SameSite là thuộc tính của cookie, do **server** quy định trong header `Set-Cookie`. Bạn có 2 cách để kiểm tra:

**Cách 1: Dùng Burp Suite (Chính xác nhất)**
1. Bắt request đăng nhập hoặc bất kỳ request nào server set cookie.
2. Nhìn vào **Response Header**, tìm dòng `Set-Cookie`.
3. Đọc phần đuôi của cookie:
   *   `Set-Cookie: session=abc; SameSite=Strict` → Strict
   *   `Set-Cookie: session=abc; SameSite=Lax` → Lax
   *   `Set-Cookie: session=abc; SameSite=None; Secure` → None
   *   `Set-Cookie: session=abc;` (không ghi gì) → **Mặc định là Lax trên Chrome**.

**Cách 2: Dùng Chrome DevTools**
1. Nhấn **F12** → tab **Application** → **Cookies**.
2. Tìm cột **SameSite**.
   *   Nếu ghi `Lax`, `Strict`, `None` → Đó là giá trị thực tế.
   *   Nếu **để trống** (như ảnh bạn gửi) → Chrome đang áp dụng mặc định `Lax`.

### 2. Cách phát hiện Same Origin

Same Origin (cùng nguồn gốc) yêu cầu **3 yếu tố giống hệt nhau**: Scheme (http/https) + Domain + Port.

**Cách 1: Nhìn vào thanh địa chỉ (URL Bar)**
So sánh URL của trang hiện tại với URL đích:
*   `https://app.example.com` và `https://intranet.example.com` → **Khác Origin** (khác subdomain).
*   `http://example.com` và `https://example.com` → **Khác Origin** (khác scheme).
*   `https://example.com` và `https://example.com:8080` → **Khác Origin** (khác port).

**Cách 2: Dùng Burp Suite**
Nhìn vào 2 header trong request:
*   `Host: 0acf...web-security-academy.net`
*   `Origin: https://attacker.com`
*   Nếu `Origin` khác `Host` → **Cross-Origin** (khác nguồn gốc).
*   Nếu `Origin` giống hệt `Host` → **Same-Origin**.

**Cách 3: Dùng Chrome DevTools (Console)**
Mở Console (F12 → tab Console), gõ lệnh:
```javascript
window.location.origin
```
Nó sẽ trả về origin chính xác của trang hiện tại (ví dụ: `https://0acf...web-security-academy.net`). Nếu bạn thử `fetch()` đến một URL khác origin, trình duyệt sẽ báo lỗi CORS (Cross-Origin Resource Sharing).

### 3. Bảng tóm tắt nhanh

| Yếu tố | Nơi kiểm tra | Dấu hiệu |
|---|---|---|
| **SameSite Cookie** | Burp Response / DevTools Application | `SameSite=Lax/Strict/None` hoặc để trống (mặc định Lax) |
| **Same Origin** | URL Bar / Burp Request (Origin vs Host) | Scheme + Domain + Port giống hệt nhau |
| **Cross-Site** | Burp Request (`Sec-Fetch-Site`) | `cross-site`, `same-site`, `same-origin` |

* Mẹo thực chiến:
Khi test CSRF, hãy kiểm tra theo thứ tự:
1. Cookie có `SameSite` gì? (Nếu `None` → CSRF dễ như ăn kẹo).
2. Nếu `Lax` → Thử bypass bằng `GET` + `_method=POST` hoặc `document.location`.
3. Nếu `Strict` → Cần tìm lỗ hổng XSS hoặc subdomain cùng site.
4. Kiểm tra `Origin`/`Referer` server có validate không? (Nếu có, thử bypass bằng cách chèn domain lab vào URL exploit).

# 3 Bypass samesite
## 3.1 bypass bằng redirect
**Tóm tắt ngắn gọn:** SameSite=Strict chặn cookie trong các request từ bên ngoài (cross-site). Nhưng nếu trang web có một "gadget" (ví dụ: chức năng chuyển hướng phía client bằng JavaScript), kẻ tấn công có thể lợi dụng nó để tạo ra một request **same-site** đến chính trang web đó. Trình duyệt coi request này là same-site nên sẽ gửi kèm cookie, bất chấp SameSite=Strict.

> [!NOTE]
> đại khái là 1 trình duyệt có redirect thì nó khả năng sẽ bị lợi dụng để bypass đi samesite
> - [ ]  liệu có thể liên hệ với redirect trong SSRF ko??
> 	- [ ]  [[0-roadmap SSRF]]

**Bối cảnh:**
- Trang web `victim.com` có cookie `session` với `SameSite=Strict`.
- Trang web này có chức năng chuyển hướng phía client (JavaScript) dựa trên tham số URL:
  `/redirect?url=/my-account`

**Kịch bản tấn công:**

1. **Nạn nhân** đang đăng nhập `victim.com` (có cookie `session` Strict).
2. **Kẻ tấn công** dụ nạn nhân truy cập vào trang `attacker.com`.
3. Trang `attacker.com` chèn một iframe ẩn:
   ```html
   <iframe src="https://victim.com/redirect?url=/change-email?email=hacker@evil.com"></iframe>
   ```
4. **Trình duyệt** tải `victim.com/redirect?...` → Đây là **cross-site request** → Cookie `session` **KHÔNG ĐƯỢC GỬI** (do SameSite=Strict).
5. **Nhưng!** JavaScript trên trang `/redirect` chạy và thực hiện:
   ```javascript
   window.location = '/change-email?email=hacker@evil.com';
   ```
6. Trình duyệt thực hiện request thứ hai đến `/change-email`. Vì request này bắt nguồn từ chính JavaScript của `victim.com`, trình duyệt coi đây là **same-site request** → Cookie `session` **ĐƯỢC GỬI KÈM**.
7. Server `victim.com` nhận request hợp lệ → **Đổi email thành công**.

**Tại sao Server-side redirect KHÔNG hoạt động?**

Nếu `/redirect` là **server-side** (trả về HTTP 302), trình duyệt sẽ ghi nhớ rằng request ban đầu là cross-site. Khi nó theo redirect đến `/change-email`, nó vẫn áp dụng hạn chế SameSite=Strict → Cookie vẫn bị chặn.

**Điểm mấu chốt:** Client-side redirect (JavaScript) làm mất dấu vết "cross-site" của request ban đầu, khiến request thứ hai trở thành same-site trong mắt trình duyệt.