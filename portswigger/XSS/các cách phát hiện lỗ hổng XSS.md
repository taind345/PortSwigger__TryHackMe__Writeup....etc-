
Đoạn này nói về **cách phát hiện 3 loại XSS**:
```text
XSS
│
├── Reflected XSS
│   └── Gửi input → server trả lại ngay trong response
│
├── Stored XSS
│   └── Gửi input → server lưu lại → trả về sau
│
└── DOM-based XSS
    └── JavaScript phía trình duyệt lấy input → đưa vào DOM/sink nguy hiểm
```

### 1. Reflected / Stored XSS

Ý chính: **Không thể dùng một payload XSS cho mọi vị trí. Phải biết input đang nằm ở context nào.**
>*những điểm chính cần phải nắm:*
>	- thứ nhất là cần <u>gửi input</u>--> <u>server</u>
>	- sever sẽ trả lại <u>response </u>
>	- mình sẽ check xem cái <u>input có ở trong response ko</u>. nếu có thì nằm ở đâu(*context* nào?)

Ví dụ: 
-ban đầu mình nhập input "xYz123"
-sau đó quan sát response , mình thấy được nó trả về input nằm trong 1 thẻ html
```html
<input value="xYz123">
```
-chọn Payload phù hợp có thể cần thoát khỏi attribute:
```html
" autofocus onfocus=alert(1) x="
```
*-> mã độc js lúc này đã bị chèn vào trình duyệt* 
### 2. DOM-based XSS từ URL
-kiến thức cần nắm [[DOM]]
**Ví dụ** 
```javascript
const name = location.search;
document.write(name);
```
-Gửi input search qua url 
```text
/page?name=xYz123
```
-Cách kiểm tra:
```text
1. Gắn chuỗi duy nhất vào URL2. Mở Developer Tools
2. Tìm chuỗi đó trong DOM
3. Xem nó đi vào đâu
4. Kiểm tra sink có nguy hiểm không
```

> *-?? DOM khác đếch gì reflected, chăng phải nó cũng check HTML , js của trình duyệt hay sao??*
>   - câu trả lời đơn giản là với *reflected* thì *input*--> *server*; và input nằm trong đâu đó trong *response*
>   - còn đối với  *DOM* thì input--> *đi thẳng vào mã DOM js*- Như ví dụ trên, *input* đi <u>vào luôn phần code DOM </u>thao tác trực tiếp vs html của trình duyệt. Và ở đây input nó ko đi tới phía server :D

> *Reflected:*
> Input → Server → HTTP Response → Browser → XSS
> 
> *DOM*:
> Input → Client-side JavaScript → DOM/JS Sink → XSS


### khái niệm quan trọng nhát là source và sink
Đây là nền tảng để hiểu DOM XSS:
```
SOURCE                         SINK
Nguồn dữ liệu                  Nơi dữ liệu được đưa vào
│                                    │
├── location.search              ├── innerHTML
├── location.hash                ├── document.write()
├── document.cookie            ├── eval()
└── postMessage                └── setTimeout()
          │
          └──────────────→ Nếu dữ liệu không được xử lý an toàn
                            → DOM XSS
```

- *Source*: nguồn dữ liệu đi vào
- *SINK*: có thể hiểu là đích
Ví dụ:
```javascript
let input = location.search; // source
element.innerHTML = input;//sink : nội dung html của element
```

### Tại sao DOM XSS khó hơn?
-Với URL:
```text
URL → JavaScript → DOM
```

-->có thể <u>dễ dàng thử input</u> và theo dõi.

-Nhưng DOM thì việc phát hiện khó hơn , như ví dụ duới đây
```javascript
document.cookie
setTimeout(input, 1000)
```
  - Input không nhất thiết xuất hiện trong URL.
  - Sink không nhất thiết là HTML.
  - Phải đọc và phân tích JavaScript để hiểu luồng dữ liệu.
    
Vì vậy Burp Scanner sử dụng:
```text
Static Analysis
      +
Dynamic Analysis
      ↓
Tự động theo dõi luồng dữ liệu
      ↓
Phát hiện DOM XSS
```
>- thực ra nó muốn nói là 
>	- với *reflected* : mày có thể lọc data trong response bằng grep để xem có input trong response hay ko
>	- nhưng với *DOM*: mày ko thể grep 1:1 được.Mày phải đọc mã JS của trang web, để biết cái biến đầu vào nó có nằm trong DOM hay ko.Mà nhiều khi code nó đồ sộ vl thì mày cũng ko thể nào *Static analysis* mà tìm ra biến *input* nó có nằm trong DOM hay ko.Do đó mày cần *dynamic analysis* để có thể theo dõi luồng data nó đi tới đâu để có thể phát hiện DOM XSS

### Bài tập kiểm tra
Đoạn code này có khả năng XSS không?
```javascript
const input = location.hash.substring(1);
document.getElementById("output").innerHTML = input;
```

Hãy trả lời 3 câu:
1. **Source là gì?**  // location.hash...
2. **Sink là gì?** // phần tử html có id="ouput", data từ source đi vào nội dụng html của "ouput"
3. **Đây là loại XSS nào?** // DOM xss
4. --> ko gây xss được do input nó bị lcoj có còn 1 ký tự thì làm sao có thể chèn mã js được 
