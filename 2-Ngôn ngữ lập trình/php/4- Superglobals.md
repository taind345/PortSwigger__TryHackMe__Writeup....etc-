[[superglobal]]

**Superglobals** là các biến đặc biệt có sẵn trong PHP. Bạn không cần tự khai báo, PHP tự tạo ra chúng để bạn lấy dữ liệu từ request, server, session, cookie, file upload, môi trường chạy chương trình.

Điểm quan trọng: superglobals có thể được truy cập ở mọi nơi trong PHP, kể cả bên trong function.

```php
<?php
function showMethod() {
    echo $_SERVER["REQUEST_METHOD"];
}

showMethod();
?>
```

---

## 1. `$_GET`

`$_GET` dùng để lấy dữ liệu được gửi qua URL query string.
![[Pasted image 20260509152114.png]]

Ví dụ URL:

```text
http://localhost/search.php?keyword=php&page=2
```

File `search.php`:

```php
<?php
$keyword = $_GET["keyword"];
$page = $_GET["page"];

echo "Từ khóa: " . $keyword;
echo "Trang: " . $page;
?>
```

Kết quả:

```text
Từ khóa: php
Trang: 2
```

Thường dùng cho:

```text
$_GET
├── Tìm kiếm
├── Phân trang
├── Lọc dữ liệu
├── Truyền id bài viết
└── Truyền tham số không nhạy cảm
```

Ví dụ:

```text
/product.php?id=15
```

```php
<?php
$id = $_GET["id"];

echo "Sản phẩm ID: " . $id;
?>
```

Không nên dùng `$_GET` cho password, token bí mật, dữ liệu nhạy cảm vì dữ liệu hiện trên URL.

---

## 2. `$_POST`

`$_POST` dùng để lấy dữ liệu được gửi từ form bằng phương thức `POST`.

Ví dụ form đăng nhập:

```php
<form method="POST" action="login.php">
    <input type="email" name="email">
    <input type="password" name="password">
    <button type="submit">Đăng nhập</button>
</form>
```

File `login.php`:

```php
<?php
$email = $_POST["email"];
$password = $_POST["password"];

echo "Email: " . $email;
?>
```

Thường dùng cho:

```text
$_POST
├── Đăng nhập
├── Đăng ký
├── Tạo bài viết
├── Gửi form liên hệ
├── Cập nhật thông tin
└── Gửi dữ liệu không nên nằm trên URL
```

Khi in dữ liệu từ người dùng ra HTML, nên dùng `htmlspecialchars()`:

```php
<?php
$name = $_POST["name"];

echo htmlspecialchars($name);
?>
```

---

## 3. `$_REQUEST`

`$_REQUEST` có thể chứa dữ liệu từ `$_GET`, `$_POST`, và đôi khi `$_COOKIE`.

Ví dụ:

```php
<?php
$username = $_REQUEST["username"];

echo $username;
?>
```

Vấn đề: `$_REQUEST` dễ gây mơ hồ vì bạn không biết dữ liệu đến từ URL, form POST hay cookie.

Trong thực tế nên ưu tiên dùng rõ ràng:

```text
Nên dùng
├── $_GET   → dữ liệu từ URL
└── $_POST  → dữ liệu từ form POST

Hạn chế dùng
└── $_REQUEST → vì nguồn dữ liệu không rõ ràng
```

---

## 4. `$_SERVER`

`$_SERVER` chứa thông tin về server, request, file đang chạy, method, URL, user agent.

Ví dụ lấy method của request:

```php
<?php
echo $_SERVER["REQUEST_METHOD"];
?>
```

Kết quả có thể là:

```text
GET
POST
PUT
DELETE
```

Ví dụ kiểm tra form có được submit bằng POST không:

```php
<?php
if ($_SERVER["REQUEST_METHOD"] === "POST") {
    echo "Form đã được gửi";
}
?>
```

Một số key hay dùng:

```text
$_SERVER
├── REQUEST_METHOD
│   └── GET, POST, PUT, DELETE...
├── REQUEST_URI
│   └── Đường dẫn hiện tại
├── PHP_SELF
│   └── File PHP hiện tại
├── HTTP_HOST
│   └── Tên host
├── HTTP_USER_AGENT
│   └── Thông tin trình duyệt
├── REMOTE_ADDR
│   └── IP của client
└── SCRIPT_NAME
    └── Tên script đang chạy
```

Ví dụ:

```php
<?php
echo $_SERVER["HTTP_HOST"];
echo $_SERVER["REQUEST_URI"];
?>
```

---

## 5. `$_SESSION`

`$_SESSION` dùng để lưu dữ liệu theo phiên làm việc của người dùng trên server.

Ví dụ sau khi đăng nhập thành công:

```php
<?php
session_start();

$_SESSION["user_id"] = 10;
$_SESSION["username"] = "tai";

echo "Đã đăng nhập";
?>
```

Ở trang khác, bạn có thể lấy lại thông tin session:

```php
<?php
session_start();

echo $_SESSION["username"];
?>
```

Muốn dùng `$_SESSION`, bắt buộc gọi:

```php
session_start();
```

trước khi đọc hoặc ghi session.

Thường dùng cho:

```text
$_SESSION
├── Trạng thái đăng nhập
├── User ID hiện tại
├── Role của user
├── Flash message
├── Giỏ hàng tạm thời
└── CSRF token
```

Ví dụ kiểm tra người dùng đã đăng nhập chưa:

```php
<?php
session_start();

if (!isset($_SESSION["user_id"])) {
    echo "Bạn chưa đăng nhập";
    exit;
}

echo "Chào user #" . $_SESSION["user_id"];
?>
```

Đăng xuất:

```php
<?php
session_start();

session_destroy();

echo "Đã đăng xuất";
?>
```

---

## 6. `$_COOKIE`

`$_COOKIE` dùng để đọc dữ liệu cookie được lưu trên trình duyệt.

Tạo cookie:

```php
<?php
setcookie("theme", "dark", time() + 3600);
?>
```

Cookie này tên là `theme`, giá trị là `dark`, tồn tại trong 1 giờ.

Đọc cookie:

```php
<?php
if (isset($_COOKIE["theme"])) {
    echo $_COOKIE["theme"];
}
?>
```

Thường dùng cho:

```text
$_COOKIE
├── Lưu theme sáng/tối
├── Lưu ngôn ngữ
├── Remember me token
├── Tracking đơn giản
└── Tuỳ chọn người dùng
```

Khác biệt giữa session và cookie:

```text
Session
├── Lưu chủ yếu trên server
├── An toàn hơn cookie
└── Phù hợp cho login state

Cookie
├── Lưu trên trình duyệt
├── Người dùng có thể xem/sửa
└── Không nên lưu dữ liệu nhạy cảm trực tiếp
```

Không nên lưu password trong cookie.

---

## 7. `$_FILES`

`$_FILES` dùng để xử lý file upload từ form.

Form upload phải có:

```html
<form method="POST" enctype="multipart/form-data" action="upload.php">
    <input type="file" name="avatar">
    <button type="submit">Upload</button>
</form>
```

File `upload.php`:

```php
<?php
$fileName = $_FILES["avatar"]["name"];
$tmpPath = $_FILES["avatar"]["tmp_name"];
$fileSize = $_FILES["avatar"]["size"];
$fileError = $_FILES["avatar"]["error"];

move_uploaded_file($tmpPath, "uploads/" . $fileName);

echo "Upload thành công";
?>
```

Cấu trúc của `$_FILES["avatar"]`:

```text
$_FILES["avatar"]
├── name
│   └── Tên file gốc
├── type
│   └── MIME type do browser gửi
├── tmp_name
│   └── Đường dẫn file tạm trên server
├── error
│   └── Mã lỗi upload
└── size
    └── Kích thước file
```

Cách an toàn hơn:

```php
<?php
if ($_FILES["avatar"]["error"] === UPLOAD_ERR_OK) {
    $tmpPath = $_FILES["avatar"]["tmp_name"];
    $originalName = basename($_FILES["avatar"]["name"]);

    $targetPath = "uploads/" . uniqid() . "-" . $originalName;

    move_uploaded_file($tmpPath, $targetPath);

    echo "Upload thành công";
}
?>
```

Khi upload file, cần kiểm tra:

```text
File upload security
├── Kích thước file
├── Đuôi file
├── MIME type thật
├── Tên file
├── Quyền ghi thư mục uploads
└── Không cho chạy file PHP trong thư mục upload
```

---

## 8. `$_ENV`

`$_ENV` chứa biến môi trường của ứng dụng hoặc server.
![[Pasted image 20260509172031.png|582]]

> EVN: nằm trên server--> là cấu hình do admin thiết lập
> GET: là lấy dữ liệu từ người dùng--> nằm trên trình duyệt của client

Ví dụ:

```php
<?php
echo $_ENV["APP_ENV"];
?>
```

Trong thực tế, biến môi trường thường ==dùng để lưu cấu hình:==

```text
$_ENV
├── APP_ENV
├── APP_DEBUG
├── DB_HOST
├── DB_NAME
├── DB_USER
├── DB_PASSWORD
├── API_KEY
└── SECRET_KEY
```

Ví dụ kết nối database dùng biến môi trường:

```php
<?php
$dbHost = $_ENV["DB_HOST"];
$dbName = $_ENV["DB_NAME"];
$dbUser = $_ENV["DB_USER"];
$dbPass = $_ENV["DB_PASSWORD"];
?>
```

Trong project hiện đại, đặc biệt với Laravel hoặc Docker, ==cấu hình thường được đặt trong file `.env`.==

Ví dụ `.env`:

```env
APP_ENV=local
DB_HOST=localhost
DB_NAME=my_app
DB_USER=root
DB_PASSWORD=secret
```

Không nên commit file `.env` thật lên GitHub nếu chứa secret.

---

## 9. `$GLOBALS`

`$GLOBALS` ==chứa toàn bộ biến global hiện có trong PHP==.

Ví dụ:

```php
<?php
$name = "Tài";

function showName() {
    echo $GLOBALS["name"];
}

showName();
?>
```

Kết quả:

```text
Tài
```

Tuy nhiên, trong code thực tế nên hạn chế dùng `$GLOBALS`, vì nó làm chương trình khó kiểm soát.

Không tốt:

```php
<?php
$tax = 0.1;

function calculateTotal($price) {
    return $price + ($price * $GLOBALS["tax"]);
}
?>
```

Tốt hơn:

```php
<?php
function calculateTotal($price, $tax) {
    return $price + ($price * $tax);
}

echo calculateTotal(100000, 0.1);
?>
```

Lý do: truyền dữ liệu qua parameter rõ ràng hơn, dễ test hơn, ít lỗi ngầm hơn.

---

## Bảng nhớ nhanh

```text
Superglobals
├── $_GET
│   └── Lấy dữ liệu từ URL
│
├── $_POST
│   └── Lấy dữ liệu từ form POST
│
├── $_REQUEST
│   └── Lấy dữ liệu tổng hợp từ GET / POST / COOKIE
│
├── $_SERVER
│   └── Lấy thông tin request và server
│
├── $_SESSION
│   └── Lưu dữ liệu phiên đăng nhập trên server
│
├── $_COOKIE
│   └── Lấy dữ liệu cookie từ trình duyệt
│
├── $_FILES
│   └── Xử lý file upload
│
├── $_ENV
│   └── Lấy biến môi trường
│
└── $GLOBALS
    └── Truy cập biến global
```

## Ví dụ tổng hợp nhỏ

File `index.php`:

```php
<?php
session_start();

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $name = $_POST["name"];

    $_SESSION["name"] = $name;

    setcookie("last_name", $name, time() + 3600);

    echo "Xin chào, " . htmlspecialchars($name);
}
?>

<form method="POST">
    <input type="text" name="name" placeholder="Nhập tên">
    <button type="submit">Gửi</button>
</form>
```

Trong ví dụ này:

```text
Ví dụ dùng Superglobals
├── $_SERVER["REQUEST_METHOD"]
│   └── Kiểm tra request có phải POST không
├── $_POST["name"]
│   └── Lấy dữ liệu từ form
├── $_SESSION["name"]
│   └── Lưu tên vào session
└── $_COOKIE
    └── Có thể đọc lại cookie ở request sau
```

## Quy tắc an toàn cần nhớ

Dữ liệu từ `$_GET`, `$_POST`, `$_REQUEST`, `$_COOKIE`, `$_FILES` đều nên được xem là **không đáng tin cậy** vì đến từ người dùng hoặc trình duyệt.

```text
Khi dùng Superglobals
├── Kiểm tra tồn tại bằng isset()
├── Validate dữ liệu
├── Escape khi in ra HTML bằng htmlspecialchars()
├── Dùng prepared statement khi đưa vào SQL
├── Không tin tưởng tên file upload
├── Không lưu password/token thô trong cookie
└── Hạn chế dùng $_REQUEST và $GLOBALS
```

Ví dụ xử lý an toàn hơn:

```php
<?php
$name = $_POST["name"] ?? "";

$name = trim($name);

if ($name === "") {
    echo "Tên không được để trống";
    exit;
}

echo htmlspecialchars($name);
?>
```