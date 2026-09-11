# 1 - tổng quan
Phần này bắt đầu chuyển sang dạy về Command Injection (Chèn lệnh hệ điều hành) – một trong những lỗi nguy hiểm nhất đưa thẳng tới RCE (thực thi mã từ xa).

Bản chất của nó rất đơn giản:
* Web app thỉnh thoảng cần gọi các lệnh của hệ điều hành bên dưới để chạy tính năng (ví dụ lệnh ping kiểm tra mạng, tạo file nén...).
* Dev lấy dữ liệu do người dùng nhập vào rồi ghép thẳng vào câu lệnh hệ thống mà không thèm lọc hay kiểm tra.
* Mày chỉ việc chèn thêm các ký tự nối lệnh (kiểu ;, |, &&) để ép máy chủ chạy thêm bất kỳ lệnh Linux/Windows nào mày muốn (như `whoami`, `cat /etc/passwd`, tạo reverse shell).
* Lệnh mày chèn vào sẽ chạy với đúng quyền hạn của tài khoản đang chạy web server (ví dụ user joe hoặc www-data).

> [!NOTE] note
> các trường hợp dễ bị command injection


**Nội dung mày sẽ học và thực hành trong phòng này gồm:**
* **Cơ chế gây lỗi** khi dùng các hàm gọi hệ thống không an toàn trong code (PHP, Python).
* **Phân biệt giữa 2 dạng:** *Verbose* (kết quả lệnh in thẳng ra giao diện web) và *Blind* (chạy ngầm không in kết quả, phải mò qua độ trễ thời gian như lệnh sleep/ping).
* **Kỹ thuật dùng các toán tử nối lệnh và payload** thực chiến trên cả Linux lẫn Windows.
* Cách phòng thủ chuẩn chỉ (lọc dữ liệu đầu vào và dùng API an toàn).
* Đục thẳng vào một con máy ảo có sẵn để lôi cờ (flag) ra nộp.

# 2-Discovering command injection
Bản chất lỗi nằm ở chỗ: <u>hầu hết ngôn ngữ (PHP, Python, Node.js) đều có sẵn mấy hàm chạy lệnh hệ điều hành như exec(), system(), subprocess.</u> Bản thân mấy hàm này không có tội, tội là ở thằng dev lười, lấy thẳng dữ liệu người dùng nhập vào rồi nối chuỗi vào câu lệnh mà không thèm lọc ký tự đặc biệt.

Ví dụ 1: Code PHP dính chưởng
```php
<?php
$songs = "/var/www/html/songs";                                    // 1

if (isset($_GET["title"])) {
    $title = $_GET["title"];                                       // 2

    $command = "grep $title /var/www/html/songtitle.txt";          // 3

    $search = exec($command);                                      // 4
    if ($search == "") {
        $return = "<p>The requested song</p><p> $title does </p><b>not</b><p> exist!</p>";
    } else {
        $return = "<p>The requested song</p><p> $title does </p><b>exist!</b>";
    }

    echo $return;
}
?>
```
* Dev viết tính năng tìm tên bài hát bằng lệnh grep của Linux:
`$command = "grep $title /var/www/html/songtitle.txt";` rồi gọi `exec($command);`
* Người dùng bình thường nhập tên bài hát thì không sao. Nhưng nếu mày nhập vào URL:
`?title=; cat /etc/passwd`
* Lúc này câu lệnh trên server bị biến thành:
`grep ; cat /etc/passwd /var/www/html/songtitle.txt`
* Dấu chấm phẩy `;` trong Linux là ký tự ngắt lệnh. Server sẽ chạy lệnh grep rỗng (lướt qua), sau đó chạy luôn lệnh thứ hai là `cat /etc/passwd` để đọc sạch file tài khoản hệ thống ném về cho mày.

Ví dụ 2: Code Python Flask cẩu thả
```python
import subprocess
from flask import Flask                                            # 1
app = Flask(__name__)

def execute_command(shell):                                        # 2
    return subprocess.Popen(shell, shell=True, stdout=subprocess.PIPE).stdout.read()

@app.route('/<shell>')                                             # 3
def command_server(shell):
    return execute_command(shell)
```
* App nhận bất kỳ chữ gì trên thanh URL rồi ném thẳng vào `subprocess.Popen(shell=True)`.
* Mày truy cập `[http://flaskapp.thm/whoami](http://flaskapp.thm/whoami)`, server bốc luôn chữ `whoami` chạy trên hệ điều hành rồi in kết quả ra web. Cái này biến trang web thành cái terminal online cho mày điều khiển máy chủ luôn.

Quy luật chung: <u>Cứ thấy app lấy input người dùng ghép chuỗi vào mấy hàm gọi hệ thống mà không có bộ lọc chặn ký tự đặc biệt thì nhét ngay mấy dấu nối lệnh </u>(`;`, `|`, `&&`) vào để chèn lệnh phá hoại.

> [!NOTE] Title
> vấn đề là, làm sao để mày biết có command injection khi blackbox

# 3-exploiting command injection
Nhận diện Command Injection thì chia làm 2 trường hợp rõ ràng:

1. Dạng lộ liễu (**Verbose Command Injection)**
Server chạy lệnh xong nhả thẳng kết quả lên màn hình web cho mày xem. Nhập `; whoami` vào ô tìm kiếm mà thấy nó in ra tên user (như `www-data` hay `root`) ngay trên trang thì coi như ăn tiền luôn, dễ nhất quả đất.

2. Dạng mù **(Blind Command Injection)**
Lệnh vẫn chạy ngầm trên server nhưng web không in kết quả ra ngoài, giao diện trước và sau khi chèn lệnh nhìn y hệt nhau. Lúc này phải dùng 2 mẹo để bắt bài:
* Thử bằng độ trễ thời gian (Time-based):
Bắt server phải câu giờ bằng lệnh ngủ. Ví dụ chèn `; sleep 10` (hoặc `; ping -c 10 127.0.0.1` trên Linux / `& timeout 10` trên Windows). Nếu thấy trang web tự nhiên quay mòng mòng đơ đúng 10 giây mới tải xong thì chắc chắn lệnh đã được chạy ngầm.
* Ép kết quả ghi ra file (Output Redirection):
Chèn dấu `>` để ném kết quả vào thư mục web public, ví dụ `; whoami > /var/www/html/output.txt`. Xong xuôi mở trình duyệt vào thẳng link `[http://target.thm/output.txt](http://target.thm/output.txt)` để đọc nội dung file.

**Các lệnh bỏ túi hay dùng nhất:**
Trên Linux:
* whoami: Xem đang chạy dưới quyền tài khoản nào.
* ls: Xem danh sách file, thư mục để bới file cấu hình, mã nguồn, mật khẩu.
* ping / sleep: Bắt server hoãn phản hồi để test lỗi dạng mù.
* nc (Netcat): Bắn reverse shell về máy mày để có terminal thao tác trực tiếp.

Trên Windows:
* whoami: Xem tài khoản hiện tại.
* dir: Liệt kê file và thư mục (tương đương `ls`).
* ping / timeout: Bắt server dừng lại vài giây để test dạng mù (tương đương `sleep`).

> [!NOTE] Title
> Nói về các vấn đề rất basic thôi, ko có gì đặc biệt

# 4-thực hành
[[THM_command injection thực hành]]
# 5- Remediation
Phần này dạy mày cách vá lỗi Command Injection và bản chất vì sao mấy trò chặn nửa mùa rất hay bị đấm cho vỡ mồm.

Có 4 ý chính mày cần nắm:

1.**Chặn ở giao diện (Frontend) chỉ để làm cảnh**
Thằng dev nhét thuộc tính `pattern="[0-9]+"` vào thẻ HTML để bắt người dùng chỉ được gõ số. Gõ chữ hay dấu chấm phẩy là trình duyệt chặn lại không cho bấm gửi.
Nhưng trò này chỉ lừa được người dùng bình thường. Mày vác curl hoặc Burp Suite bắn reques
![[Pasted image 20260828141106.png]]t thẳng vào backend thì cái bộ lọc ở trình duyệt coi như vứt vào sọt rác.
2.**Lọc dữ liệu chuẩn ở Backend (Server-side)**
Muốn an toàn thì server bắt buộc phải tự thẩm định lại dữ liệu. Thay vì tin dữ liệu gửi lên, backend dùng các hàm kiểm tra như `filter_input` trong PHP để ép kiểu. Ô nhập số thì bắt buộc chỉ được là số, lòi ra một ký tự lạ là hủy request ngay.
Nguyên tắc chuẩn là dùng Whitelist (chỉ cho qua những thứ được định nghĩa trước) thay vì Blacklist (ngồi lập danh sách các ký tự bị cấm).
![[Pasted image 20260828141124.png]]
3.**Vì sao Blacklist (chặn từ cấm) dễ bị lách qua**
Nhiều ông dev lười, chỉ viết code xóa mấy ký tự như dấu gạch chéo `/` hay dấu nháy. Lúc này kẻ tấn công sẽ đổi dạng biểu diễn dữ liệu:
* Thay vì gõ thẳng `/etc/passwd`, người ta mã hóa sang dạng Hex thành `\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64`.
* Bộ lọc nhìn thấy chuỗi ký tự lạ tưởng vô hại nên cho qua, nhưng hệ điều hành bên dưới lúc thực thi vẫn hiểu đó là đường dẫn file `/etc/passwd`.

4.**Phòng thủ nhiều lớp (Defense in Depth)**
Đừng bao giờ trông cậy vào một chốt chặn duy nhất:
* <u>Tốt nhất là bỏ hẳn việc gọi lệnh hệ điều hành, thay bằng các hàm thư viện nội bộ của ngôn ngữ</u>.
* Ép chặt kiểu dữ liệu đầu vào ở backend.
* Phân quyền tài khoản chạy web server ở mức thấp nhất (least privilege) để lỡ có bị chèn lệnh thì cũng không đọc trộm được file hệ thống hay leo quyền phá hoại.

> [!NOTE] Tóm lại
> - nếu ghép chuỗi từ input và cho các hàm thực thi commnad thì bắt buộc phải có các biện pháp bảo vệ, như kiểu blacklist hay whitelist,....
> - Nhưng thực tế bây giờ người ta dùng các hàm của thư viện để thực hiện luôn câu lệnh đơn lẻ, thay vì cứ lấy input rồi cho vào hàm thực thi shell như exec()
