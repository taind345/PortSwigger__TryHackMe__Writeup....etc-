# Tổng hợp Lý thuyết Toàn diện về JWT Attacks (PortSwigger Web Security Academy)

> Tài liệu này được dịch, tổng hợp và biên soạn lại một cách tự nhiên, trực quan từ giáo trình **JWT Attacks** của **PortSwigger Web Security Academy**, kết hợp kinh nghiệm thực chiến và sơ đồ hóa chi tiết để dễ dàng tra cứu khi pentest/làm lab.

---

## 1. Bản chất của JSON Web Token (JWT) là gì?

**JSON Web Token (JWT)** là một tiêu chuẩn mở (RFC 7519) dùng để truyền tải thông tin an toàn giữa các bên dưới dạng một đối tượng JSON nhỏ gọn và khép kín. Trong các ứng dụng web hiện đại, JWT thường được sử dụng nhiều nhất cho **xác thực (Authentication)**, **quản lý phiên (Session Management)** và **phân quyền (Access Control)**.

### Sự khác biệt cốt lõi: Stateful Session vs Stateless JWT

* **Session Token truyền thống (Stateful):**
  Server sinh ra một chuỗi ngẫu nhiên (Session ID) gửi về cho trình duyệt lưu trong Cookie. Server **bắt buộc phải lưu trữ** thông tin session đó trong Database hoặc RAM/Redis cache. Mỗi request gửi lên, server phải truy vấn database xem session ID này là của ai.
* **JWT (Stateless):**
  Server đóng gói toàn bộ dữ liệu người dùng (User ID, quyền hạn, thời gian hết hạn...) vào chính cái token và gửi cho client. Server **không cần lưu bất kỳ trạng thái nào** của token trong database. Khi nhận request kèm JWT, server chỉ cần dùng thuật toán mã hóa để **xác thực chữ ký (Signature Verification)**. Nếu chữ ký hợp lệ, server tin tưởng luôn dữ liệu bên trong token.

> [!TIP]
> Nhờ tính chất Stateless, JWT cực kỳ được ưa chuộng trong các hệ thống **Microservices** và kiến trúc phân tán: một Authentication Service cấp token, và hàng chục service khác có thể tự xác thực token độc lập mà không cần gọi ngược lại Auth Service hay dùng chung Database session.

---

## 2. Giải phẫu cấu trúc JWT (Anatomy of a JWT)

Một JWT chuẩn bao gồm **3 phần riêng biệt**, ngăn cách nhau bởi dấu chấm (`.`):

$$\text{Header} \ .\  \text{Payload} \ .\  \text{Signature}$$

![Tampering with JWT claims](images/jwt-infographic.jpg)

Ví dụ thực tế của một JWT:
```text
eyJraWQiOiI5MTM2ZGRiMy1jYjBhLTRhMTktYTA3ZS1lYWRmNWE0NGM4YjUiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTY0ODAzNzE2NCwibmFtZSI6IkNhcmxvcyBNb250b3lhIiwic3ViIjoiY2FybG9zIiwicm9sZSI6ImJsb2dfYXV0aG9yIn0.SYZBPIBg2CRjXAJ8vCER0LA_ENjII1JakvNQoP...
```

Cả 3 phần này nhìn loằng ngoằng như mã hóa, nhưng thực chất **Header** và **Payload** chỉ là **Base64URL-encoded** từ JSON thuần túy (không hề mã hóa bảo mật).

```mermaid
flowchart LR
    A["Header (JSON)"] -->|Base64URL| D["eyJhbGci..."]
    B["Payload (JSON)"] -->|Base64URL| E["eyJzdWIi..."]
    C["Signature"] -->|HMAC / RSA| F["SflKxwRJ..."]
    D -.-> G["JWT: eyJhbGci... . eyJzdWIi... . SflKxwRJ..."]
    E -.-> G
    F -.-> G
```

### 2.1. Header (JOSE Header)
Chứa siêu dữ liệu (metadata) hướng dẫn cho server biết token này được xử lý như thế nào. Khi decode Base64URL sẽ thu được một JSON object:
```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "9136ddb3-cb0a-4a19-a07e-eadf5a44c8b5"
}
```
* `typ`: Loại token (thường là `"JWT"`).
* `alg`: Thuật toán dùng để tạo chữ ký (ví dụ: `HS256`, `RS256`, `ES256`, hoặc `none`).
* Các tham số bổ sung khác: `kid` (Key ID), `jwk`, `jku`, `cty`... (đây là mỏ vàng khai thác của attacker).

### 2.2. Payload (Claims)
Chứa dữ liệu thực tế mà server muốn truyền tải (thường gọi là các *Claims* - lời tuyên bố về người dùng):
```json
{
  "iss": "portswigger",
  "exp": 1648037164,
  "name": "Carlos Montoya",
  "sub": "carlos",
  "role": "blog_author",
  "email": "carlos@carlos-montoya.net"
}
```
Các claim phổ biến:
* `iss` (Issuer): Đơn vị phát hành token.
* `sub` (Subject): Định danh người dùng (User ID / Username).
* `aud` (Audience): Đối tượng nhận token (token này cấp cho ứng dụng nào).
* `exp` (Expiration Time): Thời điểm token hết hạn (dạng epoch timestamp).
* `iat` (Issued At): Thời điểm token được tạo.
* Các trường tùy chỉnh: `role`, `isAdmin`, `permissions`...

> [!WARNING]
> **Sai lầm phổ biến nhất của Newbie:** Nhầm lẫn giữa "Mã hóa (Encryption)" và "Encoding (Base64URL)".
> Bất kỳ ai cầm JWT cũng có thể vào `jwt.io` dán token vào và đọc sạch sành sanh nội dung trong Payload. **Tuyệt đối không lưu dữ liệu nhạy cảm** (mật khẩu, mã thẻ tín dụng, API key nội bộ) vào Payload!

### 2.3. Signature (Chữ ký số)
Đây là **bức tường phòng thủ duy nhất** của JWT để ngăn chặn việc sửa đổi dữ liệu trái phép:
* Server lấy: `base64Url(Header) + "." + base64Url(Payload)`.
* Sau đó đưa qua thuật toán băm kèm theo **Khóa bí mật (Secret key)** hoặc **Khóa riêng tư (Private key)** để tạo chữ ký.
* **Nguyên lý toàn vẹn dữ liệu:** Nếu client thay đổi dù chỉ 1 dấu cách hay 1 chữ cái trong Header hoặc Payload, chữ ký sinh ra sẽ hoàn toàn sai lệch. Nếu không có khóa của server, attacker không thể tự tính ra chữ ký hợp lệ cho dữ liệu đã chỉnh sửa.

---

## 3. Phân biệt: JWT vs JWS vs JWE

Nhiều tài liệu dùng lẫn lộn các khái niệm này. Thực tế mối quan hệ của chúng như sau:

![Relationship between JWT, JWS, and JWE](images/jwt-jws-jwe.jpg)

1. **JWT (JSON Web Token):** Tiêu chuẩn chung quy định cách biểu diễn claims dưới dạng JSON.
2. **JWS (JSON Web Signature):** Token có chữ ký số (Header + Payload dưới dạng Base64URL + Signature). Dữ liệu **đọc được công khai**, nhưng **không giả mạo được**. Trong 99% tài liệu bảo mật và thực tế (kể cả PortSwigger), khi người ta nói đến "JWT", họ đang ám chỉ **JWS**.
3. **JWE (JSON Web Encryption):** Token được mã hóa toàn bộ dữ liệu. Người ngoài không có private key/secret key giải mã thì **không thể đọc được** nội dung bên trong.

---

## 4. Tại sao các lỗ hổng JWT lại xuất hiện? (Vulnerability Root Cause)

Các lỗ hổng JWT không xuất phát từ việc thuật toán mã hóa bị bẻ gãy, mà xuất phát từ **lỗi thiết kế (Design Flaws)** và **lỗi triển khai (Implementation Flaws)** của lập trình viên:

1. **Tin tưởng mù quáng vào dữ liệu từ client:** Token là hoàn toàn do client gửi lên. Trước khi verify chữ ký, server không được phép tin bất kỳ trường nào trong Header hay Payload. Nhưng nhiều hệ thống lại đọc `alg`, `kid`, `jwk` trong Header để quyết định cách thức verify — trao cho attacker quyền kiểm soát luồng kiểm tra!
2. **Server không lưu trạng thái (Stateless):** Server không lưu bản sao của token gốc. Nếu khâu verify chữ ký bị lỗi, server sẽ đọc ngay Payload giả mạo (`"role": "admin"`) và cấp quyền tối cao cho attacker.
3. **Sử dụng sai hàm của thư viện:** Rất nhiều dev nhầm lẫn giữa hàm giải mã (`decode()`) và hàm xác thực (`verify()`).
4. **Khóa bí mật yếu:** Đặt secret đơn giản, hardcoded mã nguồn, hoặc copy code mẫu từ StackOverflow.

---

## 5. Phân loại các đòn tấn công JWT (PortSwigger Attack Matrix)

```mermaid
mindmap
  root((JWT Attacks))
    1. Flawed Signature Verification
      Accepting arbitrary signatures
      Accepting unsigned tokens alg=none
    2. Weak Signing Keys
      Brute-force secret key
      Hashcat mode 16500
    3. Header Parameter Injection
      jwk injection
      jku injection
      kid path traversal
      kid SQL injection
      cty and x5c
    4. Algorithm Confusion
      RS256 to HS256
      Dùng Public Key làm HMAC Secret
      Trích xuất Public Key sig2n
```

---

### Đòn 1: Lỗi xác thực chữ ký lỏng lẻo (Flawed Signature Verification)

#### 1.1. Chấp nhận chữ ký tùy ý (Arbitrary Signatures)
* **Nguyên nhân:** Các thư viện JWT thường cung cấp 2 hàm:
  * `verify(token, secret)`: Kiểm tra chữ ký hợp lệ rồi mới trả về payload.
  * `decode(token)`: Chỉ đơn thuần Base64URL decode để lấy nội dung JSON mà **không kiểm tra chữ ký**.
  * Nhiều lập trình viên gọi nhầm hàm `decode()`. Kết quả: Client sửa payload thành `"sub": "admin"`, để nguyên chữ ký cũ hoặc gõ chữ ký rác, server vẫn tin tưởng 100%!
* **Liên kết Lab:** [[Lab-JWT authentication bypass via unverified signature]]

#### 1.2. Chấp nhận token không có chữ ký (`alg: "none"`)
* **Nguyên nhân:** Đặc tả JWS cho phép một cơ chế gọi là *Unsecured JWT* dành cho môi trường debug/nội bộ, bằng cách đặt header `"alg": "none"`.
* **Kỹ thuật tấn công:**
  1. Đổi header: `{"alg": "none", "typ": "JWT"}`.
  2. Sửa payload: đổi username thành tài khoản mục tiêu (ví dụ: `"sub": "administrator"`).
  3. Xóa hoàn toàn phần signature nhưng **phải giữ lại dấu chấm phân cách cuối cùng**:
     `base64(header).base64(payload).`
* **Vượt cơ chế filter:** Một số server có filter cấm chữ `"none"`, có thể bypass bằng chữ hoa/thường lẫn lộn (`"None"`, `"NONE"`, `"nOnE"`).
* **Liên kết Lab:** [[Lab-JWT flawed signature verification]] hoặc [[Lab-JWT authentication bypass via flawed signature verification]]

---

### Đòn 2: Bẻ khóa Secret Key yếu (Brute-forcing Weak Signing Keys)

![Signing and verifying JWTs using a symmetric algorithm](images/jwt-symmetric-signing-algorithm.jpg)

Với các thuật toán đối xứng như **HS256 (HMAC-SHA256)**, server dùng một chuỗi bí mật (Shared Secret) duy nhất để vừa ký vừa xác minh token.
* **Lỗ hổng:** Lập trình viên đặt secret quá ngắn, dùng từ điển đơn giản (như `secret`, `123456`, `password`), hoặc quên không đổi secret mặc định khi copy code mẫu từ tutorial.
* **Khai thác Offline cực nhanh với Hashcat:**
  Attacker không cần gửi request lên server (tránh bị WAF/rate-limit chặn), mà chỉ cần lấy một token hợp lệ và brute-force offline trên card đồ họa (GPU):
  ```bash
  hashcat -a 0 -m 16500 <jwt_token> <wordlist_rockyou.txt>
  ```
* **Sau khi tìm được secret:** Vào Burp Suite (tiện ích *JWT Editor*) hoặc `jwt.io`, sửa payload thành admin và dùng secret vừa tìm được để ký lại chữ ký xịn -> Chiếm tài khoản!
* **Liên kết Lab:** [[Lab-JWT authentication bypass via weak signing key]]

---

### Đòn 3: Tiêm tham số Header (JWT Header Parameter Injections)

Đặc tả JWS cung cấp nhiều tham số mở rộng trong header để giúp server tìm đúng khóa xác thực. Khi server tin tưởng dữ liệu do attacker truyền trong header này, thảm họa sẽ xảy ra.

#### 3.1. Nhúng trực tiếp Public Key qua tham số `jwk` (JWK Header Injection)
* **Bản chất:** `jwk` (JSON Web Key) cho phép nhúng trực tiếp một đối tượng JSON đại diện cho public key vào ngay header của token.
* **Lỗ hổng:** Server cấu hình sai, thay vì chỉ dùng các key trong whitelist nội bộ, nó lại lấy chính cái public key nằm trong trường `jwk` của token do client gửi lên để kiểm tra chữ ký!
* **Kịch bản khai thác:**
  1. Attacker tự sinh một cặp khóa RSA riêng trên máy mình (có cả Private Key và Public Key).
  2. Sửa payload của token thành `"sub": "administrator"`.
  3. Nhúng **Public Key của attacker** vào header `jwk`.
  4. Ký token bằng **Private Key của attacker**.
  5. Khi server nhận token, nó lấy `jwk` (public key của attacker) ra verify -> Chữ ký khớp hoàn toàn -> Bị bypass!
* **Liên kết Lab:** [[Lab-JWT authentication bypass via jwk header injection]]

#### 3.2. Tiêm đường dẫn danh sách khóa qua tham số `jku` (JKU Header Injection)
* **Bản chất:** Thay vì nhúng key, `jku` (JWK Set URL) chỉ định một đường dẫn URL (ví dụ: `https://example.com/.well-known/jwks.json`) để server tải về danh sách public key hợp lệ.
* **Khai thác:** Attacker dựng một server chứa file `jwks.json` của mình, sau đó sửa header `jku` trỏ về server attacker. Nếu server không kiểm tra whitelist domain của `jku`, nó sẽ tải key của attacker về để xác thực. (Kỹ thuật nâng cao: kết hợp Open Redirect hoặc SSRF để bypass nếu server có check domain).

#### 3.3. Thao túng tham số `kid` (Key ID Injection)
Tham số `kid` là một chuỗi định danh giúp server chọn đúng key khi có nhiều key khác nhau. Chuẩn JWS không quy định cấu trúc của `kid`, nên lập trình viên thường dùng nó làm **đường dẫn file (File Path)** hoặc **khóa truy vấn trong Database (SQL Query)**.

* **Path Traversal trên `kid`:**
  Nếu server đọc file key dựa trên `kid`, attacker có thể dùng `../../` để trỏ tới một file có nội dung tĩnh đoán trước được trên server.
  * Chiêu độc: Trỏ về `/dev/null` trên hệ điều hành Linux (`{"kid": "/../../../../../../dev/null"}`).
  * Vì `/dev/null` là file rỗng, nội dung server đọc được là **chuỗi rỗng (`""`)**.
  * Attacker chuyển sang thuật toán `HS256`, ký token với **Secret là một chuỗi rỗng Base64 (`AA==` hoặc rỗng)**!
  * **Liên kết Lab:** [[Lab-JWT kid path traversal]]

* **SQL Injection trên `kid`:**
  Nếu server query key: `SELECT key FROM jwt_keys WHERE kid = '$kid'`.
  Attacker inject:
  ```json
  {
    "alg": "HS256",
    "kid": "key1' UNION SELECT 'my_secret_string'--"
  }
  ```
  Database sẽ trả về chuỗi `my_secret_string`. Server lấy chuỗi này làm HMAC secret. Attacker chỉ việc ký token với secret `my_secret_string` là xong!
  * **Liên kết Lab:** [[Lab-JWT kid SQL injection]]

#### 3.4. Các tham số header đáng chú ý khác
* `cty` (Content Type): Báo hiệu kiểu dữ liệu của payload. Nếu bypass được signature, có thể đổi `cty` thành `text/xml` hoặc `application/x-java-serialized-object` để kích hoạt tấn công **XXE** hoặc **Insecure Deserialization**.
* `x5c` (X.509 Certificate Chain): Nhúng chuỗi chứng chỉ X.509 để verify token. Có thể bị khai thác bằng chứng chỉ tự ký hoặc lỗi trong trình parser X.509 (như CVE-2017-2800).

---

### Đòn 4: Lỗi Nhầm Lẫn Thuật Toán (Algorithm Confusion / Key Confusion)

Đây là một trong những đòn tấn công tinh tế và kinh điển nhất của JWT, xuất hiện do sự khác biệt cơ chế giữa mã hóa Đối Xứng và Bất Đối Xứng.

#### Đối xứng (Symmetric) vs Bất đối xứng (Asymmetric)

![Signing and verifying JWTs using an asymmetric algorithm](images/jwt-asymmetric-signing-algorithm.jpg)

| Đặc điểm | Thuật toán Đối Xứng (HS256) | Thuật toán Bất Đối Xứng (RS256) |
| :--- | :--- | :--- |
| **Bản chất key** | Dùng chung **1 Khóa Bí Mật (Secret)** | Dùng **cặp khóa (Key Pair)** |
| **Khâu ký (Sign)** | Dùng Secret Key | Dùng **Private Key** (chỉ server giữ) |
| **Khâu duyệt (Verify)**| Dùng Secret Key | Dùng **Public Key** (chia sẻ công khai) |

#### Bản chất xuất hiện lỗ hổng
Nhiều thư viện JWT thiết kế một hàm `verify()` dùng chung cho mọi thuật toán:
```javascript
// Mã giả của thư viện JWT
function verify(token, secretOrPublicKey) {
    var algorithm = token.header.alg;
    if (algorithm === "RS256") {
        // Dùng secretOrPublicKey như một RSA Public Key
        return rsaVerify(token, secretOrPublicKey);
    } else if (algorithm === "HS256") {
        // Dùng secretOrPublicKey như một HMAC Secret Key
        return hmacVerify(token, secretOrPublicKey);
    }
}
```

Phía lập trình viên ứng dụng nghĩ rằng hệ thống của mình chỉ dùng RS256, nên họ viết code truyền thẳng Public Key của server vào hàm verify:
```javascript
var publicKey = getServerRSAPublicKey();
jwt.verify(incomingToken, publicKey);
```

**Kẽ hở chết người:**
1. Server công khai Public Key (thường qua `/.well-known/jwks.json` hoặc chứng chỉ TLS). Attacker tải được Public Key này về.
2. Attacker đổi header token từ `"alg": "RS256"` thành **`"alg": "HS256"`**.
3. Attacker ký token bằng thuật toán **HS256**, lấy **chính nội dung file Public Key làm HMAC Secret**!
4. Khi gửi token lên server, hàm `verify()` đọc thấy `alg: "HS256"`. Nó sẽ lấy đối số thứ 2 (chính là Public Key của server) và coi đó là một chuỗi HMAC Secret để tính toán chữ ký.
5. Cả 2 bên đều dùng chung nội dung Public Key làm secret -> **Chữ ký hoàn toàn hợp lệ!**

```mermaid
sequenceDiagram
    autonumber
    actor Attacker
    participant Server as Web Server (Vulnerable)
    
    Attacker->>Server: 1. Tải Public Key công khai (/jwks.json)
    Server-->>Attacker: Trả về Public Key (RS256)
    Note over Attacker: 2. Đổi alg từ RS256 -> HS256<br/>3. Ký token bằng HS256 với Secret = Public Key
    Attacker->>Server: 4. Gửi JWT giả mạo (sub: admin, alg: HS256)
    Note over Server: 5. Server đọc alg: HS256<br/>Dùng Public Key làm HMAC Secret để verify<br/>=> Chữ ký KHỚP HOÀN TOÀN!
    Server-->>Attacker: 6. Đăng nhập thành công quyền Administrator!
```

#### Quy trình 4 bước thực hiện Algorithm Confusion:
1. **Bước 1 - Lấy Public Key của server:** Tìm ở `/jwks.json`, `/.well-known/jwks.json`, hoặc trích xuất từ 2 token có sẵn bằng công cụ `sig2n` (`docker run --rm -it portswigger/sig2n <token1> <token2>`).
2. **Bước 2 - Chuẩn hóa Public Key:** Public Key server dùng trong code thường ở định dạng **X.509 PEM**. Từng byte (kể cả dấu xuống dòng `\n`) phải khớp chính xác tuyệt đối với file trên server.
3. **Bước 3 - Sửa đổi Token:** Đổi payload thành admin, đổi header `alg` thành `HS256`.
4. **Bước 4 - Ký lại token:** Dùng HMAC-SHA256 với secret là chuỗi PEM Public Key vừa chuẩn hóa.
* **Liên kết Lab:** [[Lab-JWT algorithm confusion]]

---

## 6. Bộ công cụ thực chiến (Arsenal)

1. **Burp Suite Extension - JWT Editor:**
   * Tiện ích bắt buộc phải cài từ BApp Store.
   * Cung cấp tab riêng biệt để xem, decode, chỉnh sửa JWT trực tiếp trong HTTP request.
   * Tự động hóa các đòn đánh: *None algorithm attack*, *Embedded JWK attack*, *Algorithm Confusion*, *Signature verification bypass*.
2. **Hashcat:**
   * Dùng để brute-force offline secret key HS256:
     ```bash
     hashcat -a 0 -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt
     ```
3. **docker portswigger/sig2n & jwt_forgery.py:**
   * Khôi phục public key từ 2 token JWT đã ký khi server không công khai endpoint jwks.
4. **jwt.io Debugger:**
   * Website trực quan để test decode/encode và hiểu cấu trúc token nhanh chóng.

---

## 7. Hướng dẫn phòng chống toàn diện (Defensive Best Practices)

Để bảo vệ ứng dụng trước toàn bộ các vector tấn công JWT, cần áp dụng các nguyên tắc phòng thủ theo chiều sâu:

1. **Sử dụng thư viện cập nhật và cấu hình chặt chẽ:** Luôn cập nhật thư viện JWT lên bản mới nhất. Hiểu rõ cách thức hoạt động của các hàm verify.
2. **Kiểm tra chữ ký nghiêm ngặt (Strict Signature Verification):** Luôn bắt buộc phải verify chữ ký trước khi đọc dữ liệu payload. Tuyệt đối không dùng hàm `decode()` đơn thuần cho việc xác thực.
3. **Khóa cứng thuật toán cho phép (Algorithm Whitelisting):**
   * Server phải chỉ định rõ chỉ chấp nhận 1 thuật toán duy nhất mong muốn (ví dụ: `algorithms=['RS256']`).
   * **Tuyệt đối cấm** thuật toán `"none"`.
   * Ngăn chặn Algorithm Confusion bằng cách không dùng chung hàm verify cho cả HS256 và RS256 với cùng 1 biến khóa.
4. **Bảo mật Secret Key (đối với HS256):**
   * Secret key phải là một chuỗi ngẫu nhiên có độ entropy cao (tối thiểu 256-bit / 32 ký tự ngẫu nhiên mã hóa mạnh).
   * Không bao giờ hardcode trong source code, không dùng các từ khóa có trong từ điển.
5. **Không tin tưởng các tham số Header từ Client:**
   * Nếu dùng `kid`: Phải whitelist các giá trị hợp lệ. Tuyệt đối không nối chuỗi `kid` vào câu lệnh SQL hoặc đường dẫn filesystem.
   * Nếu dùng `jku`: Phải whitelist danh sách các domain/URL được phép tải key.
   * Từ chối tham số `jwk` nhúng trực tiếp trừ khi có cơ chế xác minh nguồn gốc đáng tin cậy.
6. **Thực hành tốt với Claims:**
   * Luôn thiết lập thời gian hết hạn (`exp`) ngắn hạn.
   * Kiểm tra đầy đủ các claim: `iss` (đúng bên phát hành), `aud` (đúng ứng dụng của mình).
   * Có cơ chế thu hồi token (Token Revocation / Blacklist) khi người dùng logout hoặc đổi mật khẩu.

---

> [!NOTE]
> Bảng tra cứu tiến độ và thứ tự giải Lab có tại: [[0-index JWT]]
