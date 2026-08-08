PHP là ngôn ngữ lập trình phía server, thường dùng để xây dựng website động, API, hệ thống quản trị, thương mại điện tử và backend cho web app. WordPress, Laravel, Symfony, Magento đều dùng PHP.
![[Pasted image 20260430044518.png|588]]

## 1. PHP dùng để làm gì?

PHP chạy trên server. Trình duyệt gửi request đến server, PHP xử lý dữ liệu, truy vấn database nếu cần, rồi trả về HTML, JSON hoặc nội dung khác cho trình duyệt.

Ví dụ luồng cơ bản:

```text
Browser → Server → PHP xử lý → Database nếu cần → Server trả kết quả → Browser
```

Ví dụ một file PHP đơn giản:

```php
<?php
echo "Xin chào, PHP!";
?>
```

Khi server chạy file này, kết quả trả về trình duyệt là:

```text
Xin chào, PHP!
```

## 2. PHP khác HTML, CSS, JavaScript thế nào?

HTML mô tả cấu trúc trang. CSS định dạng giao diện. JavaScript thường chạy trên trình duyệt để tạo tương tác. PHP chạy trên server để xử lý logic, dữ liệu, đăng nhập, giỏ hàng, thanh toán, API, kết nối database. [^1]

Ví dụ:

```php
<?php
$name = "Tài";
echo "<h1>Xin chào, $name</h1>";
?>
```

PHP tạo ra HTML động. Trình duyệt cuối cùng chỉ nhận được:

```html
<h1>Xin chào, Tài</h1>
```

## 3. Cú pháp PHP cơ bản

PHP code thường nằm trong cặp thẻ: 

```php
<?php
// code PHP ở đây
?>
```

Biến trong PHP bắt đầu bằng dấu `$`:

```php
<?php
$name = "Tài";
$age = 22;

echo $name;
echo $age;
?>
```

PHP không cần khai báo kiểu dữ liệu cố định như Java hay C, nhưng giá trị vẫn có kiểu.

Các kiểu dữ liệu hay gặp:

```php
<?php
$name = "Tài";       // string
$age = 22;           // integer
$price = 19.99;      // float
$isAdmin = true;     // boolean
$items = ["PHP", "Docker", "MySQL"]; // array
?>
```

## 4. Điều kiện `if`

Dùng khi chương trình cần rẽ nhánh.

```php
<?php
$score = 8;

if ($score >= 5) {
    echo "Đậu";
} else {
    echo "Rớt";
}
?>
```

PHP sẽ kiểm tra điều kiện `$score >= 5`. Nếu đúng, in `"Đậu"`. Nếu sai, in `"Rớt"`.

## 5. Vòng lặp

Dùng khi muốn lặp lại một hành động.

```php
<?php
for ($i = 1; $i <= 5; $i++) {
    echo "Lần thứ $i <br>";
}
?>
```

Kết quả:

```text
Lần thứ 1
Lần thứ 2
Lần thứ 3
Lần thứ 4
Lần thứ 5
```

Với mảng, thường dùng `foreach`:

```php
<?php
$languages = ["PHP", "JavaScript", "Python"];

foreach ($languages as $language) {
    echo $language . "<br>";
}
?>
```

## 6. Function trong PHP

Function giúp gom logic thành một khối có thể tái sử dụng.

```php
<?php
function sayHello($name) {
    return "Xin chào, " . $name;
}

echo sayHello("Tài");
?>
```

Kết quả:

```text
Xin chào, Tài
```

## 7. Form HTML và PHP

<u>Một ứng dụng web thật thường nhận dữ liệu từ người dùng</u>. Ví dụ form gửi tên người dùng đến PHP.

File `index.php`:

```php
<form method="POST">
    <input type="text" name="username" placeholder="Nhập tên">
    <button type="submit">Gửi</button>
</form>

<?php
if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $username = $_POST["username"];
    echo "Xin chào, " . htmlspecialchars($username);
}
?>
```

Ở đây:

```php
$_POST["username"]
```

lấy dữ liệu từ form.

Dòng này:

```php
htmlspecialchars($username)
```

giúp tránh lỗi bảo mật XSS khi in dữ liệu người dùng ra HTML.

## 8. PHP và database

<u>PHP thường đi cùng MySQL hoặc PostgreSQL</u>. Ví dụ với MySQL, bạn có thể lưu user, sản phẩm, đơn hàng, bài viết.

Ví dụ kết nối <u>MySQL bằng PDO:</u>

```php
<?php
$pdo = new PDO(
    "mysql:host=localhost;dbname=my_app;charset=utf8mb4",
    "root",
    "password"
);

$stmt = $pdo->query("SELECT * FROM users");

$users = $stmt->fetchAll(PDO::FETCH_ASSOC);

foreach ($users as $user) {
    echo $user["name"] . "<br>";
}
?>
```

Trong thực tế, nên dùng <u>prepared statement để tránh SQL injection</u>:

```php
<?php
$email = $_POST["email"];

$stmt = $pdo->prepare("SELECT * FROM users WHERE email = ?");
$stmt->execute([$email]);

$user = $stmt->fetch(PDO::FETCH_ASSOC);
?>
```

## 9. Một ứng dụng PHP nhỏ

Ví dụ tạo danh sách việc cần làm đơn giản, chưa dùng database:

```php
<?php
$todos = [
    "Học cú pháp PHP",
    "Hiểu form POST",
    "Kết nối database",
    "Học Laravel"
];
?>

<h1>Todo List</h1>

<ul>
    <?php foreach ($todos as $todo): ?>
        <li><?= htmlspecialchars($todo) ?></li>
    <?php endforeach; ?>
</ul>
```

Cú pháp này:

```php
<?= $value ?>
```

là cách viết ngắn của:

```php
<?php echo $value; ?>
```

## 10. Những keyword PHP cần nắm đầu tiên

```text
PHP nền tảng
├── echo
├── variable: $name
├── string, integer, float, boolean, array
├── if / else
├── for / while / foreach
├── function
├── include / require
├── $_GET
├── $_POST
├── $_SERVER
├── session
├── cookie
├── PDO
├── prepared statement
├── Composer
├── namespace
├── class / object
├── trait / interface
├── autoload
└── framework: Laravel hoặc Symfony
```

## 11. Lộ trình học PHP hợp lý

Giai đoạn đầu nên học theo thứ tự này:

```text
1. Cú pháp cơ bản
2. Biến, kiểu dữ liệu, toán tử
3. Điều kiện và vòng lặp
4. Function
5. Array
6. Form HTML + GET/POST
7. Session và cookie
8. File upload
9. Kết nối MySQL bằng PDO
10. Bảo mật cơ bản: XSS, SQL Injection, password_hash
11. OOP trong PHP
12. Composer
13. Laravel
```

Laravel chỉ nên học sau khi bạn đã hiểu PHP cơ bản. Nếu học Laravel quá sớm, bạn dễ biết cách dùng framework nhưng không hiểu PHP đang làm gì bên dưới.

## 12. Bài tập đầu tiên

Tạo file `index.php`:

```php
<?php
$name = "Tài";
$skills = ["PHP", "HTML", "CSS", "MySQL"];
?>

<h1>Xin chào, <?= htmlspecialchars($name) ?></h1>

<h2>Kỹ năng đang học</h2>

<ul>
    <?php foreach ($skills as $skill): ?>
        <li><?= htmlspecialchars($skill) ?></li>
    <?php endforeach; ?>
</ul>
```

Mục tiêu của bài này là hiểu ba ý chính: PHP có thể nhúng vào HTML, biến PHP bắt đầu bằng `$`, và `foreach` dùng để render danh sách động.

[^1]: -Trình duyệt: javascript
	-Server: php
	-landing page: H
