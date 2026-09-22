# 1-Tổng quan

Tóm gọn đúng 5 đòn cốt lõi mày vừa học:
* **BOLA (IDOR):** Đổi ID trên URL hoặc Body để xem/sửa trộm dữ liệu nạn nhân vì server quên check quyền sở hữu.
* Lộ dữ liệu thừa: Soi JSON thô qua Burp/F12 để lượm key, hash và các trường ẩn mà web giấu đi.
* **Mass Assignment:** Nhét thêm trường nhạy cảm (`role: admin`) vào request cập nhật để tự leo quyền quản trị.
* Lỗi **Auth & Rate Limit:** Brute-force mật khẩu/OTP thoải mái do server không giới hạn tốc độ; đục token JWT cấu hình ẩu.
* Chơi combo: Nối chuỗi từ việc bới data ẩn -> bơm quyền admin -> mở khóa endpoint cấm hốt flag.

Về phòng thủ thì làm ngược lại: Check quyền ở từng endpoint, lọc dữ liệu trước khi trả về, dùng whitelist cho dữ liệu gửi lên và siết chặt Rate Limit cùng JWT.
# 2-How API works
Toàn bộ đoạn tài liệu trên được dịch và giải thích chuẩn chỉ theo ngôn ngữ kỹ thuật cho mày:
![Structural diagram showing the three-tier flow of a RESTful API. On the left, a Client panel lists HTTP methods (GET, POST, PATCH, DELETE). In the centre, an API Server panel contains Authentication and Authorisation layers connected by an arrow. On the right, a Database panel lists resource tables (users, products, orders). Arrows labelled JSON and Response connect Client to API Server, and arrows labelled Query and Result connect API Server to Database.|700](https://cdn-images.tryhackme.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/645b19f5d5848d004ab9c9e2-1774422667931.svg)**Kiến trúc REST và Tài nguyên (Resources & Endpoints)**
REST không phải là giao thức hay tiêu chuẩn bắt buộc, mà là một phong cách kiến trúc gồm các quy ước để dev xây dựng API qua giao thức HTTP. API tuân thủ quy ước này gọi là RESTful API.
>là bộ quy tắc xây dựng API. => ta mới có tên gọi RESRful API

Khái niệm cốt lõi trong REST là "tài nguyên" (resource) – chính là các đối tượng dữ liệu như người dùng, sản phẩm, đơn hàng. Mỗi tài nguyên được định danh bằng một URL gọi là endpoint (ví dụ: `/v1/users`, `/v1/products`). Cấu trúc URL có tính phân cấp rõ ràng: `/v1` là phiên bản API, `/users` là tập hợp tài nguyên, và `/v1/users/42` trỏ tới người dùng có ID 42. Cấu trúc dễ đoán này giúp dân pentest dễ dàng mò ra cách gọi tài nguyên để khai thác các lỗ hổng phân quyền (BOLA / IDOR).

> [!NOTE] Title
> -Cấu trúc của API, gồm resource, là các epoint trỏ tới các tài nguyên, được đinh danh và truy cập qua id
> 

**Các phương thức HTTP (HTTP Methods)**
RESTful API dùng các method HTTP chuẩn tương ứng với mô hình CRUD:

* GET (Read): Lấy dữ liệu về, tuyệt đối không làm thay đổi dữ liệu trên server. Ví dụ: `GET /v1/products/1`.
* POST (Create): Tạo mới tài nguyên hoặc gửi dữ liệu đi xử lý. Ví dụ: `POST /v1/auth/login` gửi kèm thông tin đăng nhập để lấy token.
* PUT (Full Update): Ghi đè toàn bộ tài nguyên. Những trường dữ liệu không gửi kèm có thể bị xóa trắng hoặc về giá trị mặc định.
* PATCH (Partial Update): Chỉ sửa đổi đúng các trường được gửi lên trong request body, giữ nguyên toàn bộ các trường khác.
* DELETE (Delete): Xóa tài nguyên trên server. Ví dụ: `DELETE /v1/users/4`.

Điểm khác nhau giữa PUT và PATCH rất quan trọng vì cơ chế kiểm tra dữ liệu đầu vào của hai phương thức này thường khác nhau khi làm bài test bảo mật.

**Mã phản hồi (Status Codes) và ý nghĩa trong Pentest**
* 200 OK: Request thành công.
* 201 Created: Tạo tài nguyên mới thành công (thường gặp sau lệnh POST).
* 204 No Content: Xử lý thành công nhưng không có dữ liệu trả về (thường gặp sau lệnh DELETE).
* 400 Bad Request: Dữ liệu gửi lên sai định dạng, hữu ích để dò xem backend kiểm tra input thế nào.
* 401 Unauthorized: Chưa đăng nhập hoặc token không hợp lệ (server không biết mày là ai).
* 403 Forbidden: Đã đăng nhập nhưng không có quyền truy cập (server biết mày là ai nhưng cấm cửa – dấu hiệu mấu chốt khi săn lỗi BOLA/IDOR).
* 404 Not Found: Tài nguyên hoặc đường dẫn không tồn tại.
* 405 Method Not Allowed: Endpoint đó không hỗ trợ phương thức HTTP mày vừa gửi.
* 429 Too Many Requests: Bị chặn do bắn quá nhiều request trong thời gian ngắn (dính Rate Limit).
* 500 Internal Server Error: Server bị lỗi sập mã nguồn bên trong, dấu hiệu của lỗi logic hoặc dính Injection.

> [!NOTE] Title
> hiểu như này , đọc phần này[[API]]
> -> thứ nhất API nó là các hàm gọi tới các các dịch vụ(hàm từ thư viện cũng là 1 dạng), nhưng khi gọi 1 hàm API như *openAI(.....hỏi....)* , thì dữ liệu sẽ được truyền từ API-->HTTP--> server để trả về response qua cái hàm API *openAI()*, well nó chỉ đơn giản như vậy thôi, nên nó mới cần các cơ chế authentication và authorise

**Cấu trúc Request / Response và Xác thực (Authentication)**
Một request API chuẩn gồm: Method HTTP, đường dẫn URL, các Header (chứa metadata, loại dữ liệu, token xác thực) và phần Body (chủ yếu dùng định dạng JSON).

**Cóa 3 cơ chế xác thực API phổ biến nhất:**
1. **API Keys**: Dạng đơn giản nhất, client gửi một chuỗi key cố định trong header (như `X-API-Key`). Dạng này sống lâu, hay bị lộ và dễ bị lạm dụng toàn quyền nếu bị rò rỉ.
2. **Bearer Tokens:** Đăng nhập thành công, server nhả về một chuỗi token. Client kẹp token này vào header `Authorization: Bearer <token>` ở các request sau. Loại token này thường có thời hạn ngắn và thu hồi được.
3. **JSON Web Tokens (JWT)**: Định dạng phổ biến nhất của Bearer Token. Một chuỗi JWT gồm 3 phần ngăn cách bằng dấu chấm (header.payload.signature) được mã hóa Base64.

Lưu ý sống còn: <u>Payload của JWT chỉ là đổi bảng mã Base64 chứ không hề mã hóa bí mật, bất kỳ ai cầm token cũng có thể giải mã ra đọc sạch các thông tin bên trong như user_id, role và thời hạn hết hạn (exp)</u>.

> [!NOTE] Title
>well, cách hoạt động trên khá giống với cách token, và jwt hoạt động với từng HTTP request trong 1 ứng dụng web thông thường

# 3-broken object level authorization 
<u>Bản chất của BOLA (Broken Object Level Authorization) thực ra chính là lỗi IDOR quen thuộc</u>, nhưng được gọi riêng trong mảng API Security. Nó đứng top 1 OWASP API vì cực kỳ phổ biến và dễ ăn đòn nặng nhất.

**Bản chất lỗi:**
Server đã biết mày là ai (Authentication thành công với token hợp lệ), nhưng khi mày đòi xem một món đồ (Object) thì server lại quên kiểm tra xem món đồ đó có thực sự thuộc về mày hay không (Authorization thất bại).

![Diagram showing three horizontal request flows. Row 1 (Legitimate request): User 4 sends GET /v1/users/4 and receives 200 OK with their own data. Row 2 (BOLA attack): User 4 sends GET /v1/users/1 with a highlighted changed ID and receives 200 OK with User 1's data. Row 3 (Secure implementation): User 4 sends GET /v1/users/1 and receives 403 Forbidden because the token ID does not match the URL ID.|700](https://cdn-images.tryhackme.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/645b19f5d5848d004ab9c9e2-1774422709746.svg)**Tại sao BOLA lại nhan nhản khắp nơi:**
Các framework làm API chỉ hỗ trợ xác thực đăng nhập cơ bản và điều hướng route. Việc kiểm tra "thằng user_id trong token có đúng là chủ sở hữu của bản ghi ID này không" hoàn toàn phụ thuộc vào việc dev phải tự viết code tay ở từng endpoint một. Dev viết 100 endpoint mà quên kiểm tra ở đúng 1 chỗ là toang ngay.

> [!NOTE] Title
> Tao chưa hiểu nó khó chỗ nào mà ko kiểm tra xác thực cho các enpoint được?
> cần ví dụ ở phần này

![[Pasted image 20260828171344.png]]![[Pasted image 20260828171411.png]]
**Cách thức hoạt động:**
Mày đăng nhập tài khoản của mày (ví dụ user 4).
Khi mày gửi request lấy thông tin: `GET /v1/users/4/orders`, server thấy token đúng nên trả về đơn hàng của mày.
Mày táy máy sửa số 4 thành số 1: `GET /v1/users/1/orders`.

* Nếu code chuẩn: Server so sánh ID trong token (4) khác với ID trên URL (1), nó sẽ chặn lại và trả mã 403 Forbidden.
* Nếu dính BOLA: Server cứ thấy token xịn là bốc nguyên số 1 trên URL chui vào database lấy đơn hàng của user 1 trả về cho mày.

Các vị trí thường xuất hiện ID để tráo đổi:

* Nằm ngay trên đường dẫn URL: `/v1/users/1` hay `/v1/orders/1045`
* Nằm ở query string: `?user_id=1` hoặc `?owner_id=13`
* Nằm trong body JSON: `{"user_id": 1, "status": "active"}`
* Nằm ở header tự chế: `X-User-ID: 1`
>cái này là IDOR thôi, chẳng có gì cả

**Mức độ nguy hiểm:**
Nếu ID là số tăng dần (1, 2, 3...), mày chỉ cần viết vài dòng script Python hoặc ném vào Burp Intruder cho chạy từ 1 đến 1000 là vét sạch database của cả hệ thống trong vài giây. Dù dev có đổi sang dùng mã UUID ngoằng ngoằng thì chỉ cần UUID đó vô tình bị lộ ở đâu đó là vẫn bị đục như thường.

BOLA không chỉ để đọc lén dữ liệu (GET), mà còn có thể dùng để sửa, xóa (PATCH, PUT, DELETE) dữ liệu của nạn nhân nếu endpoint cập nhật cũng quên check quyền.
# 4-broken authentication
Phần này dạy <u>2 cái tật cực kỳ ẩu của dev khi làm API: Lỗi xác thực (Broken Authentication) và Phơi bày dữ liệu vô tội vạ (Excessive Data Exposure).</u>

1. **Broken Authentication** (Hổng xác thực và đăng nhập)

* Không giới hạn tốc độ (No Rate Limiting):
<u>Web thường hay có mã Captcha hoặc khóa tài khoản sau 5 lần nhập sai. Nhưng với API, dev hay bỏ qua vì để tiện cho app kết nối.</u> Hậu quả là hacker vác tool như ffuf nã hàng chục nghìn mật khẩu một phút (brute-force) hoặc lấy danh sách tài khoản lộ từ web khác sang thử (credential stuffing) mà server không hề phản ứng hay trả mã 429 Too Many Requests.
![[Pasted image 20260828172943.png]]
* Lỗi triển khai JWT cẩu thả:
   * <u>Dùng Secret key quá yếu: Đặt key băm token là mật khẩu dễ đoán</u> kiểu "secret", "password123". Hacker vứt chuỗi JWT vào hashcat hoặc jwt_tool để brute-force tìm key, tìm ra là tự tạo token giả mạo quyền admin.
* Đòn tấn công alg "none": Sửa giá trị "alg" trong header thành "none", xóa sạch phần chữ ký ở đuôi token. Nhiều server xử lý lỏng lẻo vẫn chấp nhận coi như token hợp lệ.
* Quên kiểm tra hạn sử dụng (exp): Token dùng từ mấy năm trước hoặc bị lộ vẫn gửi lên server xài ầm ầm mà không bao giờ hết hạn.


2. **Excessive Data Exposure** (Lộ thừa thãi dữ liệu)

Bệnh này sinh ra do dev backend lười biếng:

* Thay vì chỉ chọn đúng mấy trường cần hiển thị, backend bốc nguyên một hàng trong database (SELECT * FROM users) rồi ném cả đống JSON thô về cho frontend.
* Dev tưởng rằng ở giao diện web chỉ cho hiện mỗi tên với avatar thì người dùng chỉ thấy thế. Nhưng dữ liệu nhạy cảm như password_hash, api_key, internal_notes hay IP người dùng đã nằm chình ình trong gói tin phản hồi (response).![[Pasted image 20260828172927.png]]
* Mày chỉ cần bật F12 (tab Network) hoặc soi qua Burp Suite là đọc được toàn bộ bí mật.
* Đòn kết hợp chí mạng: Nếu API vừa dính BOLA (cho phép đổi ID để xem user khác) vừa dính Excessive Data Exposure thì mày có thể tải sạch mật khẩu băm và API key của toàn bộ người dùng trong database về máy.

> [!NOTE] Title
> Cái này chỉ là vấn đề trong configuration thôi

# 5-Mass Assignment
Bản chất của **Mass Assignment** (Gán dữ liệu hàng loạt) cực kỳ đơn giản: <u>Dev lười, bốc nguyên cục JSON mày gửi lên rồi ném thẳng vào database để cập nhật mà không lọc xem trường nào được phép sửa.</u>
![Diagram comparing a normal profile update with a mass assignment attack. In the top row, a PATCH request containing only an email field passes through the API server and the user's role remains customer. In the bottom row, the same request includes an injected role field set to admin. The API server processes both fields, and the user's role is escalated to admin. The injected field is highlighted in amber and the escalated result is highlighted in red.|700](https://cdn-images.tryhackme.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/645b19f5d5848d004ab9c9e2-1774422819850.svg)
1.**Cách thức hoạt động**
Bình thường giao diện web chỉ có ô đổi email. Khi gửi, trình duyệt bắn: `{"email": "abc@gmail.com"}`.
Nhưng trong database, bảng user còn có các trường nhạy cảm như `role`, `is_admin`, `balance`, `email_verified`.
Nếu backend viết code ẩu dạng `User.update(req.body)`, mày chỉ cần bật Burp Suite hoặc sửa body gửi lên thành:
`{"email": "abc@gmail.com", "role": "admin"}`
Server nuốt trọn và cập nhật trường role thành admin trong database. Thế là mày tự phong quyền admin cho tài khoản của mình mà không cần phá cơ chế đăng nhập nào cả.

> [!NOTE] Title
> cái này lý thuyết vậy, ngoài đời đâu ai cho cập nhật dữ liệu một cách vô tội vạ như vậy

2.**Mẹo mò ra các trường nhạy cảm để chèn**

* Soi response của API (kết hợp với lỗi thừa thãi dữ liệu ở bài trước): Lúc gọi xem profile, nếu JSON trả về lòi ra mấy trường như `role`, `is_admin`, `credit_balance`, mày cứ bốc đúng tên các trường đó ném vào request sửa (PUT, PATCH) hoặc request tạo tài khoản (POST).
* Đọc tài liệu API (Swagger / OpenAPI): Những trường nào tài liệu ghi là readOnly (chỉ đọc) thì mày càng phải thử nhét vào gửi lên xem server có thực sự chặn hay không.

3.**Thiếu Rate Limit ở các tính năng khác**
Không chỉ trang login, nếu các endpoint khác không giới hạn tốc độ request thì cũng ăn đủ:

* Phá mã OTP: Mã xác thực 4 chữ số chỉ có đúng 10.000 trường hợp, bắn tool vài giây là mò ra mã đúng.
* Đốt tiền hệ thống: Spam liên tục vào các endpoint kích hoạt gửi tin nhắn SMS OTP hoặc email.
* *Dấu hiệu nhận biết API có chặn tốc độ chuẩn: Server sẽ trả mã 429 Too Many Requests và kẹp các header như* `X-RateLimit-Limit`, `X-RateLimit-Remaining`.

> [!NOTE] note
> Cái này hữu ích này, header trả về dạng X-RateLimit

4.Hướng dẫn làm bài thực hành trong lab

* Bước 1: Nhìn khung trên cùng gọi `GET /v1/users/me`, thấy trong response có trường `"role": "customer"`.
* Bước 2: Ở khung JSON của `PATCH /v1/users/me`, sửa lại nội dung thành:
`{"email": "testuser@shop.thm", "role": "admin"}`
* Bước 3: Bấm Send Request. Banner phía trên sẽ lập tức nhảy từ CUSTOMER sang ADMIN.
* Bước 4: Khung bên dưới sẽ mở khóa endpoint admin `GET /v1/admin/users`, mày bấm gửi request đó để xem danh sách toàn bộ user và lấy flag.

# 6- Putting it all together
Đoạn này nó đang bảo mày là t<u>hực chiến ngoài đời hiếm khi ăn sẵn một lỗi là chiếm quyền được ngay</u>, <u>mà phải biết "chơi combo" nối nhiều lỗi lại với nhau để đấm sập hệ thống.</u>

Một lỗi lẻ tẻ như đọc trộm thông tin thì chỉ xem cho vui mắt. Nhưng nếu mày kết hợp combo 3 món: lộ dữ liệu thừa + gán bừa thuộc tính (Mass Assignment) + lỗi phân quyền, mày sẽ leo từ một thằng ất ơ lên thẳng trùm cuối Admin.

Cái bài thử thách cuối này bắt mày tự phối hợp 3 bước:

Bước 1: Bật soi profile tài khoản mày (user ID 4). Nhìn vào cục JSON thô trả về xem nó phơi bày trường ẩn nào hay ho (chính là trường role đang để customer).

Bước 2: Vác luôn cái tên trường role đó nhét vào request sửa thông tin (dùng PATCH) rồi đổi giá trị thành admin. Server ngáo ngơ nuốt trọn cục này là tài khoản của mày tự động nhảy lên làm sếp.

Bước 3: Có quyền Admin trong tay rồi thì vác mặt vào endpoint quản trị bị khóa nãy giờ (dạng `/v1/admin/...`) để hốt cái flag về nộp bài.

Mày cứ làm y hệt mấy bài trước tao với mày vừa thông não là qua dễ ợt, toàn võ cũ gom lại một chỗ thôi.