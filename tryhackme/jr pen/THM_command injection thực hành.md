# Báo cáo phân tích lỗ hổng ACKme DiagnoseIT

Mục tiêu của mình: ứng dụng `index.php` của trang **ACKme DiagnoseIT** trên TryHackMe.
Kết quả: mình tìm được một lỗ hổng **OS Command Injection (CWE-78)** — và mình đã khai thác thành công.
## 1. Chuyện là thế này

Ứng dụng có một cái ô nhập "địa chỉ IP", bảo là để test xem một thiết bị có online hay không (kiểu như ping thử ấy). Nhưng vấn đề nằm ở chỗ: cái giá trị mình gõ vào bị ứng dụng **nối thẳng vào câu lệnh `ping`** của hệ thống, mà chẳng qua một lớp lọc hay kiểm tra nào cả.

Nói nôm na, code phía server đại khái thế này:
```php
system("ping -c 4 " . $_GET['address']);
```

Nghĩa là: mình gõ gì, nó ghép vào lệnh ping y như vậy. Mà một khi input của người dùng đã chui vào shell mà không được "tẩy trùng", thì chuyện xấu là khó tránh khỏi.
![[Pasted image 20260828135341.png]]
## 2. Bằng chứng mình đã khai thác

Mình gửi payload kiểu "trá hình":

```
address=127.0.0.1;id
```

Kết quả server trả về:

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Đấy — mình chỉ cần chêm thêm dấu `;` và lệnh `id` là server **chạy lệnh đó thật**, lộ ra mình đang có quyền của user `www-data`. Khai thác **thành công**, và nó là **Remote Code Execution (RCE)** luôn.

---

## 3. Trình tự mình đã làm thế nào

| Bước | Mình làm gì | Kết quả |
|------|-------------|---------|
| 1 | Đọc source HTML trước (`index.php?address=ping`) | Nhận ra có tham số `address`, được dùng cho lệnh `ping` |
| 2 | Test giá trị hợp lệ (`address=127.0.0.1`) | Output trả về ping bình thường — chứng tỏ input được nối thẳng vào shell |
| 3 | Chèn lệnh ẩn (`address=127.0.0.1%3Bid`) | Dấu `;` cắt lệnh ping, rồi lệnh `id` được chạy ngay sau đó |

## 4. Có bao nhiêu "đường tắt" để chèn lệnh?

Không chỉ mỗi dấu `;` đâu. Tuỳ vào cách code, mình còn có thể thử:

- `;` (dấu chấm phẩy) — *cái này mình đã chứng minh được*
- `&&` hay `||` (toán tử logic trong shell)
- `|` (ống dẫn dữ liệu / pipe)
- Xuống dòng, dấu backtick `` ` ``, hay `$(...)`

---

## 5. Hậu quả nếu bị lợi dụng

- Kẻ tấn công **chạy được lệnh tuỳ ý** trên server (ở mức `www-data`)
- Đọc được dữ liệu nhạy cảm: `/etc/passwd`, source code, file cấu hình...
- Có thể **leo thang quyền** nếu kết hợp với lỗ hổng khác
- Với bài lab này thì mục tiêu thực tế là rà tìm để lấy được `flag`

---

## 6. Mình đã suy nghĩ ra sao để tìm ra lỗ hổng

### Bước 1 — Đọc trước, đừng vội tấn công

Mình bắt đầu bằng việc gửi một request hết sức vô hại `?address=ping` để xem trang hiển thị gì. Lúc này mình để ý:
- Có một cái form với ô nhập tên `address`, gửi bằng method `GET`
- Mô tả bảo "nhập IP để test xem thiết bị còn online không", kèm ví dụ `127.0.0.1`
- Tham số này được `index.php` tiếp nhận

Từ đây mình thầm đặt câu hỏi: *cái ô nhập này chắc chắn sẽ được đưa vào lệnh `ping` ở phía server.*

### Bước 2 — Điểm mấu chốt: dữ liệu có "đi qua shell" hay không

Mình tự lý luận: một chức năng "ping một địa chỉ" thì gần như bao giờ cũng được code bằng kiểu `system("ping ..." . $input)`. Mà khi input của người dùng không được lọc mà lại được ném thẳng vào lệnh hệ thống, thì nguy cơ **Command Injection** là rất cao. Vì bản chất chức năng này bắt buộc phải gọi lệnh/shell, nên nó chính là "đất diễn" màu mỡ cho lỗ hổng. ← Đây là ý chính giúp mình định hướng.

### Bước 3 — So sánh với hành vi bình thường

Mình thử gửi một giá trị hợp lệ `?address=127.0.0.1`:
- Trang trả về đúng `PING 127.0.0.1 ...`, có `icmp_seq`, có `ping statistics`
- Có dòng *"Here is your command: 127.0.0.1"* — nghĩa là ứng dụng **in y lại chuỗi mình nhập** rồi mới nối vào lệnh
- Mình để ý: nhập IP gì sai kiểu cũng vẫn chạy, chẳng thấy có thông báo "IP không hợp lệ"

→ Điều này củng cố giả thuyết: input bị nối thẳng vào lệnh, **không có bước sanitize nào**.

### Bước 4 — Chọn payload đơn giản nhất để chứng minh

Mình chọn dấu `;` (chấm phẩy). Vì sao ư? Vì nó **kết thúc lệnh `ping`** rồi mở ra để chạy một lệnh khác ngay sau đó — vừa gọn nhẹ, vừa không cần phải lo chuyện escape phức tạp.

Payload đầy đủ: `127.0.0.1;id`

Mình encode dấu `;` thành `%3B` trong URL để không làm hỏng cú pháp của request.

### Bước 5 — Nhìn kết quả và chốt hạ

Kết quả trả về hiện ra **cả hai phần**:

```
--- 127.0.0.1 ping statistics ---     ← lệnh ping đã chạy xong
uid=33(www-data) gid=33(www-data)     ← lệnh id CŨNG chạy xong luôn
```

Lệnh `id` đã thật sự chạy trên server, cho mình biết đang có quyền của `www-data`. Tới đây là **khẳng định chắc chắn: đây là lỗ hổng OS Command Injection (RCE)**.

### Tóm lại, mạch suy luận của mình chỉ gói gọn thế này:

> Đầu tiên nhận diện được **input được nối thẳng vào lệnh hệ thống** (nhờ đọc mô tả chức năng + thấy trang in lại câu lệnh). Rồi kiểm tra xem **có bộ lọc không** (thấy IP bừa vẫn chạy → không có). Sau đó dùng **dấu phân cách shell `;`** để thử chạy thêm lệnh khác. Cuối cùng xác nhận bằng output của lệnh `id`.

---

## 7. Nếu gặp ở thực tế, nên sửa thế nào

1. **Tránh dùng** `system()`/`exec()` với dữ liệu người dùng
2. Dùng **whitelist**: chỉ chấp nhận IP hợp lệ, ví dụ `filter_var($addr, FILTER_VALIDATE_IP)`
3. **Không** qua shell — dùng thư viện ping riêng (như `ICMP PING` trong code)
4. Nếu bắt buộc phải chạy shell, nhớ escape bằng `escapeshellarg()`

---

> **Lưu ý nhỏ:** Mọi khai thác trên chỉ nằm trong khuôn khổ **phòng lab TryHackMe hợp pháp** để phục vụ học tập thôi nhé. Đừng mang áp dụng lên hệ thống của người khác khi chưa được cho phép.