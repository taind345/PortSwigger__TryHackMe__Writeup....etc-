### định nghía stored xss
**Stored XSS** (hay **Persistent XSS / Second-order XSS**) có bản chất là: **attacker gửi dữ liệu độc hại → server lưu dữ liệu đó → sau này server đưa dữ liệu đã lưu vào response → browser của người dùng thực thi nó**.
-Ví dụ website có phần bình luận. Bạn gửi:
```http
POST /post/comment
comment=This post was helpful
```
Server lưu comment vào database:
```text
Database:
"This post was helpful"
```
Khi người khác mở bài viết:
```text
Database → Server → HTTP Response → Browser
```
Response:
```html
<p>This post was helpful</p>
```
Nếu attacker gửi:
```html
<script>alert(1)</script>
```
thì server lưu nó:
```text
Database:
<script>alert(1)</script>
```
Sau đó bất kỳ người dùng nào mở bài viết đều nhận:
```html
<p><script>alert(1)</script></p>
```
và JavaScript chạy trong browser của họ.
Điểm cốt lõi:
```text
Reflected XSS:
Input → Server → Response ngay lập tức → Browser
Stored XSS:
Input → Server → Database/Storage → Response sau đó → Browser
```

---

Ví dụ thực tế: attacker đăng một comment chứa payload. Attacker không cần gửi link độc hại cho từng người dùng. Chỉ cần chờ người dùng truy cập bài viết chứa comment đó. Đây là lý do Stored XSS thường nguy hiểm hơn Reflected XSS: **payload đã nằm sẵn trong ứng dụng và tự động được phân phối cho các nạn nhân**.
Ví dụ với người dùng cần đăng nhập: Reflected XSS cần dụ nạn nhân gửi request độc hại đúng lúc họ đang đăng nhập. Nếu nạn nhân chưa đăng nhập thì payload có thể không lấy được dữ liệu quan trọng. Nhưng với Stored XSS, payload nằm sẵn trên website; khi nạn nhân đăng nhập rồi truy cập trang chứa payload, XSS chạy trong session đang đăng nhập của họ.

---

### phương pháp xác định lỗ hổng với stored xss
Phần khó nhất khi tìm Stored XSS là **Entry Point và Exit Point**. **Entry Point** là nơi dữ liệu của attacker đi vào ứng dụng: comment, username, profile, form, POST body, URL, HTTP headers, email, dữ liệu từ bên thứ ba... **Exit Point** là nơi dữ liệu đó sau này xuất hiện trong response.
Ví dụ:
```text
Entry Point                  Storage              Exit Point
    │                           │                     │
Comment: "x7Kp92Qa"  →  Database/Server  →  Blog post response
```
Với *Reflected XSS,* bạn thường kiểm tra:
```text
Gửi input → Xem ngay response
```
*Nhưng với Stored XSS:*
```text
Gửi input
    ↓
Ứng dụng lưu input
    ↓
Đi đến một trang khác
    ↓
Tìm input đã lưu trong response
    ↓
Xác định context
    ↓
Thử payload
```
Ví dụ bạn nhập chuỗi duy nhất:
```text
x7Kp92Qa
```
vào username. Sau đó bạn kiểm tra các nơi khác: profile, comment, audit log, admin panel... Nếu chuỗi đó xuất hiện ở một response khác, bạn đã tìm thấy một liên kết:
```text
Username input → Admin page
```
Sau đó xác định context:
```html
<p>x7Kp92Qa</p>
```
→ HTML text context.
```html
<input value="x7Kp92Qa">
```
→ HTML attribute context.
```javascript
let name = "x7Kp92Qa";
```
→ JavaScript context.


-Sau đó thử payload phù hợp. Quy trình kiểm tra Stored XSS cuối cùng gần giống Reflected XSS; điểm khác lớn nhất là **input không xuất hiện ngay trong response ban đầu mà được lưu lại và xuất hiện trong một response sau đó**.

> Tóm lại là ntn
>*stored xss*:
>	-cần xác định entry point (~sourcre) và exit point(~sink)
	  - với thằng reflected thì nó trả luôn lại trong response--> lọc cái thằng response là xong
	  - nhưng với thằng stored xss thì phải xem nó tuần input ra những vị trí nào .Và cái input nó ko xuất hiện ở các response nằm sau đó, vd: comment trên blog

---
### bài tập && lab
*lab ở đây-->*[[XSS portswigger lab]]
**Bài tập:** Một ứng dụng có chức năng `Profile name`. Bạn nhập:
```text
x7Kp92Qa
```
Sau đó khi admin mở trang `/admin/users`, response chứa:
```html
<td>User: x7Kp92Qa</td>
```
Hãy xác định: **1. Entry point là gì? 2. Exit point là gì? 3. Dữ liệu có phải Stored XSS ngay không? 4. Bạn cần làm gì tiếp theo để xác định có XSS?**

1-entry point là trường user nơi nhập vào profile name
2-exit point là thẻ <td> <td> ở /admin/uers
3-có phải stored xss
4-chèn thêm một đoạn script để xem có thể thực thi đoạn script hay ko.
