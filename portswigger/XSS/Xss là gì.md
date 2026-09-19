*phần này mình sẽ dịch trực tiếp từ portswiggeer*
## Cross-site scripting là gì?

Cross-site scripting (còn được gọi là XSS) là một lỗ hổng bảo mật web cho phép kẻ tấn công <u>can thiệp vào các tương tác mà người dùng thực hiện với ứng dụng</u> dễ bị tấn công. Nó <u>cho phép kẻ tấn công vượt qua chính sách same origin</u> – chính sách được thiết kế để cách ly các trang web khác nhau với nhau. Các lỗ hổng cross-site scripting thường cho phép kẻ tấn công giả mạo người dùng nạn nhân, thực hiện bất kỳ hành động nào mà người dùng có thể thực hiện và truy cập vào bất kỳ dữ liệu nào của người dùng đó. Nếu người dùng nạn nhân có quyền truy cập đặc quyền trong ứng dụng, kẻ tấn công có thể chiếm toàn quyền kiểm soát tất cả chức năng và dữ liệu của ứng dụng.
*-> đại khái là:
	- nó bên client-side nên nó sẽ chèn mã js vào trang web, người dùng khi kích hoạt cái mã js đó trên trình duyệt thì nó sẽ rất nguy hiểm*
### XSS hoạt động như thế nào?
Cross-site scripting hoạt động bằng cách thao tác một trang web dễ bị tấn công để nó trả về JavaScript độc hại cho người dùng.

### XSS – (Proof of Concept)

Bạn có thể xác nhận hầu hết các loại lỗ hổng XSS bằng cách chèn một payload khiến trình duyệt của chính bạn thực thi một đoạn JavaScript tùy ý. Từ lâu, cách làm phổ biến là sử dụng hàm *alert()* .

Thật không may, có một chút trở ngại nếu bạn sử dụng Chrome. Từ phiên bản 92 trở đi (20/07/2021), các iframe khác nguồn gốc bị ngăn gọi alert(). Vì các iframe này được dùng để xây dựng một số cuộc tấn công XSS nâng cao hơn, đôi khi bạn sẽ cần sử dụng một payload PoC thay thế. Trong trường hợp này, chúng tôi khuyên dùng hàm *print()*. 

## Các kiểu tấn công XSS là gì?

**Có ba kiểu tấn công XSS chính:**
- *Reflected XSS (XSS phản chiếu):* tập lệnh độc hại đến từ chính yêu cầu HTTP hiện tại.
- *Stored XSS (XSS lưu trữ):* tập lệnh độc hại đến từ cơ sở dữ liệu của trang web.
- *DOM-based XSS* (XSS dựa trên DOM): lỗ hổng tồn tại trong mã phía máy khách chứ không phải mã phía máy chủ.
>Minh vẫn chưa hiểu phần này lắm, chắc phải đào sâu hơn mới hiểu được 

### Reflected cross-site scripting

Reflected XSS là dạng cross-site scripting <u>đơn giản</u> nhất. Nó phát sinh khi một ứng dụng nhận dữ liệu trong một yêu cầu HTTP và đưa dữ liệu đó vào phản hồi tức thời một cách không an toàn.*-> tức là script được đưa vào qua url, hay request từ client của trình duyệt. Lúc này trình duyệt sẽ thực thi mã js độc hại với trang web.*
**-> cái mã js này nó có đi vào backend của server ko** ?
-ok đây là 1 trang web ko an toàn
``` html
//https://insecure-website.com/status?message=All+is+well
<p>Status: All is well.</p>
```
*-> có thể thấy trang web này cho phép ta truyền text dưới dạng tham số vào url*
-do đó thằng hacker có thể chèn vào tham số của url một script<> js-> nó sẽ đi vào phía backend của server và nó sẽ skibidi cái trang web đó
``` html
https://insecure-website.com/status?message=<script>/*+Bad+stuff+here...+*/</script>

<p>Status: <script>/* Bad stuff here... */</script></p>
```

### Stored cross-site scripting
Stored XSS (còn được gọi là XSS dai dẳng hay XSS bậc hai) phát sinh khi một ứng dụng nhận dữ liệu từ một nguồn không đáng tin cậy và sau đó đưa dữ liệu đó vào các phản hồi HTTP về sau một cách không an toàn.*-> cái mã js độc hại được lưu luôn trong backend của server*

Dữ liệu nói trên có thể được gửi đến ứng dụng thông qua các yêu cầu HTTP; ví dụ: <u>bình luận</u> trên một bài đăng blog, <u>biệt danh người dùng </u>trong phòng chat, hoặc chi tiết liên hệ trên đơn hàng của khách. Trong các trường hợp khác, dữ liệu có thể đến từ các nguồn không đáng tin cậy khác; ví dụ: một ứng dụng webmail hiển thị thư nhận qua SMTP, một ứng dụng tiếp thị hiển thị bài đăng mạng xã hội, hoặc một ứng dụng giám sát mạng hiển thị dữ liệu gói tin từ lưu lượng mạng.*--> đại khái là đưa mã js vào những nơi có thể tuồn mã js vaò được, như POST chẳng hạn*

VÍ DỤ:Một ứng dụng bảng tin cho phép người dùng gửi tin nhắn, được hiển thị cho những người dùng khác:-> tức cái đoạn txt này nó sẽ đi qua các trình duyệt của các client khác 
``` html
<p>Hello, this is my message!</p>
```
do cái backend nó ko xử lý đầu vào, nên m có thể chèn <u>script</u> vào --> phát tán cho các người dùng khác click vào
``` html
<p><script>/* Bad stuff here... */</script></p>
```

Đọc thêm
- Stored cross-site scripting
- Cross-site scripting cheat sheet

### DOM-based cross-site scripting

DOM-based XSS phát sinh khi một ứng dụng chứa một số JavaScript phía client xử lý dữ liệu từ nguồn không đáng tin cậy một cách không an toàn, <u>thường bằng cách ghi dữ liệu trở lại DOM.</u>*--> là sao  ??*

Trong ví dụ sau, một ứng dụng sử dụng JavaScript để đọc giá trị từ một trường nhập liệu và ghi giá trị đó vào một phần tử trong HTML:
```  js
var search = document.getElementById('search').value; // lấy gia trị nhập vào
var results = document.getElementById('results');
results.innerHTML = 'You searched for: ' + search;// trường kết quả lấy dữ liệu từ input 

```
Nếu kẻ tấn công có thể kiểm soát giá trị của trường nhập liệu, chúng có thể dễ dàng tạo một giá trị độc hại khiến tập lệnh của mình được thực thi:

You searched for: 
```html  
<img src=1 onerror='/* Bad stuff here... */'>
```

>Trong trường hợp điển hình, trường nhập liệu sẽ được điền từ một phần của yêu cầu HTTP, chẳng hạn như tham số chuỗi truy vấn URL, cho phép kẻ tấn công thực hiện tấn công bằng một URL độc hại, tương tự như reflected XSS.*=>tham số url được làm 1 phânf của input*


## XSS có thể được sử dụng để làm gì?
*phần này thuần lý thuyết*
Kẻ tấn công khai thác lỗ hổng cross-site scripting thường có thể:
- Mạo danh hoặc giả dạng người dùng nạn nhân.
- Thực hiện bất kỳ hành động nào mà người dùng có thể thực hiện.
- Đọc bất kỳ dữ liệu nào mà người dùng có thể truy cập.
- Đánh cắp thông tin đăng nhập của người dùng.
- Thực hiện phá hoại giao diện trang web (virtual defacement).
- Chèn chức năng trojan vào trang web.

**Tác động của các lỗ hổng XSS**
Tác động thực tế của một cuộc tấn công XSS thường phụ thuộc vào bản chất của ứng dụng, chức năng và dữ liệu của nó, cũng như trạng thái của người dùng bị xâm phạm. Ví dụ:

- Trong một ứng dụng kiểu brochureware, nơi tất cả người dùng đều ẩn danh và mọi thông tin đều công khai, tác động thường sẽ rất nhỏ.
- Trong một ứng dụng chứa dữ liệu nhạy cảm, chẳng hạn như giao dịch ngân hàng, email hoặc hồ sơ y tế, tác động thường sẽ nghiêm trọng.
- Nếu người dùng bị xâm phạm có đặc quyền cao trong ứng dụng, thì tác động thường sẽ ở mức đặc biệt nghiêm trọng, cho phép kẻ tấn công chiếm toàn quyền kiểm soát ứng dụng dễ bị tổn thương và xâm phạm tất cả người dùng cùng dữ liệu của họ.




### Content Security Policy (CSP)

Content Security Policy (CSP) là một cơ chế của trình duyệt nhằm giảm thiểu tác động của cross-site scripting và một số lỗ hổng khác. Nếu một ứng dụng sử dụng CSP có chứa hành vi giống XSS, thì CSP có thể cản trở hoặc ngăn chặn việc khai thác lỗ hổng. Thường thì CSP có thể bị vượt qua để cho phép khai thác lỗ hổng cơ bản.

Đọc thêm
- Content Security Policy

Dangling markup injection

Dangling markup injection là một kỹ thuật có thể được sử dụng để thu thập dữ liệu giữa các tên miền trong các tình huống không thể khai thác cross-site scripting đầy đủ, do các bộ lọc đầu vào hoặc các biện pháp phòng vệ khác. Kỹ thuật này thường có thể được khai thác để chiếm đoạt thông tin nhạy cảm hiển thị cho những người dùng khác, bao gồm các token CSRF có thể được sử dụng để thực hiện các hành động trái phép thay mặt người dùng.

Đọc thêm
- Dangling markup injection

### Cách ngăn chặn tấn công XSS

Việc ngăn chặn cross-site scripting đôi khi đơn giản nhưng cũng có thể khó hơn nhiều tùy thuộc vào độ phức tạp của ứng dụng và cách ứng dụng xử lý dữ liệu mà người dùng có thể kiểm soát.

Nói chung, việc ngăn chặn hiệu quả các lỗ hổng XSS thường đòi hỏi sự kết hợp của các biện pháp sau:

- Lọc đầu vào ngay khi nhận. Tại điểm nhận đầu vào của người dùng, hãy lọc càng chặt chẽ càng tốt dựa trên những gì được mong đợi hoặc là đầu vào hợp lệ.
- Mã hóa dữ liệu ở đầu ra. Tại điểm dữ liệu mà người dùng có thể kiểm soát được xuất ra trong các phản hồi HTTP, hãy mã hóa đầu ra để ngăn dữ liệu bị diễn giải như nội dung hoạt động. Tùy thuộc vào ngữ cảnh đầu ra, việc này có thể yêu cầu áp dụng kết hợp mã hóa HTML, URL, JavaScript và CSS.
- Sử dụng các tiêu đề phản hồi phù hợp. Để ngăn XSS trong các phản hồi HTTP không có ý định chứa bất kỳ HTML hoặc JavaScript nào, bạn có thể sử dụng các tiêu đề Content-Type và X-Content-Type-Options để đảm bảo trình duyệt diễn giải phản hồi theo đúng cách bạn mong muốn.
- Content Security Policy. Như một tuyến phòng thủ cuối cùng, bạn có thể sử dụng Content Security Policy (CSP) để giảm mức độ nghiêm trọng của bất kỳ lỗ hổng XSS nào vẫn còn tồn tại.

Đọc thêm
- Cách ngăn chặn XSS
- Phát hiện lỗ hổng XSS bằng trình quét lỗ hổng web của Burp Suite

Các câu hỏi thường gặp về cross-site scripting

Các lỗ hổng XSS phổ biến như thế nào?  
Lỗ hổng XSS rất phổ biến, và XSS có lẽ là lỗ hổng bảo mật web xảy ra thường xuyên nhất.

Các cuộc tấn công XSS phổ biến như thế nào?  
Rất khó để có được dữ liệu đáng tin cậy về các cuộc tấn công XSS trong thực tế, nhưng có lẽ nó ít bị khai thác thường xuyên hơn so với các lỗ hổng khác.

### Sự khác biệt giữa XSS và CSRF là gì?  
XSS liên quan đến việc khiến một trang web trả về JavaScript độc hại, trong khi CSRF liên quan đến việc dụ người dùng nạn nhân thực hiện các hành động mà họ không có ý định thực hiện.

Sự khác biệt giữa XSS và SQL injection là gì?  
XSS là lỗ hổng phía máy khách nhắm vào những người dùng khác của ứng dụng, trong khi SQL injection là lỗ hổng phía máy chủ nhắm vào cơ sở dữ liệu của ứng dụng.

Làm thế nào để ngăn chặn XSS trong PHP?  
Lọc đầu vào của bạn với danh sách trắng các ký tự được phép và sử dụng gợi ý kiểu hoặc ép kiểu. Thoát đầu ra của bạn bằng htmlentities và ENT_QUOTES cho ngữ cảnh HTML, hoặc dùng Unicode escape cho JavaScript trong ngữ cảnh JavaScript.

Làm thế nào để ngăn chặn XSS trong Java?  
Lọc đầu vào của bạn với danh sách trắng các ký tự được phép và sử dụng một thư viện như Google Guava để mã hóa HTML đầu ra cho ngữ cảnh HTML, hoặc sử dụng Unicode escape cho JavaScript trong ngữ cảnh JavaScript.