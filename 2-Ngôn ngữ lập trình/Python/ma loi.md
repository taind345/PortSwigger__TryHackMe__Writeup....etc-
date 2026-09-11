Khi debug Python, bạn sẽ gặp một số **Exception (ngoại lệ)** rất phổ biến. Đây là những lỗi mà sinh viên mới học Python thường gặp nhất:

```text
Lỗi Python thường gặp
├── SyntaxError
│   └── Sai cú pháp
├── IndentationError
│   └── Sai thụt lề
├── NameError
│   └── Dùng biến chưa khai báo
├── TypeError
│   └── Thao tác trên kiểu dữ liệu không phù hợp
├── ValueError
│   └── Giá trị hợp lệ về kiểu nhưng không hợp lệ về nội dung
├── IndexError
│   └── Truy cập chỉ số vượt phạm vi
├── KeyError
│   └── Truy cập key không tồn tại trong dict
├── AttributeError
│   └── Gọi thuộc tính/phương thức không tồn tại
├── ZeroDivisionError
│   └── Chia cho 0
├── FileNotFoundError
│   └── Không tìm thấy file
├── ModuleNotFoundError
│   └── Không tìm thấy module import
├── ImportError
│   └── Lỗi khi import
├── OverflowError
│   └── Kết quả tính toán quá lớn (ít gặp)
├── MemoryError
│   └── Hết bộ nhớ
├── RecursionError
│   └── Đệ quy quá sâu
├── AssertionError
│   └── assert thất bại
├── StopIteration
│   └── Iterator đã hết phần tử
└── KeyboardInterrupt
    └── Người dùng nhấn Ctrl + C
```

Ví dụ:

### 1. SyntaxError

```python
if x == 1
    print(x)
```

```
SyntaxError: expected ':'
```

---

### 2. IndentationError

```python
if True:
print("Hello")
```

```
IndentationError: expected an indented block
```

---

### 3. NameError

```python
print(a)
```

```
NameError: name 'a' is not defined
```

---

### 4. TypeError

```python
"5" + 3
```

```
TypeError: can only concatenate str (not "int") to str
```

---

### 5. ValueError

```python
int("abc")
```

```
ValueError: invalid literal for int() with base 10: 'abc'
```

Đây chính là lỗi bạn vừa gặp:

```python
int('')
int(' ')
int('abc')
```

---

### 6. IndexError

```python
a = [1, 2, 3]
print(a[5])
```

```
IndexError: list index out of range
```

---

### 7. KeyError

```python
d = {"a": 1}
print(d["b"])
```

```
KeyError: 'b'
```

---

### 8. AttributeError

```python
x = 5
x.append(1)
```

```
AttributeError: 'int' object has no attribute 'append'
```

---

### 9. ZeroDivisionError

```python
10 / 0
```

```
ZeroDivisionError: division by zero
```

---

### 10. ModuleNotFoundError

```python
import abcxyz
```

```
ModuleNotFoundError: No module named 'abcxyz'
```

---

## Cách đọc traceback để debug

Ví dụ:

```text
Traceback (most recent call last):
  File "main.py", line 16, in solve
    if int(x1[i]) == p:
ValueError: invalid literal for int() with base 10: ''
```

Đọc từ dưới lên:

```text
ValueError
├── Loại lỗi: ValueError
├── Thông báo: int('') không hợp lệ
├── Hàm xảy ra lỗi: solve
├── File: main.py
└── Dòng: 16
```

Khi debug, hãy tự hỏi theo thứ tự:

```text
1. Lỗi thuộc loại gì?
2. Xảy ra ở dòng nào?
3. Biến nào trong dòng đó có giá trị bất thường?
4. In ra bằng print(repr(...)) để kiểm tra.
```

Ví dụ:

```python
print(i)
print(repr(x1))
print(repr(x1[i]))
```

`repr()` rất hữu ích vì nó giúp thấy được:

```python
repr(' ')   # "' '"
repr('')    # "''"
repr('\n')  # "'\\n'"
```

Nếu bạn học Python để thi OJ/competitive programming, thì **80% lỗi runtime** sẽ rơi vào khoảng:

```text
├── IndexError
├── ValueError
├── TypeError
├── NameError
└── AttributeError
```

Chỉ cần đọc đúng traceback là thường sẽ tìm ra lỗi rất nhanh.