# 1-
Xác thực (Authentication) là quá trình ứng dụng web kiểm tra danh tính của người dùng đang gửi request. Khâu này thường diễn ra tại máy chủ ứng dụng: server so khớp thông tin đăng nhập do client gửi lên với dữ liệu lưu trong database. Khi thông tin trùng khớp, server sẽ nhả về một session token để kẹp vào mọi request tiếp theo cho đến khi hết hạn phiên, và web dùng token đó để phân quyền xem request được phép làm những gì.

Vượt mặt xác thực (Authentication bypass) là bất kỳ đòn tấn công nào giúp kẻ xấu truy cập vào tính năng của một tài khoản mà không cần cung cấp đúng thông tin đăng nhập của tài khoản đó. Đòn bypass không nhất thiết lúc nào cũng phải mò password hay chôm session token. Rất nhiều vụ thành công là nhờ khai thác những giả định chủ quan của lập trình viên về luồng xác thực, hoặc sửa đổi dữ liệu mà server tin tưởng mù quáng không qua kiểm chứng.

**Mục tiêu bài học**
Học xong phần này, mày sẽ nắm được cách:

* **Dò tìm (enumerate) username** hợp lệ bằng công cụ `ffuf` dựa trên sự khác biệt trong phản hồi của form đăng ký.
* **Brute-force form đăng nhập** bằng danh sách username tùy chỉnh kết hợp với từ điển mật khẩu (wordlist).
* Phát hiện và **khai thác lỗi Parameter Pollution** (ô nhiễm tham số) trong luồng đặt lại mật khẩu bằng `curl`.
* **Chỉnh sửa cookie dạng chữ rõ (plain text), mã băm (hash) và mã hóa Base64** để thao túng trạng thái xác thực mà server nhìn thấy.

# 2-Type of Authentication bypass
Có rất nhiều loại lỗ hổng dẫn đến vượt mặt xác thực (Authentication Bypass). Phổ biến nhất trong thực tế là bộ 4 kỹ thuật sau:
![700](https://cdn-images.tryhackme.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/645b19f5d5848d004ab9c9e2-1776682954081.svg)
Để vượt mặt màn đăng nhập (Authentication Bypass), dân tình hay dùng 4 bài tủ này:

* Dò username: Thử ném đại mấy cái tên vào form đăng ký hoặc quên mật khẩu. Nếu web báo "tài khoản đã tồn tại", tức là mày vừa bắt trúng một tài khoản có thật trên hệ thống.
* Vét cạn mật khẩu: Lấy đống username xịn vừa tìm được, ghép với cuốn từ điển các mật khẩu hay dùng rồi cho tool bắn liên thanh vào form login xem dính con nào không.
* Lỗi logic: Thường nằm ở tính năng "Quên mật khẩu". Thằng dev ghép nối các bước xử lý ngáo ngơ khiến link đặt lại mật khẩu của người ta bị gửi nhầm thẳng về hòm thư của mày.
* Sửa cookie: Server nhả về cái cookie dởm (dạng chữ đọc được, hash đơn giản hoặc Base64 dễ dịch ngược). Mày chỉ việc sửa số ID hoặc sửa quyền hạn trong cookie ở máy mày rồi gửi lên là server tưởng mày là Admin ngay.

Hậu quả khi đục được:

* Đọc trộm dữ liệu: Cướp được nick thường thì xem trộm thông tin cá nhân của người ta. Cướp được nick Admin thì nắm trọn quyền sửa xóa dữ liệu, lục tung cả database.
* Làm bàn đạp đấm sâu hơn: Cướp nick hỗ trợ thì đọc sạch vé yêu cầu của khách. Cướp nick Admin thì nhiều khi còn lợi dụng tính năng upload file để thả mã độc, chiếm quyền điều khiển luôn cả con máy chủ.
* Thử vận may ở web khác: Dân tình hay lười, dùng chung một mật khẩu cho cả chục trang web. Mày bới được pass ở trang này thì vác đi đăng nhập dạo ở các trang khác rất dễ ăn trúng tiếp.

# 3- Username emuneration

Dò tìm username (Username enumeration) thực chất là bài đi bới xem trang web đang có những tài khoản nào tồn tại. Thu thập được danh sách tài khoản xịn rồi thì mày mới vác đi dò mật khẩu (brute-force) tiếp được.

Lỗ hổng này hay lòi ra ở mấy chỗ:

* Trang đăng ký: Báo "tên này đã có người dùng".
* Trang đăng nhập: Báo "tài khoản không tồn tại" (khác với câu báo "sai mật khẩu").
* Trang quên mật khẩu: Báo "email này chưa được đăng ký".

Dấu hiệu nhận biết:

* Lộ liễu nhất là câu báo lỗi khác nhau: Đăng ký tên chưa ai dùng thì báo thành công, nhập tên trùng (như admin) thì văng lỗi "An account with this username already exists".
* Tinh vi hơn: Câu chữ giống nhau nhưng độ dài gói tin, mã phản hồi (status code), hoặc thời gian phản hồi (response time) giữa 2 trường hợp có sự chênh lệch. Tool tự động dựa vào mấy điểm khác biệt này để nhận diện.

Dùng ffuf để quét tự động:
ffuf là công cụ bắn request tự động siêu nhanh. Nó sẽ lấy từng cái tên trong từ điển ném vào vị trí đánh dấu FUZZ để gửi lên server.
![How FFuF Determines valid username|700](https://cdn-images.tryhackme.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/645b19f5d5848d004ab9c9e2-1776682977409.svg)
Cú pháp lệnh chạy:
```
ffuf -w /usr/share/wordlists/SecLists/Usernames/Names/names.txt -X POST -d "username=FUZZ&email=x&password=x&cpassword=x" -H "Content-Type: application/x-www-form-urlencoded" -u http://MACHINE_IP/customers/signup -mr "username already exists"
```
![[Pasted image 20260827234836.png]]
Ý nghĩa từng cờ:

* -w: Trỏ tới file từ điển chứa danh sách tên cần đem đi thử.
* -X POST: Gửi request bằng phương thức POST.
* -d: Dữ liệu form gửi lên. Chỗ nào cắm cờ FUZZ thì ffuf sẽ thay tên vào đó, mấy ô email hay password thì điền bừa chữ x là được.
* -H: Khai báo định dạng gửi lên là form web chuẩn.
* -u: Link trang đăng ký mục tiêu (nhớ thay MACHINE_IP thành IP máy mày).
* -mr: Bộ lọc. Chỉ in ra kết quả nào mà server trả lời có cụm từ "username already exists" (chứng tỏ username đó đã có người xài thật).

Chạy xong nó sẽ ra một danh sách các tài khoản có thật. Mày tạo một file tên là valid_usernames.txt, paste toàn bộ đống username đó vào, mỗi dòng 1 tên (nhớ xóa hết mấy thông số thừa thãi) để lát nữa mang sang bài sau đi vét cạn mật khẩu.

> [!NOTE] Title
>- fuff là công cụ bắn request
> -nó giống với burp intruder nhưng nhanh hơn

**thực hành**
![[Pasted image 20260828001736.png|621]]![[Pasted image 20260828000846.png]]
# 4-brute force login form
Tấn công brute-force vào form đăng nhập là trò nã liên tục các cặp tài khoản và mật khẩu vào cổng login cho tới khi trúng thì thôi. Mày đưa cho tool 1 danh sách username và 1 danh sách password, nó sẽ tự động thử mọi tổ hợp cho đến khi ra kết quả hoặc vét sạch từ điển.

Brute-force chỉ ngon ăn khi danh sách username đủ gọn. 5 username ghép với 100 pass thì chỉ tốn 500 request, bắn vài giây là xong. Nhưng nếu ném 10.000 username vào thì thành 1 triệu request, vừa chậm vừa dễ bị hệ thống chặn IP hoặc khóa tài khoản. Bước dò username ở bài trước đã mớm sẵn cho mày một danh sách ngắn nhưng chất lượng để mang vào bài này quất.

**Dấu hiệu nhận biết login thành công:**
Mọi tool dò pass đều cần một tín hiệu để biết request nào ăn tiền. Ở trang web mục tiêu, nếu mày nhập sai nó sẽ trả về mã 200 (và tải lại trang login), còn nếu nhập đúng nó sẽ trả về mã 302 để chuyển hướng mày vào trang Dashboard. Sự thay đổi mã trạng thái từ 200 sang 302 chính là tín hiệu để bắt bài. Web khác thì có thể nhận biết qua độ dài gói tin, cookie mới được cấp, hoặc nội dung trả về.

> [!NOTE] Title
> Dùng error based để dựa vào đó dùng fuff


**Chạy đòn đánh bằng ffuf:**
Thằng ffuf cho phép mày cắm nhiều từ điển cùng lúc bằng cách tự đặt tên mốc đánh dấu thay vì dùng mỗi chữ FUZZ mặc định. Nhờ đó mày đổi được cả username lẫn password độc lập trong cùng một request.

Tại thư mục chứa file valid_usernames.txt, mày chạy lệnh này:
`ffuf -w valid_usernames.txt:W1,/usr/share/wordlists/SecLists/Passwords/Common-Credentials/10-million-password-list-top-100.txt:W2 -X POST -d "username=W1&password=W2" -H "Content-Type: application/x-www-form-urlencoded" -u [http://10.49.190.50/customers/login](http://10.49.190.50/customers/login) -fc 200`
![[Pasted image 20260828001947.png]]
Giải mã các cờ trong lệnh:
* -w: <u>Nạp 2 từ điển cùng lúc, ngăn cách bằng dấu phẩy. </u>Cụm W1 đại diện cho file username vừa lọc, cụm W2 đại diện cho top 100 password phổ biến.
* -d: Gán W1 vào ô username và W2 vào ô password trong gói tin POST.
* -fc 200: Bộ lọc vứt bỏ toàn bộ những phản hồi trả về mã 200 (tức là đăng nhập xịt). Màn hình sẽ chỉ hiện duy nhất kết quả nào trả về mã khác 200 (mã 302 login thành công).

**Thực hành**
![[Screenshot From 2026-08-28 00-42-05.png]]
-do máy mình ko có wordlít nên mình cho con AI nó chạy bash luôn cho lẹ, logic cũng vậy thôi ko cso gì phức tạp cả

# 5-Logic flaws
Lỗi logic (Logic flaw) là dạng lỗ hổng xảy ra khi mày gửi dữ liệu hoàn toàn hợp lệ, nhưng lại khiến luồng xử lý của hệ thống chạy theo một hướng oái oăm mà dev đéo lường trước được. Nó không giống SQLi hay tràn bộ đệm dùng mấy ký tự dị hợm, lỗi logic dùng dữ liệu sạch 100% nhưng khai thác sự đá nhau giữa 2 luật xử lý ngớ ngẩn. <u>Mấy con tool quét tự động thường mù tịt với lỗi này, chỉ có soi code hoặc mò bằng tay mới ra</u>.

**Ví dụ 1: Đá nhau giữa chữ hoa và chữ thường**
Thằng định tuyến (router) của web thì dễ tính, coi `/admin` và `/adMin` là một. Nhưng đoạn code kiểm tra quyền admin lại dùng so sánh nghiêm ngặt:
`if( url.substr(0,6) === '/admin')`
Mày gửi request vào `/adMin`, đoạn check quyền thấy khác chữ nên cho qua thẳng, trong khi router bên dưới vẫn bốc mày ném vào trang Admin. Thế là lách qua ngon ơ.

> [!NOTE] Title
> code authorise, và thằng router nó ko đồng bộ, đáng lẽ cái case /adMIN cũng phải được backend ném vào code authorise nhưng ở đây thì ko

**Ví dụ 2:** Lỗi ô nhiễm tham số **(Parameter Pollution)** khi quên mật khẩu
![[Pasted image 20260828004850.png]]
Trang reset pass của bài này chia làm 2 bước và nhận dữ liệu ở 2 nơi khác nhau:

* Bước check tài khoản: Nó đọc email từ thanh URL (Query string: `?email=...`).
* Bước gửi thư: Backend PHP lại dùng biến trời đánh `$_REQUEST` để lấy địa chỉ nhận link.

Cái tai hại của `$_REQUEST` trong PHP là nó gom cả URL (GET) và Body (POST) lại làm một. Nếu trùng tên biến `email`, giá trị trong Body sẽ đè bẹp giá trị trên URL.

> [!NOTE] Title
> ok gửi thư resettoken về $_Request = tham số(url+body) . Giá trị body được ưu tiên hơn 

**Khai thác thực tế cướp nick Robert:**
Trang web cấp cho mỗi tài khoản một hòm thư nội bộ dạng `{username}@customer.acmeitsupport.thm`, thư gửi về đây sẽ hiện thẳng thành một cái vé hỗ trợ (ticket) trong tài khoản.

Các bước húp cờ:

1. Mày đăng ký một tài khoản bất kỳ (ví dụ tên mày là `hacker123`). Hòm thư của mày sẽ là `hacker123@customer.acmeitsupport.thm`.
2. Mở Terminal trên AttackBox, bắn lệnh curl này để bẻ lái link reset pass của thằng Robert về hòm thư của mày:

curl '[http://10.49.190.50/customers/reset?email=robert@acmeitsupport.thm](http://10.49.190.50/customers/reset?email=robert@acmeitsupport.thm)' -H 'Content-Type: application/x-www-form-urlencoded' -d 'username=robert&email=hacker123@customer.acmeitsupport.thm'

> [!NOTE] Title
> để ý có 2 trường email là cái ở url và cái ở Body trong request gửi lên
> -backend xử lý lỗi , khiến nó sẽ nhận cái email trong phần body chứ ko phải trong url
> -email reset gửi về email của hacker

(Nhớ thay `hacker123` bằng cái username mày vừa tạo).

3. Đăng nhập vào nick của mày trên web, chui vào mục Support Tickets sẽ thấy một cái ticket mới toanh chứa link reset mật khẩu của Robert.
4. Bấm vào link đó để đổi pass hoặc login thẳng vào nick Robert, lục mục tickets của nó là lòi ra cái cờ (flag).![[Screenshot From 2026-08-28 01-19-38.png]]
> [!NOTE]
> => hiểu cái attack chain là được, thực ra viết scipt mình viết hay sai nên cho AI viết, nhưng mình đọc hiểu thì mình vẫn hiểu script

# 6-cookie manipulation
Vì giao thức HTTP mắc bệnh "gửi xong là quên" (phi trạng thái), server phải nhả về Cookie để trình duyệt lưu lại, lần sau gửi kèm lên để server nhận diện ai với ai. Nếu server không khóa hoặc ký điện tử (signature) cái cookie này lại, mày ở phía client có thể thoải mái sửa giá trị trong cookie để lừa server cấp quyền Admin.

Có 3 kiểu cookie cùi bắp mà dân pentest hay gặp và đục nước béo cò:

1.**Cookie dạng chữ rõ (Plain Text)**
Server lưu thẳng tuột quyền hạn vào cookie mà không thèm che đậy gì, kiểu `logged_in=true; admin=false`.

* Cách đục: Mở tab Storage trên trình duyệt hoặc dùng curl sửa `admin=false` thành `admin=true` rồi gửi lại request. Server đọc thấy `admin=true` là tưởng mày là Admin thật và nhả cờ ra ngay:
`curl -H "Cookie: logged_in=true; admin=true" [http://10.49.190.50/cookie-test](http://10.49.190.50/cookie-test)`

**2 Cookie dạng băm (Hash)**
Nhiều ông dev nghĩ ném giá trị qua hàm băm một chiều (như MD5, SHA-1, SHA-256) là hacker chịu chết không sửa được.

* Sai lầm: Băm không phải là ký điện tử. Cùng một chuỗi đầu vào thì luôn luôn ra đúng một chuỗi băm cố định. Nếu server băm số ID người dùng (ví dụ ID `1`), mày có thể vác chuỗi băm đó lên mấy trang như CrackStation tra ngược lại xem giá trị gốc là gì, hoặc tự tạo mã băm cho tài khoản Admin rồi nhét vào cookie.

3.**Cookie dạng mã hóa định dạng (Base64 / Base32)**
Nhiều dev nhét cả cục dữ liệu JSON vào cookie rồi mã hóa Base64 cho gọn, nhìn qua cứ tưởng là bảo mật.

* Sự thật: Base64 chỉ là đổi bảng mã cho đúng chuẩn truyền tin chứ đéo có tí tính năng bảo mật nào.
* Cách đục: Mày bốc cái chuỗi Base64 `eyJpZCI6MSwiYWRtaW4iOmZhbHNlfQ==` đem giải mã (decode) ra sẽ thấy cục JSON gốc `{"id":1,"admin":false}`. Mày chỉ việc sửa tay thành `{"id":1,"admin":true}`, mã hóa Base64 ngược lại rồi nhét đè vào cookie gửi lên là xong phim.

Chốt lại bài này: cứ thấy cookie nào đọc được, giải mã được hoặc đoán được quy luật băm mà không có chữ ký bảo vệ từ server thì cứ sửa giá trị để leo quyền.

> [!NOTE] Title
>Hmmm, thực ra các cơ chế trên nhắm đến 1 điều là bảo vệ tính toàn vẹn cookie, nhưng mà toàn mấy cơ chế cổ lỗ sĩ, và thường dễ đoán có quy luật, đoán được quy luật là lòi ra cách sửa cookie sao cho có thể giả thành cookie của các người dùng khác.
>-cái này chắc mang tính lý thuyết thôi, chứ thực tế chẳng có ai dùng mấy cách dễ đoán này cả , haizzz :)

