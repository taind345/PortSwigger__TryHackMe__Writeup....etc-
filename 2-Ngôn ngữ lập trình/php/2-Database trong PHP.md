[[Minh_hoa_prepare_sql]]

Trong web app PHP, database là nơi lưu dữ liệu bền vững: user, sản phẩm, đơn hàng, bài viết, lịch sử giao dịch, quyền truy cập. PHP không tự lưu dữ liệu lâu dài; PHP thường **nhận request → xử lý logic → đọc/ghi database → trả response**.

```text
PHP + Database
├── PHP
│   └── Xử lý logic ứng dụng
│
├── Database
│   └── Lưu dữ liệu lâu dài
│
└── PDO
    └── Cầu nối giữa PHP và database
```

---

## 1. MySQL

**MySQL** là hệ quản trị cơ sở dữ liệu quan hệ rất phổ biến trong PHP. WordPress, Laravel app nhỏ/vừa, hệ thống CMS, website bán hàng thường dùng MySQL hoặc MariaDB.

Dữ liệu trong MySQL được lưu theo bảng.

Ví dụ bảng `users`:

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Dữ liệu mẫu:

```text
users
├── id
├── name
├── email
├── password
└── created_at
```

Ví dụ một dòng dữ liệu:

```text
id: 1
name: Tài
email: tai@example.com
password: hashed_password_here
created_at: 2026-04-30 10:00:00
```

---

## 2. PostgreSQL

**PostgreSQL** cũng là database quan hệ, thường được đánh giá cao về độ chặt chẽ, tính năng nâng cao, transaction, JSON, constraint, indexing và các hệ thống cần độ tin cậy cao.

Với PHP, PostgreSQL vẫn có thể dùng qua PDO giống MySQL.

Ví dụ bảng `users` trong PostgreSQL:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Khác biệt nhỏ:

```text
MySQL
└── id INT AUTO_INCREMENT PRIMARY KEY

PostgreSQL
└── id SERIAL PRIMARY KEY
```

Nếu mới học PHP database, bạn nên bắt đầu với **MySQL + PDO**, sau đó học PostgreSQL nếu cần.

---

## 3. PDO

**PDO**, viết đầy đủ là **PHP Data Objects**, là cách hiện đại để PHP làm việc với database.

==PDO giúp PHP kết nối với nhiều loại databas==e khác nhau:

```text
PDO
├── MySQL
├── PostgreSQL
├── SQLite
├── SQL Server
└── Một số database khác
```

Ưu điểm lớn của PDO:

```text
PDO
├── Dùng được với nhiều database
├── Hỗ trợ prepared statement
├── Giảm nguy cơ SQL Injection
├── Có chế độ bắt lỗi bằng exception
└── Code sạch hơn mysqli truyền thống
```

---

## 4. Connection string

**Connection string** là chuỗi cấu hình để PHP biết cần kết nối đến database nào.

Với MySQL:

```php
<?php
$pdo = new PDO(
    "mysql:host=localhost;dbname=my_app;charset=utf8mb4",
    "root",
    "password"
);
?>
```

Phân tích:

```text
mysql
├── Loại database

host=localhost
├── Database server nằm trên máy hiện tại

dbname=my_app
├── Tên database

charset=utf8mb4
├── Bộ mã ký tự, nên dùng utf8mb4

root
├── Username database

password
└── Password database
```

Với PostgreSQL:

```php
<?php
$pdo = new PDO(
    "pgsql:host=localhost;port=5432;dbname=my_app",
    "postgres",
    "password"
);
?>
```

Cách viết tốt hơn là bật exception mode:

```php
<?php
$pdo = new PDO(
    "mysql:host=localhost;dbname=my_app;charset=utf8mb4",
    "root",
    "password",
    [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
    ]
);
?>
```

Nên tách kết nối database vào file riêng.

File `database.php`:

```php
<?php

$host = "localhost";
$dbname = "my_app";
$username = "root";
$password = "password";

$pdo = new PDO(
    "mysql:host=$host;dbname=$dbname;charset=utf8mb4",
    $username,
    $password,
    [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
    ]
);
```

File khác chỉ cần:

```php
<?php

require "database.php";
```

---

## 5. `query`

`query()` dùng để chạy SQL trực tiếp.

Ví dụ đọc toàn bộ user:

```php
<?php

require "database.php";

$stmt = $pdo->query("SELECT * FROM users");

$users = $stmt->fetchAll();

foreach ($users as $user) {
    echo $user["name"] . "<br>";
}
```

`query()` phù hợp khi câu SQL **không chứa dữ liệu từ người dùng**.

Ví dụ an toàn:

```php
$pdo->query("SELECT * FROM users");
```

Không nên làm thế này:

```php
$email = $_GET["email"];

$pdo->query("SELECT * FROM users WHERE email = '$email'");
```

Lý do: dễ bị **SQL Injection**.

---

## 6. `prepare`

`prepare()` dùng để chuẩn bị câu SQL có placeholder. Đây là cách nên dùng khi câu SQL có dữ liệu từ người dùng.

Ví dụ:

```php
<?php

require "database.php";

$email = $_POST["email"];

$stmt = $pdo->prepare("SELECT * FROM users WHERE email = ?");
```

Dấu `?` là placeholder. Nó đại diện cho giá trị sẽ được đưa vào sau.

Có hai kiểu placeholder phổ biến.

Kiểu `?`:

```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = ?");
```

Kiểu named placeholder:

```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = :email");
```

Named placeholder thường dễ đọc hơn trong câu SQL dài.

---

## 7. `execute`

`execute()` dùng để chạy câu SQL đã được `prepare()`.  
![[Pasted image 20260509141612.png]]

Ví dụ dùng `?`:

```php
<?php

require "database.php";

$email = $_POST["email"];

$stmt = $pdo->prepare("SELECT * FROM users WHERE email = ?");
$stmt->execute([$email]);

$user = $stmt->fetch();
```

Ví dụ dùng named placeholder:

```php
<?php

require "database.php";

$email = $_POST["email"];

$stmt = $pdo->prepare("SELECT * FROM users WHERE email = :email");
$stmt->execute([
    "email" => $email
]);

$user = $stmt->fetch();
```

Luồng chuẩn:

```text
prepare()
├── Chuẩn bị SQL

execute()
├── Truyền dữ liệu thật vào placeholder

fetch() / fetchAll()
└── Lấy kết quả
```

---

## 8. `fetch`

`fetch()` lấy **một dòng dữ liệu** từ kết quả query.

Ví dụ tìm user theo email:

```php
<?php

require "database.php";

$email = "tai@example.com";

$stmt = $pdo->prepare("SELECT * FROM users WHERE email = :email");
$stmt->execute([
    "email" => $email
]);

$user = $stmt->fetch();

if ($user) {
    echo $user["name"];
} else {
    echo "Không tìm thấy user";
}
```

`fetch()` phù hợp khi bạn mong đợi một kết quả duy nhất:

```text
fetch()
├── Tìm user theo id
├── Tìm user theo email
├── Lấy một bài viết theo slug
└── Lấy một đơn hàng theo mã đơn
```

---

## 9. `fetchAll`

`fetchAll()` lấy **nhiều dòng dữ liệu**.

Ví dụ lấy toàn bộ users:

```php
<?php

require "database.php";

$stmt = $pdo->query("SELECT * FROM users ORDER BY id DESC");

$users = $stmt->fetchAll();

foreach ($users as $user) {
    echo $user["name"] . " - " . $user["email"] . "<br>";
}
```

`fetchAll()` phù hợp khi kết quả là danh sách:

```text
fetchAll()
├── Danh sách user
├── Danh sách sản phẩm
├── Danh sách bài viết
├── Danh sách đơn hàng
└── Kết quả tìm kiếm
```

Không nên dùng `fetchAll()` nếu bảng có hàng triệu dòng mà không giới hạn, vì nó đưa toàn bộ kết quả vào memory.

Nên dùng `LIMIT`:

```php
$stmt = $pdo->query("SELECT * FROM users ORDER BY id DESC LIMIT 20");
$users = $stmt->fetchAll();
```

---

# 10. CRUD

CRUD là bốn thao tác nền tảng với database:

```text
CRUD
├── CREATE
│   └── Thêm dữ liệu
├── READ
│   └── Đọc dữ liệu
├── UPDATE
│   └── Cập nhật dữ liệu
└── DELETE
    └── Xóa dữ liệu
```

Giả sử có bảng `users`:

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 10.1 CREATE

CREATE là thêm dữ liệu mới.

Ví dụ thêm user:

```php
<?php

require "database.php";

$name = $_POST["name"];
$email = $_POST["email"];
$password = password_hash($_POST["password"], PASSWORD_DEFAULT);

$stmt = $pdo->prepare("
    INSERT INTO users (name, email, password)
    VALUES (:name, :email, :password)
");

$stmt->execute([
    "name" => $name,
    "email" => $email,
    "password" => $password
]);

echo "Tạo user thành công";
```

SQL chính:

```sql
INSERT INTO users (name, email, password)
VALUES (:name, :email, :password)
```

Không lưu password thô. Luôn dùng:

```php
password_hash($password, PASSWORD_DEFAULT);
```

---

## 10.2 READ

READ là đọc dữ liệu.

Đọc nhiều user:

```php
<?php

require "database.php";

$stmt = $pdo->query("SELECT id, name, email, created_at FROM users");

$users = $stmt->fetchAll();

foreach ($users as $user) {
    echo $user["name"] . " - " . $user["email"] . "<br>";
}
```

Đọc một user theo id:

```php
<?php

require "database.php";

$id = $_GET["id"];

$stmt = $pdo->prepare("SELECT id, name, email FROM users WHERE id = :id");
$stmt->execute([
    "id" => $id
]);

$user = $stmt->fetch();

if (!$user) {
    echo "Không tìm thấy user";
    exit;
}

echo $user["name"];
```

SQL chính:

```sql
SELECT id, name, email FROM users WHERE id = :id
```

---

## 10.3 UPDATE

UPDATE là cập nhật dữ liệu đã có.

```php
<?php

require "database.php";

$id = $_POST["id"];
$name = $_POST["name"];
$email = $_POST["email"];

$stmt = $pdo->prepare("
    UPDATE users
    SET name = :name, email = :email
    WHERE id = :id
");

$stmt->execute([
    "id" => $id,
    "name" => $name,
    "email" => $email
]);

echo "Cập nhật user thành công";
```

SQL chính:

```sql
UPDATE users
SET name = :name, email = :email
WHERE id = :id
```

Phần `WHERE` rất quan trọng. Nếu quên `WHERE`, bạn có thể cập nhật toàn bộ bảng.

Nguy hiểm:

```sql
UPDATE users SET name = 'Unknown';
```

Câu này đổi tên tất cả user.

---

## 10.4 DELETE

DELETE là xóa dữ liệu.

```php
<?php

require "database.php";

$id = $_POST["id"];

$stmt = $pdo->prepare("DELETE FROM users WHERE id = :id");

$stmt->execute([
    "id" => $id
]);

echo "Xóa user thành công";
```

SQL chính:

```sql
DELETE FROM users WHERE id = :id
```

Cũng như `UPDATE`, `DELETE` phải có `WHERE`.

Nguy hiểm:

```sql
DELETE FROM users;
```

Câu này xóa toàn bộ bảng `users`.

Trong app thật, nhiều hệ thống không xóa cứng mà dùng **soft delete**:

```sql
UPDATE users
SET deleted_at = NOW()
WHERE id = :id
```

---

# 11. Transaction

Transaction dùng khi nhiều thao tác database phải thành công hoặc thất bại cùng nhau.

Ví dụ chuyển tiền:

```text
Chuyển 100.000 từ A sang B
├── Trừ 100.000 khỏi tài khoản A
├── Cộng 100.000 vào tài khoản B
└── Ghi lịch sử giao dịch
```

Nếu trừ tiền A thành công nhưng cộng tiền B thất bại, dữ liệu sẽ sai. Transaction giải quyết vấn đề đó.

```text
Transaction
├── beginTransaction()
│   └── Bắt đầu giao dịch
├── commit()
│   └── Xác nhận toàn bộ thay đổi
└── rollBack()
    └── Hủy toàn bộ thay đổi nếu có lỗi
```

Ví dụ:

```php
<?php

require "database.php";

try {
    $pdo->beginTransaction();

    $stmt = $pdo->prepare("
        UPDATE accounts
        SET balance = balance - :amount
        WHERE id = :from_id
    ");
    $stmt->execute([
        "amount" => 100000,
        "from_id" => 1
    ]);

    $stmt = $pdo->prepare("
        UPDATE accounts
        SET balance = balance + :amount
        WHERE id = :to_id
    ");
    $stmt->execute([
        "amount" => 100000,
        "to_id" => 2
    ]);

    $stmt = $pdo->prepare("
        INSERT INTO transactions (from_id, to_id, amount)
        VALUES (:from_id, :to_id, :amount)
    ");
    $stmt->execute([
        "from_id" => 1,
        "to_id" => 2,
        "amount" => 100000
    ]);

    $pdo->commit();

    echo "Chuyển tiền thành công";
} catch (Exception $e) {
    $pdo->rollBack();

    echo "Có lỗi xảy ra: " . $e->getMessage();
}
```

Nếu một bước lỗi, toàn bộ thay đổi bị hủy.

Transaction thường dùng cho:

```text
Transaction
├── Thanh toán
├── Chuyển tiền
├── Tạo đơn hàng + chi tiết đơn hàng
├── Trừ tồn kho
├── Ghi log giao dịch
└── Các thao tác cần toàn vẹn dữ liệu
```

---

# 12. Ví dụ thực tế: đăng ký user

File `register.php`:

```php
<?php

require "database.php";

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $name = trim($_POST["name"] ?? "");
    $email = trim($_POST["email"] ?? "");
    $password = $_POST["password"] ?? "";

    if ($name === "" || $email === "" || $password === "") {
        echo "Vui lòng nhập đủ thông tin";
        exit;
    }

    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        echo "Email không hợp lệ";
        exit;
    }

    $hashedPassword = password_hash($password, PASSWORD_DEFAULT);

    $stmt = $pdo->prepare("
        INSERT INTO users (name, email, password)
        VALUES (:name, :email, :password)
    ");

    try {
        $stmt->execute([
            "name" => $name,
            "email" => $email,
            "password" => $hashedPassword
        ]);

        echo "Đăng ký thành công";
    } catch (PDOException $e) {
        echo "Email có thể đã tồn tại";
    }
}
?>

<form method="POST">
    <input type="text" name="name" placeholder="Tên">
    <input type="email" name="email" placeholder="Email">
    <input type="password" name="password" placeholder="Mật khẩu">
    <button type="submit">Đăng ký</button>
</form>
```

Trong ví dụ này:

```text
register.php
├── Nhận dữ liệu từ $_POST
├── Validate dữ liệu
├── Hash password
├── prepare INSERT SQL
├── execute với dữ liệu thật
└── Lưu user vào database
```

---

# 13. Ví dụ thực tế: đăng nhập user

```php
<?php

require "database.php";

session_start();

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $email = trim($_POST["email"] ?? "");
    $password = $_POST["password"] ?? "";

    $stmt = $pdo->prepare("
        SELECT id, name, email, password
        FROM users
        WHERE email = :email
    ");

    $stmt->execute([
        "email" => $email
    ]);

    $user = $stmt->fetch();

    if (!$user) {
        echo "Sai email hoặc mật khẩu";
        exit;
    }

    if (!password_verify($password, $user["password"])) {
        echo "Sai email hoặc mật khẩu";
        exit;
    }

    session_regenerate_id(true);

    $_SESSION["user_id"] = $user["id"];
    $_SESSION["user_name"] = $user["name"];

    echo "Đăng nhập thành công";
}
?>

<form method="POST">
    <input type="email" name="email" placeholder="Email">
    <input type="password" name="password" placeholder="Mật khẩu">
    <button type="submit">Đăng nhập</button>
</form>
```

Luồng xử lý:

```text
Login
├── Người dùng nhập email/password
├── SELECT user theo email
├── Nếu không có user
│   └── Báo lỗi
├── Nếu có user
│   └── password_verify()
├── Nếu password đúng
│   ├── session_regenerate_id(true)
│   ├── Lưu user_id vào $_SESSION
│   └── Đăng nhập thành công
└── Nếu password sai
    └── Báo lỗi
```

---

# 14. Bảng nhớ nhanh

```text
Database trong PHP
├── MySQL
│   └── Database phổ biến, dễ bắt đầu
│
├── PostgreSQL
│   └── Database mạnh, chặt chẽ, nhiều tính năng nâng cao
│
├── PDO
│   └── Cầu nối PHP với database
│
├── connection string
│   └── Chuỗi cấu hình kết nối database
│
├── query
│   └── Chạy SQL trực tiếp, chỉ nên dùng khi không có input người dùng
│
├── prepare
│   └── Chuẩn bị SQL có placeholder
│
├── execute
│   └── Chạy SQL đã prepare với dữ liệu thật
│
├── fetch
│   └── Lấy một dòng kết quả
│
├── fetchAll
│   └── Lấy nhiều dòng kết quả
│
├── transaction
│   └── Gom nhiều thao tác thành một đơn vị an toàn
│
└── CRUD
    ├── CREATE → INSERT
    ├── READ   → SELECT
    ├── UPDATE → UPDATE
    └── DELETE → DELETE
```

---

# 15. Quy tắc an toàn cần nhớ

```text
Khi dùng PHP + Database
├── Luôn dùng PDO hoặc ORM đáng tin cậy
├── Dùng prepare + execute cho dữ liệu từ user
├── Không nối chuỗi trực tiếp vào SQL
├── Luôn hash password bằng password_hash()
├── Kiểm tra password bằng password_verify()
├── Validate dữ liệu trước khi ghi database
├── Không hiển thị lỗi database thô cho user production
├── Cẩn thận với UPDATE / DELETE không có WHERE
├── Dùng transaction cho thao tác nhiều bước
└── Không commit password database thật lên GitHub
```

Mẫu quan trọng nhất cần nhớ:

```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = :email");

$stmt->execute([
    "email" => $email
]);

$user = $stmt->fetch();
```

Đây là cấu trúc nền tảng của PHP database hiện đại: **PDO + prepared statement + fetch/fetchAll**.