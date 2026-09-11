# 1-tổng quan


# 2- session manager là gì?
Trước khi lôi mấy cái lỗ hổng ra mổ xẻ, mày phải hiểu quản lý phiên (session management) là cái quái gì đã. Như tao nói ở bài trước, chẳng ai điên mà đi kẹp tài khoản và mật khẩu vào từng cái request gửi đi cả. Nhưng ngặt nỗi giao thức HTTP nó lại mắc bệnh "não cá vàng" (phi trạng thái - gửi xong là quên). Thế nên người ta mới đẻ ra Session để theo dõi mày suốt quá trình lướt web. Quản lý phiên chính là quy trình cai quản đống session này sao cho an toàn.

**Vòng đời quản lý phiên**
Cách ngon nhất để ngấm là nhìn vào 4 giai đoạn vòng đời của nó:
![[Pasted image 20260825125834.png]]
**Khởi tạo phiên (Session Creation)**
Mày chắc mẩm bước này chỉ xảy ra sau khi mày gõ tài khoản mật khẩu chứ gì? Sai bét. Nhiều trang web nó đã cấy session cho mày ngay từ lúc mày vừa mở web lên rồi, để nó theo dõi hành vi kể cả khi mày chưa thèm đăng nhập. Nhưng bài này tao với mày chỉ tập trung vào session sau khi đã đăng nhập thôi. Một khi mày nhập đúng thông tin, server sẽ ném cho mày một giá trị session để kẹp vào mọi request tiếp theo. Cách mấy cái giá trị này được sinh ra, dùng và lưu trữ chính là mấu chốt để bảo vệ bước này.

**Theo dõi phiên (Session Tracking)**
Có giá trị session rồi, mày cứ đính nó vào từng request mới. Nhờ thế mà web nó nhớ được mày. Cứ mỗi lần mày gửi request, server sẽ lôi cái session đó ra, đối chiếu xem mày là thằng nào và có quyền hạn gì. Nếu khâu theo dõi này mà dev code ẩu, hacker có thể nhảy vào cướp phiên (hijack) hoặc giả danh nạn nhân ngon ơ.

> [!NOTE] Title
> Giá trị của session sẽ được đính vào từng request

**Hết hạn phiên (Session Expiry)**
Vì HTTP là "não cá vàng", lỡ mày đang lướt web xong tắt cái rụp trình duyệt, server nó mù tịt không thể biết mày đã rời đi. Đó là lúc tính năng "hết hạn phiên" lên sàn. Bản thân session phải có tuổi thọ. Hết giờ mà mày vẫn vác cái session cũ rích đấy đi gửi request thì server nó phải từ chối và đá mày về trang login để đăng nhập lại từ đầu.

**Hủy phiên (Session Termination)**
Nhiều lúc mày chủ động bấm "Đăng xuất". Lúc này, web nó phải tự động tiêu hủy cái session của mày đi. Bước này nghe giống hết hạn, nhưng khác ở chỗ: dù tuổi thọ session vẫn còn, nhưng một khi đã ấn đăng xuất là cái session đó phải "chết" ngay lập tức. Lỗi ở khâu này có thể giúp hacker cắm rễ vào tài khoản của nạn nhân mãi mãi.

Chốt lại, trong bài này tao với mày sẽ soi xem từng cái khâu trên có thể bị hổng ở đâu. Muốn bảo mật thì phải bọc lót được toàn bộ vòng đời này. Nhưng trước khi đi vào thực hành, cứ nhai thêm tí lý thuyết ở phần tới đã.

> [!NOTE] Title
> Tổng kết lại , là hiểu được vòng đời 4 quá trình của session, cái này nằm ở phần backend,:
> 	-tạo phiên--> gắn phiên vào mỗi request--> phiên hết hạn--> hủy phiên

# 3-authentication vs authorisation
Để hiểu mấy cái lỗ hổng phổ biến trong quản lý phiên, mày phải soi kỹ 2 khái niệm: Xác thực (Authentication) và Phân quyền (Authorisation). Nghe thì na ná nhau và hay bị nhầm, nhưng mỗi thằng ôm một nhiệm vụ riêng biệt.

Để tao vạch rõ ra cho mày qua mô hình IAAA:
![[Pasted image 20260825131425.png]]
**Định danh (Identification)**
Là quá trình xác minh mày là thằng nào. Bắt đầu bằng việc mày tự xưng danh. Trên web, mày nhập username hoặc email để gào lên với server: "Tao là chủ cái acc này!".

**Xác thực (Authentication)**
Là quá trình check xem mày có nổ không, hay mày đúng là thằng mày vừa xưng. Nếu định danh là đưa ra cái tên, thì xác thực là đưa ra bằng chứng. Ví dụ, mày phải ói ra cái password khớp với username đó. Web check đúng thì cho qua, và đây chính là lúc quá trình "Khởi tạo phiên" (Session Creation) bắt đầu chạy.

**Phân quyền (Authorisation)**
Là kiểm tra xem mày có đủ tuổi để làm cái việc mày vừa yêu cầu không. Ví dụ: thằng user nào cũng xem được bài viết, nhưng chỉ Admin mới được quyền xóa. Trong vòng đời quản lý phiên, khâu "Theo dõi phiên" (Session Tracking) gánh cái nhiệm vụ phân quyền này.

**Truy vết (Accountability)**
Là quá trình ghi log lại toàn bộ những trò mày làm. Hệ thống sẽ bám theo session của mày, lưu lại từng hành động. Cái này sống còn ở chỗ: lỡ có biến, người ta lôi log ra là ghép lại được ngay toàn cảnh sự việc.

**Mối liên hệ giữa IAAA và Quản lý phiên**
Giờ phân biệt được rồi thì ốp ngược lại vào hệ thống nhé:

* Xác thực: Quyết định việc session được đẻ ra như thế nào.
* Phân quyền: Check xem thằng đang cầm session này có quyền táy máy vào tính năng nó vừa gọi hay không.
* Truy vết: Gom log các request kèm theo đúng cái session đó, để lỡ web có toang còn biết lôi đầu thằng nào ra chịu trách nhiệm.

> [!NOTE] Title
> -Đại khái là gì , Hiểu cái IAAA là gì: nó bao gồm định danh, xác thực, phân quyền, truy vết
> -cái giai đoạn xác thực nó thực hiện ở pha quản lý "session tracking"

# 4-cokkie and tokens
Trước khi nhảy vào chọc ngoáy bảo mật, mày phải biết hiện nay người ta đang xài loại session gì đã. Có 2 phái chính là Cookie và Token, mỗi thằng đều có điểm ngon và điểm phế riêng.

### Quản lý phiên bằng Cookie
Cách này được gọi là kiểu chơi đồ cổ (old-school). Khi web muốn bắt đầu theo dõi mày, nó sẽ ném trả một cái header `Set-Cookie` trong gói response. Trình duyệt của mày đọc cái này xong sẽ tự động lưu lại. Nhìn thử con header này xem:

`Set-Cookie: session=12345;`

==Trình duyệt sẽ ghi nhớ một cookie tên là `session` mang giá trị `12345`,== và nó ==chỉ có hiệu lực với đúng cái tên miền (domain) vừa gửi nó tới.== Thêm nữa, người ta có thể gắn thêm một mớ thuộc tính (attributes) vào cái header này để tăng giáp bảo vệ. Cần nhớ mấy thằng cốt lõi sau:

* `Secure`: Ra lệnh cho trình duyệt chỉ được phép gửi cookie này qua đường HTTPS an toàn. Có lỗi chứng chỉ hay dùng HTTP thường là nó chặn không cho gửi.
* `HTTPOnly`: Cấm tiệt mấy đoạn script JavaScript chạy trên trình duyệt (client-side) được phép đọc giá trị cookie. Tránh bị XSS móc lốp.
* `Expire`: Hẹn giờ chết cho cookie. Hết hạn là trình duyệt tự động vứt sọt rác.
* `SameSite`: Nhắc trình duyệt có được phép gửi cookie chéo trang (cross-site) hay không, mục đích là để chống lại mấy đòn tấn công CSRF.
[[Content Discovery]]
Cái chốt hạ cần nhớ của hệ Cookie là: Trình duyệt tự đứng ra lo liệu. Nó check domain, check thuộc tính xong là tự động kẹp cookie vào request gửi đi, không cần mày phải viết thêm dòng code JavaScript nào.

> [!NOTE] Title
> -server gửi response với Header *set-cookie: session = 12345*, cái response này kèm một đống thuộc tính như trên 
> - sau đó trình duyệt sẽ nhớ 1 cookie tên là session có giá trị là 12345, cái biến session này chỉ có tác dụng với chính domain của server

### Quản lý phiên bằng Token
Trò này thì mới mẻ hơn. Thay vì ỷ lại vào trình duyệt, nó dùng code JavaScript ở client để tự xử. Sau khi mày đăng nhập thành công, server sẽ vứt cho mày một cái token giấu trong phần body của response. Code JavaScript trên máy mày sẽ bế cái token này nhét vào `LocalStorage` của trình duyệt.

Khi mày gửi request mới, JavaScript phải tự mò vào kho lôi token ra và đính nó vào header. Cái loại token nhẵn mặt nhất là JSON Web Tokens (JWT), thường được truyền đi qua cái header `Authorization: Bearer`.
Ngặt nỗi, vì không dùng cơ chế quản lý tự động của trình duyệt nên mảng này hơi giống miền viễn tây - mạnh ai nấy làm. Dù có tiêu chuẩn đấy nhưng chả có cơ chế nào ép buộc thiên hạ phải tuân theo 100% cả.


> ![[Pasted image 20260825133556.png]]

**Lên bàn cân so sánh**
**Hệ Cookie:**
* Trình duyệt tự động kẹp vào request.
* Tận dụng được các thuộc tính (như HttpOnly, Secure) để tăng bảo mật.
* Dễ ăn đòn CSRF vì trình duyệt tự động gửi hộ khi bị lừa.
* Bị khóa chết vào một domain, khó xài cho mấy hệ thống web phân tán đa miền.

**Hệ Token:**
* Phải dùng code JavaScript để tự móc token kẹp vào header của từng request.
* Không có lớp bảo vệ tự động, lập trình viên phải tự tìm cách cất giấu token cho kỹ.
* Mặc định chống được CSRF vì nó không tự động thêm vào request, và domain khác cũng không đọc trộm được LocalStorage.
* Chơi cực mượt với các hệ thống phân tán vì nó xài qua JavaScript và bản thân cái token thường đã chứa đủ thông tin để tự xác thực rồi.

> [!NOTE] tự tóm tắt lại
> - với session, thì trình duyệt sẽ gắn header set-cookie: session=.... vào http response+ atribute.Trình duyệt sau đó đọc, và gán sesion vô mọi request tới domain đó
> - với jwt , thì trình duyệt sẽ trả gắn token trong http ressponse và mã js lưu cái token đó vào local storage, và từ đó dùng cái token đó gán vô mọi request

# 5- secure the session life circle
Đến đoạn thực chiến về bảo mật quản lý phiên rồi đây. Bám vào đúng 4 giai đoạn vòng đời, đây là những chỗ hay bị lỗi nhất mà mày cần nắm để đi săn:

> [!NOTE] Title
> 4 giai đoạn vòng đời của session , tồn lại những nguy cơ lỗ hổng nào

**Khởi tạo phiên (Session Creation)**
Đây là giai đoạn lòi ra nhiều lỗ hổng nhất:
* Giá trị session yếu (Weak Session Values): Do dev tự chế cơ chế tạo session ngớ ngẩn (ví dụ lấy luôn Base64 của username làm session). Kẻ tấn công đảo ngược thuật toán là đoán được session của thằng khác để cướp nick.
* Giá trị session bị kiểm soát (Controllable Session Values): Hay gặp ở token JWT. Nếu server lười kiểm tra chữ ký (signature) hoặc tạo chữ ký sơ sài, hacker có thể tự làm giả token với quyền admin.
* Cố định phiên (Session Fixation): Web cấp session cho mày trước cả khi đăng nhập, nhưng sau khi đăng nhập xong thì không thèm đổi (rotate) session mới. Hacker gài sẵn session cũ này cho nạn nhân dùng, đợi nạn nhân login xong là nhảy vào dùng chung.
* Truyền session không an toàn (Insecure Session Transmission): Thường gặp ở mấy cơ chế SSO (Single Sign-On). Lúc chuyển giao thông tin phiên từ server xác thực sang server ứng dụng qua redirect, nếu dính lỗi Open Redirect thì hacker có thể ép chuyển hướng session về server của nó.

> [!NOTE] tự tóm tắt lại
> *weak session* : nằm ở quy tắc cấp session cho user quá dễ đoán quy luật
> *controllable session value* : có khả năng chỉnh sửa jwt từ phía local storage, thường do vấn đề cái signature jwt sơ sài,  dẫn đến ko đảm bảo tính toàn vẹn của JWT(cái này nằm ở) [[JWT là gì]]
> *session fixation*: vấn đề này do dev ẩu thôi, ko gặp trong thực tế
> *session transmissison* : do cấu hình lúc truyền gói tin ko an toàn, hacker có thể chặn bắt, hoặc redirect về máy hacker( cái này khá lý thuyết)

**Theo dõi phiên (Session Tracking)**
Giai đoạn này hay ăn đòn ở 2 mảng:
* Vượt mặt phân quyền (Authorisation Bypass):
* Leo quyền dọc (Vertical): User thường chui vào trang dành riêng cho Admin.
* Leo quyền ngang (Horizontal): Thao tác đúng quyền nhưng sờ vào dữ liệu của người khác (kiểu lỗi IDOR kinh điển).
* Ghi log thiếu sót (Insufficient Logging): Không ghi lại hành động gắn với từng session cụ thể, hoặc chỉ ghi log mấy request bị từ chối mà quên log các request thành công. Đến lúc bị cướp session thì chịu chết không điều tra được.

> [!NOTE] think
> leo quyền dọc và leo quyền ngang , mình cũng ko hiểu rằng nó liên quan gì đến đoạn sesison tracking nữa? uhm , có lẽ nó đánh vào hành động authorise diễn ra ở giai đoạn session tracking chăng?, có lẽ đa số lỗ hổng nằm ở việc config phân quyền cho session tương ứng
> -nhưng mà thực sự t vẫn chưa hiểu cái ghi log thiếu sót là sao. tại sao nó lại ảnh hưởng tới quá trình xác minh session từ phía server, có lẽ là có nhiều sự cố xảy ra, nhiều thao tác trong quá trình dùng, một số sesison sẽ bị hủy, thay bằng sesison mới, một số session lại được thay thế, và một số session sẽ được tạo mới, ...vân vân và mây mây các trường hợp có thể xảy ra, vậy nên nó sẽ cần ghi log hẳn hoi, để biết được giá trị sesion nào vẫn còn hiệu lực ở thời điểm hiện tại


**Hết hạn phiên (Session Expiry)**
Vấn đề duy nhất ở đây là thời gian sống của session quá dài. Session giống như vé xem phim, hết xuất chiếu là phải hủy. App ngân hàng thì phải đặt thời gian hết hạn cực ngắn, còn webmail thời gian dài hơn thì phải bám theo địa điểm/IP (đổi IP bất thường là phải đá văng ra ngay).

**Hủy phiên (Session Termination)**
Lỗi mấu chốt là khi người dùng bấm Đăng xuất nhưng server không chịu hủy phiên ở phía backend. Đặc biệt với JWT (vốn mang thời hạn nằm ngay trong token), nếu server không có danh sách đen (blocklist) để chặn các token đã logout thì hacker vẫn dùng lại được token cũ đó. Ngoài ra, khi đổi mật khẩu cũng bắt buộc phải hủy toàn bộ các session đang mở trên mọi thiết bị.

> [!NOTE] Think
> Có lẽ nó muốn nói đén việc token cũ sẽ xử lý như nào khi người dùng đăng xuất?hmm, có lẽ mình có thể hiểu là server sẽ cho cái token cũ vào blacklist ?ummm, liệu có phải là một cách hợp lý ko?nhưng đại khái mình hiểu ý muốn nói ở đây rồi, là đề cập đến các rủi ro khi không xử lý session hay token khi mà người dùng đăng xuất-là một hành động sẽ gây "sesion termination"

# 6- thực hành
**my chain thought**
bây giờ mình cần phải tự làm , mày mò các thứ, vậy thì mới đúng nghĩa thực hành,ok mình sẽ bắt tay vào làm lab.
Có vẻ ko cần kali linux nên mình sẽ làm bằng máy fedora của mình thôi vậy, đầu tiên cứ kết nối tới machine và xem cái web nó như thế nào đã.Hình như cái lab này bắt mình đăng nhập, xong inspect đẻ xrm trường local storage của trình duyệt nó lueu những gì, sau đó mình có thể chình sue thử xêm có được ko, mà chỉnh suẳ cái gì ??đọc qua thì nó bảo chỉnh sửa role gì gì đó, vậy nó là token hay session, và t tưởng nó chỉ là một cái id thôi chứ, sao lại có cả role ở đây ?? khá là khó hiểu, phải làm thử mới biết được

- đơn giản thôi, vào trang web, nhấn vô mọi thứ, xuất file burpsite, cho AI agent đọc, bảo nó lọc ra các enpoint nhạy cảm, sau đó vào thôi
- cái thứ 2 cần đề cập là cái phân quyền khi đi vào các enpoint đó, nó đơn giản là dùng sesion để xác thực và phân quyền, đồng thời có 1 cái token(JSON) lưu cái role của người dùng, nó sẽ kẹp cái token đó vào mỗi requeest tới enpoint để có thể authorise truy câpj tới enoint.
- Và cái token này ko có cơ chế " bảo vệ tính toàn vẹn", nên tao có thể dễ dàng chỉnh sửa role" lecture, mà khi kẹp vào request đi đến server, server nó vẫn tin tao là lecture thật, thế là tao vào được các enpoint nhạy cảm thôi, so EZ
![[Pasted image 20260825154235.png]]nói chung là chỉ nói thôi thì không hình dung hết nổi đâu, phải thực hành nhiều nhiều vào, khi đó mày sẽ biết đống kiến thức mày học sẽ áp dụng ntnt, và cũng đừng cố nhớ các bước làm gì, cứ phá cứ hỏi tung tóe đi, ra kết quả là được, conf kiến thức sẽ tự chui vào đầu mày thôi [[mind set học đúng]]


> [!NOTE] Title
> -dành việc phân tích dữ liệu cho AI
> -cái chính ở đây là gì, mình hiểu kiến trúc system backend, các mà nó dùng token để authorise,cũng như authentication, và hiểu caí bài lab này lỗ hổng ở đâu: " đơn giản nằm ở việc xác minh tính toàn vẹn của JWT, nó liên quan đến signature"


