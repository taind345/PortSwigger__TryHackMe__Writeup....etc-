## Cách phòng chống XSS
PortSwigger đưa ra 4 lớp chính:
```text
Phòng chống XSS
│
├── 1. Filter input
├── 2. Encode output
├── 3. Response headers
└── 4. CSP
```

### 1. Filter input khi nhận dữ liệu
-Ví dụ trường tuổi:
```text
age = 21
```
   - Ứng dụng chỉ nên nhận:

```text
0–120
```

   - Không cần cho phép:
```html
<script>alert(1)</script>
```

-Đây gọi là **allowlist / whitelist**:
```text
Cho phép những gì hợp lệ
thay vì
cố gắng chặn mọi thứ nguy hiểm
```

### 2. Encode output
-Đây là biện pháp rất quan trọng.
   - Input:
```html
<script>alert(1)</script>
```
   - Nếu output được encode:*tức là response trả về browser cái input dưới dạng mã hóa*
```html
&lt;script&gt;alert(1)&lt;/script&gt;
```
   - Trình duyệt sẽ hiển thị nó như **text**, không thực thi.
```text
browser Input -> server
			       |	
encoded input<-------|
--> browser ko thực thi được script
```
>tức là mã hóa đầu vào

-Nhưng encoding phải phụ thuộc vào **context**: *tức là mỗi dạng content khác nhau thì có cách mã hóa input khác nhau*
```text
HTML context       → HTML encoding
URL context        → URL encoding
JavaScript context → JavaScript encoding
CSS context        → CSS encoding
```
>Đây là lý do một payload XSS dùng được ở vị trí này **không nhất thiết dùng được ở vị trí khác**. *--> tại sao ?*
>   -> do là ...

### 3. Response headers
-Nếu response không chứa HTML hoặc JavaScript:
```http
Content-Type: application/json
```
   - Có thể kết hợp:
```http
X-Content-Type-Options: nosniff
```
-Ý tưởng:
```text
Server nói:
"Đây là JSON"

Browser:
"Được, tôi không tự đoán nó là HTML/JavaScript"
```
>tức là với *http reponse*, phần *header* với trường *content-type* sẽ giúp browser biết được <u>kiểu dữ liệu mà server trả về là gì</u>: là js , json , text hay html .....

### 4. CSP
Đây là **lớp phòng thủ cuối cùng**:
*đọc thêm ở đây-->* [[CSP]]
```text
Input validation
       ↓
Output encoding
       ↓
Security headers
       ↓
CSP
```
CSP không nên là biện pháp duy nhất để chống XSS.
