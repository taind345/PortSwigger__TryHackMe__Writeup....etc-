


## 1: Đọc từ bàn phím, in ra màn hình

```python
n = int(input())
arr = list(map(int, input().split()))
stack = []
for num in arr:
    if stack and (stack[-1] + num) % 2 == 0:
        stack.pop()
    else:
        stack.append(num)
print(len(stack))
```

## 2: Đọc từ file, ghi ra file

```python
with open('input.txt', 'r') as f:
    n = int(f.readline())
    arr = list(map(int, f.readline().split()))

# Xử lý
stack = []
for num in arr:
    if stack and (stack[-1] + num) % 2 == 0:
        stack.pop()
    else:
        stack.append(num)

with open('output.txt', 'w') as f:
    f.write(str(len(stack)))
```

## 3: Dùng sys.stdin/stdout (nhanh nhất)
![[Pasted image 20260616214910.png]]

```python
import sys
data = sys.stdin.read().split()
n = int(data[0])
arr = list(map(int, data[1:]))
stack = []
for num in arr:
    if stack and (stack[-1] + num) % 2 == 0:
        stack.pop()
    else:
        stack.append(num)
print(len(stack))
```

![[Pasted image 20260617064447.png]]
> [!NOTE] Note
> nhập 1 chuỗi  thành 1 list số


```python
input().split()            # "1 2 3" → ['1', '2', '3']

map(int, ['1', '2', '3']) # Biến mỗi string thành int → map object

list(map object)          # Chuyển map object thành list [1, 2, 3]
```

## ouput 
Từ phiên bản Python 3.6 trở đi, **f-string** là cách viết được ưa chuộng nhất vì nó vừa dễ đọc, vừa có tốc độ xử lý nhanh nhất. Bạn chỉ cần đặt chữ f trước dấu ngoặc kép và truyền thẳng tên biến vào trong dấu ngoặc nhọn {}.


```python
name = "Đức Tài"
age = 21
gpa = 3.456

# Truyền trực tiếp biến vào {}, muốn định dạng float thì thêm :.2f
print(f"Tên: {name}, Tuổi: {age}, GPA: {gpa:.2f}")
```