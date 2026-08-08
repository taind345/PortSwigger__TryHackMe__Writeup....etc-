**Đúng, chủ yếu là tab `Sources`.** Nhưng nói chính xác hơn: `Sources` giúp bạn xem các mã JavaScript **đã được tải vào browser và đang có khả năng chạy trong trang đó**.
Ví dụ trang web tải:
```text
https://example.com/
    ├── index.html
    ├── app.js
    ├── jquery.js
    └── chunk-abc.js
```
Trong **DevTools → Sources**, bạn thường có thể xem các file JavaScript đó và đọc:
```javascript
location.hash
document.cookie
fetch(...)
XMLHttpRequest
innerHTML
eval(...)
```
Tức là bạn có thể phân tích:
```text
Browser
   ↓
JavaScript được tải
   ↓
Đọc dữ liệu từ browser
   ├── URL
   ├── Cookie
   ├── DOM
   ├── localStorage
   └── sessionStorage
        ↓
Thao tác với trang / gửi request
```
Tuy nhiên, **không phải lúc nào bạn cũng thấy “tất cả mã JS” dễ dàng**. JavaScript có thể bị **minify**, **bundle** thành một file rất lớn, tải động sau khi bạn tương tác, hoặc được tạo động bởi JavaScript khác. Khi đó bạn dùng **Sources → Ctrl + Shift + F** để tìm toàn bộ mã nguồn đã tải, hoặc **Network** để xem các file `.js` được tải.
Trong bối cảnh học **DOM XSS**, quy trình thường là:
```text
Sources
  ↓
Tìm Source
(location, location.hash, location.search...)
  ↓
Đặt Breakpoint
  ↓
Theo dõi dữ liệu qua các biến
  ↓
Tìm Sink
(innerHTML, eval, document.write...)
```
Ví dụ:
```javascript
const input = location.hash.substring(1);
const data = input;
document.getElementById("output").innerHTML = data;
```
Bạn có thể dùng **Sources** để theo dõi:
```text
location.hash
     ↓
   input
     ↓
    data
     ↓
 innerHTML
```
**Tóm lại:** `Sources` là nơi chính để xem và debug JavaScript phía client; `Network` giúp bạn biết JavaScript nào đã được tải; `Elements` giúp xem DOM sau khi JavaScript đã thay đổi nó; `Console` dùng để chạy thử và kiểm tra.