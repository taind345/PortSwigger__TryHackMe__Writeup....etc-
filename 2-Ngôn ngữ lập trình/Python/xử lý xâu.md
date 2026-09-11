
## Các hàm xử lý xâu thông dụng trong Python

### 1. Khởi tạo và truy cập

```python
s = "Hello World"
print(s[0])      # 'H' - truy cập bằng chỉ số
print(s[-1])     # 'd' - phần tử cuối
print(s[2:5])    # 'llo' - cắt chuỗi (slice)
print(len(s))    # 11 - độ dài chuỗi
```

### 2. Tìm kiếm và đếm

```python
s = "hello world hello"

# Tìm vị trí
print(s.find("world"))     # 6 - vị trí đầu tiên
print(s.find("python"))    # -1 - không tìm thấy
print(s.index("world"))    # 6 (giống find nhưng lỗi nếu không tìm thấy)

# Đếm số lần xuất hiện
print(s.count("hello"))    # 2
print(s.count("l"))        # 4

# Kiểm tra tồn tại
print("world" in s)        # True
print("python" in s)       # False
```

### 3. Biến đổi chuỗi

```python
s = "Hello World"

# Chữ hoa/chữ thường
print(s.upper())           # "HELLO WORLD"
print(s.lower())           # "hello world"
print(s.title())           # "Hello World" (viết hoa đầu mỗi từ)
print(s.capitalize())      # "Hello world" (viết hoa chữ đầu)
print(s.swapcase())        # "hELLO wORLD" (đảo hoa-thường)

# Căn chỉnh
print(s.center(20, '*'))   # "****Hello World*****"
print(s.ljust(15, '-'))    # "Hello World----"
print(s.rjust(15, '-'))    # "----Hello World"

# Thay thế
print(s.replace("World", "Python"))  # "Hello Python"
print(s.replace("l", "L", 2))        # "HeLLo World" - thay 2 lần đầu
```

### 4. Kiểm tra loại ký tự

```python
s1 = "12345"
s2 = "Hello123"
s3 = "HELLO"
s4 = "hello"
s5 = " \t\n"
s6 = "Hello World"

print(s1.isdigit())        # True - toàn chữ số
print(s1.isnumeric())      # True - toàn số
print(s2.isalnum())        # True - chữ và số
print(s3.isupper())        # True - toàn chữ hoa
print(s4.islower())        # True - toàn chữ thường
print(s5.isspace())        # True - toàn khoảng trắng
print(s6.istitle())        # True - dạng tiêu đề (viết hoa đầu từ)
print("abc".isalpha())     # True - toàn chữ cái
```

### 5. Xử lý khoảng trắng

```python
s = "  hello world  \t\n"
print(s.strip())           # "hello world" - xóa cả 2 đầu
print(s.lstrip())          # "hello world  \t\n" - xóa đầu trái
print(s.rstrip())          # "  hello world" - xóa đầu phải

# Xóa ký tự cụ thể
s2 = "***hello***"
print(s2.strip('*'))       # "hello"
```

### 6. Nối và tách chuỗi

```python
# Nối chuỗi
parts = ["Hello", "World", "Python"]
print(" ".join(parts))     # "Hello World Python"
print("-".join(parts))     # "Hello-World-Python"

# Tách chuỗi
s = "apple,banana,orange"
print(s.split(","))        # ['apple', 'banana', 'orange']
print(s.split(",", 1))     # ['apple', 'banana,orange'] - tách 1 lần

s2 = "line1\nline2\nline3"
print(s2.splitlines())     # ['line1', 'line2', 'line3']

# Tách với partition/rpartition
s3 = "hello.world.py"
print(s3.partition("."))   # ('hello', '.', 'world.py') - tách lần đầu
print(s3.rpartition("."))  # ('hello.world', '.', 'py') - tách lần cuối
```

### 7. Định dạng chuỗi

```python
name = "Alice"
age = 25

# f-string (Python 3.6+)
print(f"My name is {name}, age {age}")

# format()
print("My name is {}, age {}".format(name, age))
print("My name is {1}, age {0}".format(age, name))
print("PI: {:.2f}".format(3.14159))  # "PI: 3.14"

# % operator
print("My name is %s, age %d" % (name, age))
```

### 8. Kiểm tra bắt đầu/kết thúc

```python
s = "hello world"
print(s.startswith("hello"))   # True
print(s.startswith("he"))      # True
print(s.endswith("world"))     # True
print(s.endswith("ld"))        # True

# Kiểm tra với tuple
print(s.endswith(("world", "python")))  # True (có 1 cái đúng)
```

### 9. Mã hóa và giải mã

```python
s = "hello"

# Mã hóa
encoded = s.encode('utf-8')    # b'hello'
print(encoded)

# Giải mã
decoded = encoded.decode('utf-8')  # 'hello'
print(decoded)
```

### 10. Các hàm hữu ích khác

```python
# Điền số 0
n = "42"
print(n.zfill(5))              # "00042"

# Tìm tất cả vị trí
import re
s = "hello world hello"
print([m.start() for m in re.finditer('hello', s)])  # [0, 12]

# Đảo ngược chuỗi
s = "hello"
print(s[::-1])                 # "olleh"

# Kiểm tra các ký tự
print("abc123".isascii())      # True
print("hello".isprintable())   # True
print("\t".isprintable())      # False
```

### 11. Ví dụ tổng hợp

```python
# Xử lý tên người dùng
def process_username(username):
    username = username.strip()  # Xóa khoảng trắng thừa
    username = username.lower()  # Chuyển chữ thường
    username = username.replace(" ", "_")  # Thay space bằng _
    return username

print(process_username("  John Doe  "))  # "john_doe"

# Đếm từ trong câu
def count_words(text):
    words = text.split()
    return len(words)

print(count_words("Hello world from Python"))  # 4
```

### 12. Lưu ý quan trọng

```python
# Chuỗi trong Python là bất biến (immutable)
s = "hello"
# s[0] = 'H'  # ❌ Lỗi: 'str' object does not support item assignment

# Muốn thay đổi, tạo chuỗi mới
s = 'H' + s[1:]  # ✅ "Hello"

# So sánh chuỗi
print("abc" == "ABC")      # False (phân biệt hoa thường)
print("abc".lower() == "ABC".lower())  # True

# Duyệt chuỗi
s = "hello"
for char in s:
    print(char)  # h e l l o

for i, char in enumerate(s):
    print(f"Index {i}: {char}")
```

## Tóm tắt các hàm quan trọng nhất

- `len()`, `find()`, `count()`, `in`
- `upper()`, `lower()`, `title()`
- `strip()`, `lstrip()`, `rstrip()`
- `split()`, `join()`
- `startswith()`, `endswith()`
- `isdigit()`, `isalpha()`, `isalnum()`
- `replace()`, `format()`, f-string
```