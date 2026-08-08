[[anonymous function]]

Function là một khối code được đặt tên để **tái sử dụng logic**. Thay vì viết đi viết lại cùng một đoạn xử lý, ta gom nó vào function rồi gọi khi cần.

Ví dụ cơ bản:

```php
<?php
function sayHello() {
    echo "Xin chào!";
}

sayHello();
?>
```

Kết quả:

```text
Xin chào!
```

---

## 1. `function`

`function` là từ khóa dùng để khai báo hàm.

Cú pháp:

```php
<?php
function tenHam() {
    // code xử lý
}
?>
```

Ví dụ:

```php
<?php
function showMessage() {
    echo "Đây là function đầu tiên của tôi";
}

showMessage();
?>
```

Tên function nên mô tả hành động, ví dụ:

```php
calculateTotal()
getUserName()
sendEmail()
validatePassword()
```

---

## 2. Parameter

Parameter là **biến đầu vào được khai báo khi định nghĩa function**.

Ví dụ:

```php
<?php
function sayHello($name) {
    echo "Xin chào, $name";
}
?>
```

Ở đây:

```php
$name
```

là **parameter**.

Nó giống như một “chỗ trống” để function nhận dữ liệu từ bên ngoài.

---

## 3. Argument

Argument là **giá trị thật được truyền vào khi gọi function**.

```php
<?php
function sayHello($name) {
    echo "Xin chào, $name";
}

sayHello("Tài");
?>
```

Ở đây:

```php
"Tài"
```

là **argument**.

Phân biệt ngắn gọn:

```text
Parameter → biến lúc khai báo hàm
Argument  → giá trị lúc gọi hàm
```

Ví dụ rõ hơn:

```php
<?php
function add($a, $b) {
    echo $a + $b;
}

add(3, 5);
?>
```

Trong ví dụ này:

```text
$a, $b → parameter
3, 5   → argument
```

---

## 4. `return`

`return` dùng để **trả kết quả từ function ra bên ngoài**.

Ví dụ không dùng `return`:

```php
<?php
function add($a, $b) {
    echo $a + $b;
}

add(3, 5);
?>
```

Hàm này chỉ in ra kết quả, nhưng không trả kết quả để dùng tiếp.

Ví dụ dùng `return`:

```php
<?php
function add($a, $b) {
    return $a + $b;
}

$result = add(3, 5);

echo $result;
?>
```

Kết quả:

```text
8
```

Lợi ích của `return` là bạn có thể dùng kết quả cho bước xử lý tiếp theo:

```php
<?php
function calculateTotal($price, $quantity) {
    return $price * $quantity;
}

$total = calculateTotal(100000, 3);
$finalPrice = $total + 20000;

echo $finalPrice;
?>
```

---

## 5. Default parameter

Default parameter là **giá trị mặc định của parameter** nếu người gọi hàm không truyền argument.

Ví dụ:

```php
<?php
function sayHello($name = "bạn") {
    echo "Xin chào, $name";
}

sayHello();
?>
```

Kết quả:

```text
Xin chào, bạn
```

Nếu truyền argument:

```php
<?php
sayHello("Tài");
?>
```

Kết quả:

```text
Xin chào, Tài
```

Ví dụ thực tế:

```php
<?php
function calculateShippingFee($city, $fee = 30000) {
    return "Phí ship đến $city là $fee VND";
}

echo calculateShippingFee("Hà Nội");
?>
```

---

## 6. Type hint

Type hint dùng để **khai báo kiểu dữ liệu mong muốn của parameter**.

Ví dụ:

```php
<?php
function add(int $a, int $b) {
    return $a + $b;
}

echo add(3, 5);
?>
```

Ở đây:

```php
int $a
int $b
```

nghĩa là `$a` và `$b` nên là số nguyên.

Các type hint phổ biến:

```text
int     → số nguyên
float   → số thực
string  → chuỗi
bool    → true / false
array   → mảng
object  → object
mixed   → nhiều kiểu dữ liệu
```

Ví dụ:

```php
<?php
function formatName(string $name) {
    return strtoupper($name);
}

echo formatName("tai");
?>
```

Kết quả:

```text
TAI
```

---

## 7. Return type

Return type dùng để **khai báo kiểu dữ liệu mà function sẽ trả về**.

Ví dụ:

```php
<?php
function add(int $a, int $b): int {
    return $a + $b;
}
?>
```

Đoạn này có nghĩa:

```text
$a phải là int
$b phải là int
hàm add trả về int
```

Ví dụ khác:

```php
<?php
function getUserName(): string {
    return "Tài";
}

function isAdult(int $age): bool {
    return $age >= 18;
}

function getNumbers(): array {
    return [1, 2, 3, 4, 5];
}
?>
```

Viết return type giúp code rõ ràng, dễ đọc và dễ bắt lỗi hơn.

---

## 8. Anonymous function

Anonymous function là **hàm không có tên**. Nó thường được gán vào biến hoặc truyền vào function khác.

Ví dụ gán vào biến:

```php
<?php
$sayHello = function ($name) {
    return "Xin chào, $name";
};

echo $sayHello("Tài");
?>
```

Kết quả:

```text
Xin chào, Tài
```

Anonymous function thường gặp khi xử lý array:

```php
<?php
$numbers = [1, 2, 3, 4, 5];

$squared = array_map(function ($number) {
    return $number * $number;
}, $numbers);

print_r($squared);
?>
```

Kết quả:

```text
Array
(
    [0] => 1
    [1] => 4
    [2] => 9
    [3] => 16
    [4] => 25
)
```

Ở đây, function không tên được truyền vào `array_map`.

---

## 9. Arrow function

Arrow function là cách viết ngắn hơn của anonymous function.

Cú pháp:

```php
fn($x) => $x * 2
```

Ví dụ:

```php
<?php
$numbers = [1, 2, 3, 4, 5];

$doubled = array_map(fn($number) => $number * 2, $numbers);

print_r($doubled);
?>
```

Kết quả:

```text
Array
(
    [0] => 2
    [1] => 4
    [2] => 6
    [3] => 8
    [4] => 10
)
```

Anonymous function dạng dài:

```php
function ($number) {
    return $number * 2;
}
```

Arrow function dạng ngắn:

```php
fn($number) => $number * 2
```

Arrow function phù hợp với logic ngắn, một dòng.

---

## 10. Built-in function

Built-in function là **hàm có sẵn trong PHP**. Bạn không cần tự viết lại.

Ví dụ với string:

```php
<?php
$name = "tai";

echo strlen($name);
echo strtoupper($name);
echo ucfirst($name);
?>
```

Một số built-in function phổ biến:

```text
String
├── strlen()
├── strtoupper()
├── strtolower()
├── trim()
├── str_replace()
└── explode()

Array
├── count()
├── array_push()
├── array_map()
├── array_filter()
├── array_reduce()
└── in_array()

Math
├── round()
├── ceil()
├── floor()
├── rand()
└── max()

Security
├── htmlspecialchars()
├── password_hash()
└── password_verify()

Database / PDO
├── prepare()
├── execute()
├── fetch()
└── fetchAll()
```

Ví dụ thực tế:

```php
<?php
$email = "  TEST@EMAIL.COM  ";

$cleanEmail = strtolower(trim($email));

echo $cleanEmail;
?>
```

Kết quả:

```text
test@email.com
```

---

## Ví dụ tổng hợp

```php
<?php
function calculateTotalPrice(
    float $price,
    int $quantity,
    float $taxRate = 0.1
): float {
    $subtotal = $price * $quantity;
    $tax = $subtotal * $taxRate;

    return $subtotal + $tax;
}

$total = calculateTotalPrice(100000, 2);

echo "Tổng tiền: " . number_format($total) . " VND";
?>
```

Phân tích:

```text
function              → khai báo hàm
calculateTotalPrice   → tên hàm
$price, $quantity     → parameter
100000, 2             → argument
$taxRate = 0.1        → default parameter
float, int            → type hint
: float               → return type
return                → trả kết quả
number_format()       → built-in function
```

Kết quả:

```text
Tổng tiền: 220,000 VND
```

## Bài tập nhỏ

Viết function tính điểm trung bình:

```php
<?php
function calculateAverage(float $math, float $english, float $science): float {
    return ($math + $english + $science) / 3;
}

$average = calculateAverage(8.5, 7.0, 9.0);

echo "Điểm trung bình: " . round($average, 2);
?>
```

Trong PHP thực tế, function là nền móng trước khi học OOP, Controller, Service, Laravel route handler và API logic.