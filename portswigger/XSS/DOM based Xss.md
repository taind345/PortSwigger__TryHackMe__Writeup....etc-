## 1-tổng quan
-trước tiên ta cần hiểu tổng quan ![[điểm chung của các loại XSS]]
-**DOM-based XSS** xảy ra khi JavaScript phía trình duyệt lấy dữ liệu từ một nguồn mà attacker kiểm soát (Source) rồi đưa dữ liệu đó vào một điểm xử lý nguy hiểm (Sink), khiến JavaScript độc hại được thực thi. Khác với Reflected XSS và Stored XSS, dữ liệu gây XSS *không cần phải đi qua server rồi xuất hiện trong HTTP response.*
Ví dụ:
```javascript
let input = location.search;
document.getElementById("output").innerHTML = input;
```
Nếu attacker tạo URL:
```text
/page?name=<img src=x onerror=alert(1)>
```
thì luồng là:
```text
URL
 ↓
location.search       ← Source
 ↓
input
 ↓
innerHTML             ← Sink
 ↓
DOM thay đổi
 ↓
JavaScript thực thi
```

---
## 2.xác định sink
**Source** là nơi JavaScript lấy dữ liệu: `location.search`, `location.hash`, `location`, `document.cookie`, `postMessage`... **Sink** là nơi dữ liệu được đưa vào và có khả năng gây hành vi nguy hiểm: `innerHTML`, `document.write()`, `eval()`, `setTimeout()`...
Khi kiểm tra DOM XSS, bạn<u> cần phân biệt hai loại sink.</u>

**1. HTML sink:** Ví dụ:
```javascript
output.innerHTML = location.search;
```
Bạn đặt một chuỗi duy nhất vào URL:
```text
?page=x7Kp92Qa
```
Sau đó mở **Developer Tools → Elements** và tìm `x7Kp92Qa` trong DOM. Không nên dùng **View Source**, vì View Source hiển thị HTML ban đầu từ server, còn DOM có thể đã bị JavaScript thay đổi sau khi trang tải.
Nếu thấy:
```html
<div id="output">x7Kp92Qa</div>
```
thì bạn biết dữ liệu từ `location.search` đã đi vào DOM. Tiếp theo cần xác định context và thử xem có thể phá context hay không. Ví dụ nếu dữ liệu nằm trong:
```html
<input value="x7Kp92Qa">
```
thì bạn có thể kiểm tra việc chèn dấu `"` để xem có thoát được khỏi attribute hay không.

**2. JavaScript execution sink:** Ví dụ:
```javascript
let code = location.hash;
eval(code);
```
Ở đây input có thể **không xuất hiện trong DOM**, nên không thể chỉ dùng `Ctrl + F` trong Elements để tìm. Bạn phải mở **Sources → JavaScript**, tìm nơi `location.hash` được đọc, đặt **breakpoint**, rồi theo dõi dữ liệu đi qua các biến:
```text
location.hash
      ↓
    input
      ↓
    code
      ↓
    eval()
```
Đây gọi là theo dõi **taint flow**: dữ liệu từ **Source → các biến trung gian → Sink**.
Cách nhớ toàn bộ DOM XSS:
```text
Source
  ↓
JavaScript xử lý
  ↓
Biến trung gian
  ↓
Sink
  ↓
JavaScript execution
```
Một điểm cần chú ý là [[encoding#1. URL Encoding]]. Ví dụ browser có thể encode dữ liệu trong `location.search` hoặc `location.hash` trước khi JavaScript xử lý. Nếu payload bị encode thành dạng như `%3Cscript%3E`, nó có thể không còn được browser hiểu là HTML/JavaScript nữa, khiến XSS không hoạt động.

**DOM Invader** <u>của Burp Suite có thể tự động hỗ trợ việc theo dõi luồng dữ liệu Source → Sink,</u> thay vì phải tự đọc và debug những đoạn JavaScript dài hoặc đã bị minify.
Câu chốt:
>tóm lại là , *mình cần xác định sink*, có 2 loại phổ biến là *html sink* và *js code sink*
>Với *html sink* thì mình dùng tab *elenment để check* với *js code sink* thì *dùng tab cosole để xem sink nó nằm ở đâu*, theo dõi cái input nó tuồn tới được các đoạn nào trong DOM js


## 3- Sources and sinks in third-party dependencies
### Những sink nguy hiểm từ thư viện phổ biến
**JQuery**
- `attr()` – nếu gán `href` từ URL, chèn `javascript:alert()`.
- `$()` – selector sink: nếu nhận input từ `location.hash`, tạo DOM XSS (phiên bản cũ không chặn `#`).  
  ```js
  $(location.hash) // nguy hiểm nếu hash chứa <img src=x onerror=alert(1)>
  ```
 >*hiểu như thế nào về $() trong JQuery??*
-đại khái là *$(#)* nó giúp truy cập nhanh html như kiểu *getelementbyiID*
 
**AngularJS** 
- Khi có `ng-app`, biểu thức trong `{{ }}` được thực thi, không cần dấu `<>` hay sự kiện.

**Sink kinh điển (cả vanilla JS lẫn thư viện)**
`document.write()`, `innerHTML`, `outerHTML`, `insertAdjacentHTML`, `eval()`.
jQuery: `html()`, `append()`, `after()`, `prepend()`, `replaceWith()`…

---
### ví dụ
 **1-** *jQuery `attr()` – sink `href` lấy từ URL*
```html
<a id="backLink" href="/home">Quay lại</a>
<script>
  $(function() {
    var url = new URLSearchParams(window.location.search).get('returnUrl');
    $('#backLink').attr('href', url);
  });
</script>
// cái thằng trên kia có tác dụng gán url cho href #backlink
```
URL gửi cho nạn nhân:  
`https://site.com/page?returnUrl=javascript:alert(document.cookie)`  
→ Khi bấm “Quay lại”, JavaScript trong href chạy.

 **2-** *jQuery `$()` selector sink – từ `location.hash`*
```js
$(window).on('hashchange', function() {
  var element = $(location.hash);  // sink
  element[0].scrollIntoView();
});
// ở đây mình hiểu nó lấy hash của 1 element xong rồi nó hiển thị dạng embeded ra màn hình cái element đó
```
Dùng iframe tự động thêm hash chứa XSS:
```html
<iframe src="https://victim.com/page#" onload="this.src+='<img src=x onerror=alert(1)>'"></iframe>
```
→ Phiên bản jQuery cũ sẽ parse hash thành HTML, tạo thẻ `<img>` và lỗi `onerror` kích hoạt alert.

>chưa hiểu cho lắm??
>-*iframe* : giúp nhúng 1 trang web , hay link bên vàotrong trang web hiện tại .Nó giống chức năng embeded của obsidian
>-*onload* là event được kích hoạt khi nội dung của event đó được tải xong


**3-** *AngularJS – biểu thức trong `{{ }}` không cần `<>`*
**Bối cảnh:** Trang dùng `ng-app`, hiển thị nội dung từ URL vào template.
Ví dụ code server trả về HTML như sau:
```html
<body ng-app>
  <p>Bạn tìm: {{ searchQuery }}</p>
  <!-- searchQuery được chèn trực tiếp từ URL param 'q' -->
  -- đại khái nó lấy từ location.search --> đưa vào {{}}
</body>
```
Nếu ứng dụng không lọc, attacker truyền:
`?q={{constructor.constructor('alert(1)')()}}`
→ AngularJS sẽ thực thi script bên trong

**4-** *Sink kinh điển (Vanilla JS)*

| Sink               | Code lỗi                                                        | Input khai thác                     |
| ------------------ | --------------------------------------------------------------- | ----------------------------------- |
| `document.write()` | `document.write(location.hash.slice(1));`                       | URL `#<img src=x onerror=alert(1)>` |
| `innerHTML`        | `document.getElementById('box').innerHTML = params.get('msg');` | `?msg=<img src=x onerror=alert(1)>` |
| `eval()`           | `eval('var x = "' + params.get('x') + '";');`                   | `?x=";alert(1);//`                  |
>*document write*: nó sẽ cho phép nhận đầu vào là script và thực thi.Như trên ví dụ kia , input ta dùng hash của thẻ image với envent handler
>với *innerHTML* thì ta thường chèn vào 1 element có event handler
>với *eval()*, thì ta hiểu *eval(..string..)* sẽ giúp thực thi string như mã Js


**5-** *Sink từ jQuery (thường gặp)*
Tất cả các hàm dưới đây nếu nhận input không sạch sẽ biến thành sink XSS:
```js
// Vulnerable nếu userInput = "<img src=x onerror=alert(1)>"
$('#area').html(userInput);      // gán HTML trực tiếp
$('#area').append(userInput);    // chèn thêm HTML
$('#area').after(userInput);     // chèn sau phần tử
$('#area').prepend(userInput);   // chèn đầu phần tử
$('#area').replaceWith(userInput); // thay thế toàn bộ
// 
```


###  Cách phòng tránh tối giản
- **Không** đưa dữ liệu không tin cậy vào các sink trên.
- Dùng hàm an toàn: `textContent` thay `innerHTML`.
- Với URL: kiểm tra giao thức (chỉ cho phép `http:`, `https:`).
---

### Bài tập 
**Đề bài:** Cho đoạn code sau từ một trang web:
```html
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<a id="profileLink" href="/profile">My Profile</a>
<script>
  $(function() {
     var returnUrl = new URLSearchParams(window.location.search).get('next');
     // láy tham số từ url từ search
     $('#profileLink').attr('href', returnUrl);
     //truy cập vào cái thẻ profile link để thế returnURL thành link cho thẻ đó
  });

</script>
```

**Yêu cầu:**  
1. Xác định **nguồn** (source) và **sink** trong đoạn code. 
	souce là trường next location.search , sink là '#profilelink'
2. Tạo một URL khai thác để khi nạn nhân bấm vào link "My Profile" thì hiện alert với nội dung `document.cookie`.  
	 ``` html
	 ?next=javascript:alert(document.cookie)
	 ```
3. Đề xuất cách sửa lỗi ngắn gọn nhất (chỉ cần 1-2 dòng code).
```js

$('#profileLink').attr('href', /^https?:\/\//.test(returnUrl) ? returnUrl : '/profile');
```
_Kiểm tra URL bắt đầu `http://` hoặc `https://`, không đúng thì về mặc định an toàn._
## Thực hành
### lab 1 
[[XSS portswigger lab]]

### lab2
làm tiếp lab về nội dung phần này ở đây [[XSS portswigger lab]]
