### định nghĩa lại reflected xss
**Reflected XSS** là trường hợp: **input từ HTTP request → server nhận → server đưa ngay input đó vào HTTP response một cách không an toàn → trình duyệt render và có thể thực thi JavaScript**.
*xem thêm tại -->* [[các cách phát hiện lỗ hổng XSS]]
-<u>Ví dụ</u> ứng dụng có chức năng tìm kiếm:
```text
GET /search?term=gift
```
Server trả về:
```html
<p>You searched for: gift</p>
```
Bây giờ attacker thay `gift` bằng payload:
```text
/search?term=<script>alert(1)</script>
```
Server phản hồi:
```html
<p>You searched for: <script>alert(1)</script></p>
```
Browser nhận response và thực thi script. Luồng chính là:
*-->đây là kịch bản tấn công*
```text
Attacker tạo URL độc hại
        ↓
Victim click URL
        ↓
HTTP Request chứa payload
        ↓
Server phản chiếu payload ngay vào Response
        ↓
Browser parse Response
        ↓
JavaScript thực thi
```
>*--> tức là đường url độc hại sẽ khiến browser của nạn nhân <u>request input chứa payload</u> tới server.Sau đó server sẽ trả về html nhúng payload js.Browswer của người dùng sẽ thục thi mã js độc hại*

---

Đây chính là điểm khác với **DOM XSS** mà bạn vừa học:
```text
Reflected XSS:
Input → Server → HTTP Response → Browser → XSS

DOM XSS:
Input → JavaScript phía Browser → DOM Sink → XSS
```

---

**Impact** của Reflected XSS có thể rất lớn: nếu JavaScript chạy trong session của nạn nhân, attacker có thể thực hiện các hành động mà nạn nhân có quyền thực hiện, đọc dữ liệu nạn nhân có thể đọc, sửa dữ liệu nạn nhân có thể sửa hoặc thực hiện hành động dưới danh nghĩa nạn nhân. Tuy nhiên, Reflected XSS thường nhẹ hơn Stored XSS vì attacker cần một **cơ chế delivery** như gửi link độc hại qua email, tin nhắn, website hoặc mạng xã hội. Với Stored XSS, payload đã được lưu trong ứng dụng nên nạn nhân chỉ cần truy cập trang bị nhiễm là có thể bị tấn công.


---
### Cách tìm Reflected XSS thủ công 
-có thể hiểu như một quy trình 5 bước:
**Bước 1: Tìm mọi entry point.** Không chỉ *kiểm tra query parameter* như `?term=...`, mà còn *kiểm tra body của POST request*, *path URL* và đôi khi cả *HTTP headers*.

**Bước 2: Gửi một chuỗi ngẫu nhiên.** Ví dụ:
```text
GET /search?term=x7Kp92Qa
```
Chuỗi này cần ngắn, chỉ gồm chữ và số, nhưng đủ đặc biệt để bạn dễ tìm lại trong response.

**Bước 3: Tìm chuỗi đó trong HTTP response.** Nếu response chứa:
```html
<p>You searched for: x7Kp92Qa</p>
```
thì input đã được **reflected**.

**Bước 4: Xác định context.** Đây là phần rất quan trọng. Ví dụ input được phản chiếu ở đâu?
```html
<p>x7Kp92Qa</p>
```
→ *HTML text context.*
```html
<input value="x7Kp92Qa">
```
→ *HTML attribute context*.
```javascript
let name = "x7Kp92Qa";
```
→ *JavaScript string context*.
Mỗi <u>context cần cách thoát và payload khác nhau</u>. Vì vậy không có một payload XSS duy nhất dùng được ở mọi nơi.
**Bước 5: Thử payload phù hợp.** Ví dụ nếu input được phản chiếu trực tiếp trong HTML text và không được encode:
```html
<script>alert(document.domain)</script>
```
Nếu response chứa nguyên vẹn payload và browser thực thi được thì đã xác nhận XSS. Sau đó nên kiểm tra trên browser thật, vì <u>thấy payload xuất hiện trong Burp Repeater chưa chắc đồng nghĩa với việc nó thực sự được browser thực thi</u>.

*phân biệt lại các loại xss 1 lần nũa cho nhớ*
> *-Reflected XSS* = input từ request được server phản chiếu ngay vào response không an toàn.*--> đó là lý do nó được gọi là refected :D*
 *-Stored XSS* là: input được server lưu lại trước, sau đó xuất hiện trong một response ở thời điểm khác.
 -*Self-XSS* là: **nạn nhân phải tự nhập hoặc tự paste payload vào browser**, nên thường không được xem là Reflected XSS thông thường và impact thấp hơn.


---

### **Bài tập:** Với request:
```http
GET /search?query=Tai HTTP/1.1
```
và response:
```html
<h1>Search result for: Tai</h1>
```
hãy trả lời: **1. Input nằm ở đâu? 2. Nó có bị reflected không? 3. Đây là context gì? 4. Nếu bạn muốn kiểm tra XSS, bước tiếp theo nên làm gì?**
```txt
1-input nằm ở ?query
2-có bị reflected--> do trong response có chứa input mà 
3-context ở đây là thẻ html text content
4-muốn kiểm tra , thì cần gửi thử hàm <script> alert() <script> vào input thửu xem nó có chạy được cái mã js hay ko . 
```


### ôn tập nhanh
```
Reflected XSS
│
├── Definition
│   └── Request input → Immediate HTTP Response → Browser executes
│
├── Entry Point
│   ├── URL Query Parameter
│   ├── POST Body
│   ├── URL Path
│   └── HTTP Headers
│
├── Reflection
│   └── Input từ Request xuất hiện lại trong Response
│
├── Context
│   ├── HTML Text
│   ├── HTML Attribute
│   ├── JavaScript String
│   └── CSS / URL
│
├── Testing
│   ├── Submit unique random value
│   ├── Find reflection in Response
│   ├── Identify context
│   ├── Test candidate payload
│   ├── Try alternative payload
│   └── Verify in real Browser
│
├── Impact
│   ├── Perform actions as victim
│   ├── Read accessible data
│   ├── Modify accessible data
│   └── Act as victim
│
├── Delivery
│   ├── Malicious URL
│   ├── Email
│   ├── Message
│   └── Attacker-controlled website
│
├── Compared with Stored XSS
│   ├── Reflected → Immediate response
│   └── Stored → Saved → Later response
│
└── Compared with DOM XSS
    ├── Reflected → Server reflects input
    └── DOM → Client-side JavaScript processes input
```

### Làm lab
xem tại đây -->[[XSS portswigger lab]]
