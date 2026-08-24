# 1-intro
Trong phòng học này, bạn sẽ học cách kiểm thử thủ công một ứng dụng web để tìm các vấn đề bảo mật chỉ bằng các công cụ tích hợp sẵn trên trình duyệt. <u>Trong nhiều trường hợp, các công cụ và kịch bản quét tự động sẽ bỏ sót nhiều lỗ hổng tiềm ẩn cũng như các thông tin hữu ích.</u>
*=> tức là mình bình thường dùng tool là được chứ ko cần phải đọc source code html mù mắt à =D*
Mục tiêu bài học
Hoàn thành phòng học này, bạn sẽ có thể:

* Sử dụng trình duyệt để xem mã nguồn dạng văn bản đọc được của một trang web
* Kiểm tra các phần tử trang và thay đổi chúng để xem các nội dung thường bị ẩn hoặc bị chặn *=> *
* Kiểm tra và kiểm soát luồng chạy JavaScript của trang *=> debugger*
* Nhận biết các yêu cầu mạng (network requests) do trang tạo ra *=> network*
* Nhận biết các dữ liệu được trang web lưu trữ cục bộ trên thiết bị *=> storage*

Yêu cầu tiên quyết
Bạn nên nắm vững nội dung từ các phòng học sau trước khi bắt đầu:
* Web Application Basics
* HTTP in Detail

Truy cập máy thực hành
Khởi chạy AttackBox bằng nút Start AttackBox và máy Lab bằng nút Start Lab Machine bên dưới. Sau đó, mở Mozilla Firefox trên AttackBox và truy cập địa chỉ [http://10.49.159.172](http://10.49.159.172) để vào trang web thực hành.
# 2- viewing the page source
Với tư cách là một penetration tester, vai trò của bạn khi đánh giá một website hoặc ứng dụng web là xác định các tính năng có thể tồn tại lỗ hổng và cố gắng khai thác chúng để kiểm tra xem lỗ hổng đó có thực sự tồn tại hay không. Các tính năng này thường là những phần trên website yêu cầu sự tương tác của người dùng.

Việc tìm kiếm các phần tương tác trên website có thể đi từ việc phát hiện một biểu mẫu đăng nhập cho đến việc kiểm tra thủ công mã JavaScript của trang web. Nơi bắt đầu tốt nhất là ngay trên trình duyệt của bạn; hãy khám phá website, ghi lại từng trang, khu vực, tính năng và kèm theo tóm tắt cho từng phần. Sau khi truy cập website Acme IT Support, bạn sẽ thấy trang tương ứng.

Một ví dụ về bản đánh giá trang web cho Acme IT Support sẽ trông như thế này:

| Tính năng | Endpoint | Tóm tắt |
| --- | --- | --- |
| Trang chủ | / | Trang này chứa tóm tắt về các hoạt động của Acme IT Support, cùng với ảnh tập thể nhân viên công ty. |
| Tin mới nhất | /news | Trang này chứa danh sách các bài viết tin tức mới xuất bản của công ty. Mỗi bài viết có một liên kết kèm số ID, ví dụ: /news/article?id=1 |
| Bài viết tin tức | /news/article?id=1 | Hiển thị bài viết tin tức cụ thể. Một số bài viết dường như bị khóa và chỉ dành riêng cho khách hàng trả phí (premium). |
| Trang liên hệ | /contact | Trang này chứa biểu mẫu để khách hàng liên hệ với công ty. Nó bao gồm các trường nhập tên, email, nội dung tin nhắn và nút gửi. |
| Khách hàng | /customers | Liên kết này chuyển hướng đến /customers/login. |
| Đăng nhập khách hàng | /customers/login | Trang này chứa biểu mẫu đăng nhập với các trường username và password. |
| Đăng ký khách hàng | /customers/signup | Trang này chứa biểu mẫu đăng ký người dùng với các trường nhập username, email, password và xác nhận password. |
| Đặt lại mật khẩu | /customers/reset | Biểu mẫu đặt lại mật khẩu với trường nhập địa chỉ email. |
| Trang quản trị khách hàng | /customers | Trang này liệt kê các ticket hỗ trợ mà người dùng đã gửi cho công ty và có thêm nút Create Ticket. |
| Tạo ticket | /customers/ticket/new | Trang này chứa biểu mẫu với ô nhập nội dung sự cố IT và tùy chọn tải lên file để tạo ticket hỗ trợ. |
| Tài khoản khách hàng | /customers/account | Trang này cho phép người dùng chỉnh sửa username, email và password của họ. |
| Đăng xuất | /customers/logout | Liên kết này đăng xuất người dùng khỏi khu vực dành cho khách hàng. |
*=> đây là các endpint*
Chúng ta sẽ bắt đầu tìm hiểu kỹ hơn về một số trang vừa phát hiện trong bài học tiếp theo.
# 3-viewing the page source
Mã nguồn trang (page source) là đoạn mã dạng văn bản đọc được mà web server trả về cho trình duyệt/client mỗi khi chúng ta gửi một yêu cầu. Mã này bao gồm HTML (HyperText Markup Language), CSS (Cascading Style Sheets) và JavaScript, có nhiệm vụ hướng dẫn trình duyệt hiển thị nội dung gì, trình bày ra sao và tạo tính tương tác nhờ JavaScript. Đối với mục đích của chúng ta, việc xem mã nguồn trang có thể giúp phát hiện thêm nhiều thông tin về ứng dụng web.
*đọc ở đây* [[0-java script]]
### Làm thế nào để xem mã nguồn trang?

Khi đang xem một trang web, bạn có thể nhấn chuột phải vào trang và chọn View Page Source (Xem nguồn trang) từ menu.

Hầu hết trình duyệt cũng hỗ trợ thêm tiền tố `view-source:` vào trước URL, ví dụ: `view-source:[https://www.google.com/](https://www.google.com/)`.

### Cùng xem mã nguồn trang!

Hãy thử xem mã nguồn trang chủ của website Acme IT Support. Việc giải thích toàn bộ những gì bạn thấy ở đây nằm ngoài phạm vi của room này, và bạn cần tham khảo các khóa học thiết kế/phát triển web để hiểu đầy đủ. Những gì chúng ta có thể làm là trích xuất các thông tin quan trọng đối với mình.

Ở đầu trang, bạn sẽ thấy đoạn mã bắt đầu bằng `<!--` và kết thúc bằng `-->`; đó là các chú thích (comment). Comment là thông điệp do lập trình viên để lại, thường dùng để giải thích mã nguồn cho người khác hoặc ghi chú/nhắc nhở cho chính họ. Các comment này không hiển thị trên trang web thực tế. Comment ở đây giải thích rằng trang chủ hiện tại chỉ là tạm thời trong khi trang mới đang được phát triển. Hãy truy cập trang web được đề cập trong comment để lấy flag đầu tiên.
![[Pasted image 20260812101721.png]]
Các<u> liên kết đến trang khác trong HTML được viết trong thẻ anchor</u> (các phần tử HTML bắt đầu bằng `<a`), và đường dẫn bạn được chuyển hướng đến được lưu trong thuộc tính `href`.
Ví dụ, bạn sẽ thấy liên kết đến trang liên hệ ở dòng 31.
![[Pasted image 20260812101727.png]]
Nếu xem tiếp mã nguồn bên dưới, bạn sẽ thấy một liên kết bị ẩn dẫn tới trang bắt đầu bằng `secr`; hãy truy cập liên kết này để lấy một flag khác. Trong thực tế tất nhiên bạn sẽ không nhận được flag, nhưng bạn có thể phát hiện một khu vực riêng tư mà doanh nghiệp dùng để lưu trữ thông tin công ty, nhân viên hoặc khách hàng.

<u>Các file bên ngoài như CSS, JavaScript và hình ảnh có thể được nhúng vào bằng mã HTML.</u> Trong ví dụ này, bạn sẽ nhận ra các file này đều được lưu trong cùng một thư mục.
![[Pasted image 20260812101739.png]]
Nếu bạn truy cập thư mục này trên trình duyệt, thông thường bạn sẽ thấy lỗi cấu hình: một trang trắng hoặc trang 403 Forbidden báo rằng bạn không có quyền truy cập. <u>Nhưng ở đây, tính năng liệt kê thư mục (directory listing) đã được bật, hiển thị toàn bộ các file trong thư mục</u>. Đôi khi đây không phải vấn đề lớn nếu tất cả file đều an toàn để công khai, nhưng trong một số trường hợp, các file sao lưu (backup), mã nguồn hoặc thông tin bảo mật khác có thể bị lộ tại đây. Trong trường hợp này, có một flag nằm trong file `flag.txt`.

Nhiều trang web ngày nay không được xây dựng từ đầu mà sử dụng các framework. Framework là tập hợp các đoạn mã được viết sẵn giúp lập trình viên dễ dàng tích hợp các tính năng phổ biến như blog, quản lý người dùng, xử lý biểu mẫu..., tiết kiệm nhiều thời gian phát triển.

Xem mã nguồn trang thường cung cấp manh mối liệu trang web có đang dùng framework hay không, cụ thể là framework nào và thậm chí cả phiên bản nào. Việc biết framework và phiên bản là một phát hiện quan trọng, vì có thể tồn tại các lỗ hổng công khai trên phiên bản đó nếu trang web chưa cập nhật bản mới nhất. Ở cuối trang, bạn sẽ tìm thấy một comment ghi thông tin về framework cùng phiên bản đang dùng, kèm liên kết đến trang chủ của framework. Khi truy cập trang chủ của framework, bạn sẽ thấy website hiện tại đã cũ. Hãy đọc thông báo cập nhật và sử dụng thông tin tìm được để phát hiện thêm một flag nữa.
![[Pasted image 20260812112904.png]]![[Pasted image 20260812113348.png]]
![[Pasted image 20260812113426.png]]
# 4-develper tool viewing the page source
Mọi trình duyệt hiện đại đều tích hợp Developer Tools. Đây là bộ công cụ hỗ trợ nhà phát triển web trong việc gỡ lỗi (debug) ứng dụng, đồng thời giúp bạn xem các thành phần bên dưới để hiểu trang web đang hoạt động thế nào. Dưới góc độ pentester, chúng ta có thể lợi dụng các công cụ này để hiểu rõ hơn về ứng dụng web. Bài này tập trung vào 3 tính năng chính: Inspector, Debugger và Network.

**Mở Developer Tools**
Cách truy cập Developer Tools khác nhau tùy thuộc vào từng trình duyệt. Nếu chưa biết cách mở, bạn có thể nhấn vào nút View Site ở góc trên bên phải nhiệm vụ này để xem hướng dẫn chi tiết cho trình duyệt đang dùng.

**Inspector** (Công cụ kiểm tra phần tử)
Mã nguồn trang (Page source) không phải lúc nào cũng phản ánh chính xác những gì hiển thị trên màn hình. CSS, JavaScript và tương tác của người dùng có thể làm thay đổi nội dung cũng như giao diện. Vì vậy, chúng ta cần một công cụ để xem chính xác trạng thái hiện tại của trang web. Thẻ Inspector cung cấp giao diện hiển thị trực tiếp (live view) tại thời điểm thực. Ngoài ra, bạn còn có thể chỉnh sửa và tương tác trực tiếp với các phần tử trên trang.

Trên trang web Acme IT Support, hãy nhấn vào mục News. <u>Bạn sẽ thấy 3 bài báo. Hai bài đầu đọc được bình thường, nhưng bài thứ ba bị che bởi một thông báo nổi yêu cầu đăng ký tài khoản premium</u>. Những hộp thoại đè lên nội dung như vậy được gọi là **paywall** (rào cản trả phí).
![[Pasted image 20260812113649.png]]
Hãy nhấp chuột phải vào thông báo premium đó (paywall) và chọn Inspect (Kiểm tra) từ menu để mở công cụ nhà phát triển.
![[Pasted image 20260812113707.png]]
Trong thẻ Inspector, bạn sẽ thấy các phần tử HTML tạo nên trang web.
![[Pasted image 20260812113715.png]]
Tìm phần tử DIV có class là premium-customer-blocker và nhấn vào nó. Tại khung Styles bên cạnh, bạn sẽ thấy các dòng mã CSS đang áp dụng cho phần tử này (ví dụ: margin-top: 60px, text-align: center). Thuộc tính cần chú ý là *display: block.* <u>Khi nhấp vào chữ block, bạn có thể đổi thành giá trị khác</u>. Hãy thử gõ none để làm ẩn hộp thoại này. Khi đó, nội dung bị ẩn bên dưới cùng một đoạn flag sẽ xuất hiện. Nếu phần tử chưa có trường display, bạn có thể nhấp vào dòng cuối cùng trong khung style để tự thêm vào.

Bạn có thể thoải mái thử nghiệm với Inspector để thay đổi bất kỳ thông tin nào trên giao diện web. Hãy lưu ý rằng mọi thao tác thay đổi này chỉ diễn ra trên trình duyệt của riêng bạn; khi bạn tải lại trang (refresh), giao diện sẽ quay về trạng thái mặc định.
![[Pasted image 20260812114916.png]]
# 5-debugger
[[devtool#debugger]]

Bảng này trong Developer Tools được dùng để gỡ lỗi (debug) JavaScript, giúp các nhà phát triển web tìm ra nguyên nhân vì sao một tính năng nào đó không hoạt động. Với vai trò là pentester, chúng ta có thể đào sâu vào mã nguồn JavaScript. <u>Trên Firefox và Safari, tính năng này có tên là Debugger, còn trên Google Chrome thì được gọi là Sources.</u>

Trên trang web Acme IT Support, hãy nhấn vào trang Contact. Mỗi khi trang được tải lại, bạn có thể nhận thấy một chớp đỏ xuất hiện rất nhanh trên màn hình. Chúng ta sẽ dùng thẻ Debugger để tìm hiểu xem chớp đỏ này là gì và liệu nó có chứa thông tin gì thú vị hay không. Thực tế khi làm pentester, bạn sẽ ít khi phải gỡ lỗi một chấm đỏ như thế này, nhưng bài tập này giúp bạn làm quen và thực hành với công cụ Debugger.

Ở cột bên trái trong thẻ Debugger,<u> bạn sẽ thấy danh sách tất cả các tài nguyên mà trang web hiện tại đang sử dụng</u><u></u>. Nếu nhấn vào thư mục assets, bạn sẽ thấy một tệp có tên flash.min.js. Nhấn vào tệp này để xem nội dung bên trong.
![[Pasted image 20260812130825.png]]
Thông thường khi xem các tệp JavaScript, bạn sẽ thấy toàn bộ mã nguồn nằm trên cùng một dòng do <u>tệp đã được tối ưu hóa (minimize) - tức là xóa bỏ toàn bộ khoảng trắng, thẻ tab và dòng mới để giảm dung lượng tệp</u>. <u>Tệp này cũng không ngoại lệ, thậm chí nó còn được làm xáo trộn mã (obfuscated)</u> để cố tình gây khó đọc và ngăn người khác sao chép.

Bạn có thể định dạng lại bằng tùy chọn Pretty Print (thường có biểu tượng hai dấu ngoặc nhọn `{ }`) để dễ nhìn hơn. Mặc dù vậy, do mã đã bị làm xáo trộn, việc hiểu toàn bộ logic của tệp vẫn khá khó khăn.<u> Khi cuộn xuống cuối tệp flash.min.js, bạn sẽ thấy dòng mã:</u> `flash['remove']();`.
![[Pasted image 20260812131047.png]]
Đoạn JavaScript nhỏ này chính là thứ đã xóa thông báo màu đỏ khỏi trang web ngay sau khi tải. Chúng ta có thể tận dụng một tính năng khác của Debugger gọi là **breakpoint** (điểm dừng). Đây là các điểm trong mã nguồn dùng để buộc trình duyệt tạm dừng việc thực thi JavaScript tại thời điểm mong muốn.

<u>Nếu nhấn vào số dòng 110 (dòng chứa đoạn mã trên), bạn sẽ thấy số dòng chuyển sang màu xanh dương. </u><u>Bạn đã đặt thành công một breakpoint tại đây.</u> Hãy thử tải lại trang (refresh), bạn sẽ thấy khung màu đỏ dừng lại trên màn hình thay vì biến mất, bên trong có chứa một đoạn flag.
![[Pasted image 20260812131126.png]]**bài tập**
What is the flag in the red box?
![[Pasted image 20260812132109.png|564]]-vào trang contact, bật debuger lên và đặt break point trwiocs hàm settimeout, tiếp đó tải lại trang
![[Pasted image 20260812132219.png|506]]
# 6- Devtool Network

Thẻ **Network** trong Developer Tools <u>được dùng để theo dõi mọi yêu cầu (request) bên ngoài mà trang web thực hiện</u>. Nếu <u>mở thẻ Network và tải lại trang, bạn sẽ thấy toàn bộ các tệp mà trang yêu cầu</u>. Hãy thử thao tác này trên trang Contact; bạn có thể nhấn vào biểu tượng thùng rác để xóa danh sách nếu quá nhiều mục.
![[Pasted image 20260812132507.png]]
Khi thẻ Network đang mở, hãy điền biểu mẫu liên hệ và nhấn nút Send Message. Bạn sẽ thấy một sự kiện xuất hiện trong thẻ Network; đây là biểu mẫu đang được gửi ngầm qua **AJAX**. <u>AJAX là phương pháp gửi và nhận dữ liệu mạng ngầm trong ứng dụng web mà không làm gián đoạn trang hiện tại</u>.
![[Pasted image 20260812132544.png]]
Khi kiểm tra yêu cầu, bạn có thể xem request headers, chi tiết cookie và phản hồi HTML hỗ trợ cho việc thu thập thông tin cũng như khai thác. Hãy kiểm tra mục mới xuất hiện trong thẻ Network do biểu mẫu liên hệ tạo ra, sau đó xem trang mà dữ liệu được gửi tới để lấy flag.
**bài tập**
What is the flag under the Response tab on the contact-msg network request?
![[Pasted image 20260812133246.png]]*=>* nhần vô sendmessage thì ta thấy gói tin gửi về --> ta thấy trong response flag

# 7-devtool storage
Thẻ Storage trong developer tools <u>cho phép chúng ta xem và quản lý dữ liệu mà trang web lưu trữ trên trình duyệt</u>. <u>Dữ liệu này được lưu ở phía client và có thể chứa thông tin nhạy cảm hoặc hữu ích trong quá trình kiểm thử thủ công</u> (manual pentest). Việc kiểm tra bộ nhớ trình duyệt giúp chúng ta hiểu cách ứng dụng xử lý xác thực, dữ liệu phiên (session data), tùy chọn người dùng và các giá trị lưu trữ khác.
*=>liên hệ với kiến thức JS* [[Browser API#5. localStorage ⭐]]
Trên trang web, hãy tạo một tài khoản mới bằng cách truy cập `[http://10.49.159.172/customers/signup](http://10.49.159.172/customers/signup)`. Sau khi đăng ký, mở thẻ Storage trong developer tools để quan sát giao diện.
![[Pasted image 20260812133854.png]]
Thẻ Storage có các mục quan trọng sau:
* *Local Storage*: Lưu trữ dữ liệu cố định trên trình duyệt, <u>dữ liệu vẫn còn ngay cả khi đã đóng trình duyệt.</u>
* *Session Storage*: <u>Lưu trữ dữ liệu tạm thời </u>cho duy nhất thẻ hoặc phiên làm việc hiện tại của trình duyệt.
* *Cookies*: Các mẩu dữ liệu nhỏ do server gửi và lưu trên trình duyệt, thường dùng để quản lý phiên và xác thực.
* *Cache Storage*: <u>Lưu trữ các tài nguyên đệm</u> như hình ảnh, script và phản hồi API <u>để tăng tốc độ tải trang</u>.
![[Pasted image 20260812134019.png|700]]
Trong các tùy chọn lưu trữ, <u>cookie là thành phần quan trọng nhất đối với pentester</u>. Khi chuyển đến mục Cookies, bạn sẽ thấy dữ liệu được trang web lưu ở phía client.

Dữ liệu này thường bao gồm định danh phiên (session ID), cài đặt người dùng và đôi khi là các token xác thực. <u>Cookie cũng đi kèm các cờ bảo mật (security flags) quan trọng:</u>

* *Cờ HttpOnly*: Ngăn JavaScript truy cập cookie, giúp chống lại tấn công XSS.
* *Cờ Secure*: Đảm bảo cookie chỉ được truyền qua kết nối HTTPS.
* Thuộc tính *SameSite*: Hỗ trợ giảm thiểu rủi ro từ các cuộc tấn công CSRF.
![[Pasted image 20260812134936.png]]
Việc xem xét kỹ các thuộc tính cookie sẽ tiết lộ cách ứng dụng quản lý phiên và liệu ứng dụng có tuân thủ các quy chuẩn bảo mật hay không.
**bài tập**
What is the value of the HttpOnly flag after logging in?

# 8- Kết luận
In this room, we focused on manually assessing a web application using only the browser, no automated tools, no scanners, just observation and logical thinking. From **walking through an application** and **exploring the website** to analysing the **Page Source**, we learned how much information is openly exposed to anyone willing to look closely. Small details such as comments, hidden links, and misconfigurations can often reveal valuable insights. 

By diving into the **developer tools**, including the **Inspector**, **Debugger**, **Network**, and **Storage** tabs, we explored how modern web applications function behind the scenes. We saw how content can be manipulated client-side, how network requests expose application behaviour, and how **cookies** and local storage handle session data. Understanding these components helps build a strong foundation in identifying client-side weaknesses. 

Manual testing sharpens your pentesting mindset. Before relying on automated tools, learning to observe, question, and analyse what’s happening in the browser will make you far more effective. Tools are powerful, but a trained eye and a curious mindset are even more powerful.