Dưới đây là nội dung đã được định dạng lại cho Obsidian, bạn có thể copy trực tiếp:


## 1. Cấu trúc dữ liệu cơ bản

```python
# List/Array
arr = [1, 2, 3, 4, 5]
arr.append(6)           # Thêm cuối
arr.pop()              # Xóa cuối
arr.insert(0, 0)       # Thêm tại vị trí
arr.remove(3)          # Xóa giá trị
arr.sort()             # Sắp xếp
sorted_arr = sorted(arr)  # Trả về mảng mới
arr.reverse()          # Đảo ngược
==> truy cập được chỉ số
# String
s = "Hello"
s.split()              # Tách thành list
" ".join(list)         # Nối list thành string
s.strip()              # Xóa khoảng trắng đầu/cuối
s.replace("old", "new") # Thay thế
s.startswith("H")      # Kiểm tra prefix
s.endswith("o")        # Kiểm tra suffix

# Dictionary/HashMap
d = {"a": 1, "b": 2}
d["c"] = 3             # Thêm/Update
d.get("a", 0)          # Lấy giá trị (có default)
d.keys()               # Danh sách keys
d.values()             # Danh sách values
d.items()              # Cặp key-value

# Set
s = set()
s = {1, 2, 3}
s.add(4)               # Thêm phần tử
s.remove(2)            # Xóa phần tử
union = s1 | s2        # Hợp
intersection = s1 & s2 # Giao
difference = s1 - s2   # Hiệu
```

## 2. Các thuật toán tìm kiếm và sắp xếp

```python
# Binary Search
import bisect
idx = bisect.bisect_left(arr, x)  # Tìm vị trí chèn
idx = bisect.bisect_right(arr, x)

# Sắp xếp nâng cao
arr.sort(key=lambda x: x[1])     # Sắp xếp theo key
arr.sort(key=lambda x: (-x[0], x[1]))  # Sắp xếp đa điều kiện
```

## 3. Các cấu trúc dữ liệu nâng cao

```python
# Heap (Min-heap mặc định)
import heapq
heap = []
heapq.heappush(heap, 5)          # Thêm phần tử
min_val = heapq.heappop(heap)    # Lấy phần tử nhỏ nhất
heapq.heapify(arr)               # Chuyển list thành heap

# Max-heap (sử dụng giá trị âm)
heapq.heappush(max_heap, -value)
max_val = -heapq.heappop(max_heap)

# Deque (Queue/Stack)
from collections import deque
dq = deque()
dq.append(1)           # Thêm cuối
dq.appendleft(2)       # Thêm đầu
dq.pop()              # Xóa cuối
dq.popleft()          # Xóa đầu

# Counter
from collections import Counter
cnt = Counter(arr)     # Đếm tần suất
most_common = cnt.most_common(k)  # K phần tử phổ biến nhất

# DefaultDict
from collections import defaultdict
dd = defaultdict(int)  # Giá trị mặc định 0
dd = defaultdict(list) # Giá trị mặc định []

# OrderedDict (giữ thứ tự chèn)
from collections import OrderedDict
```

## 4. Các công cụ toán học

```python
# Toán học cơ bản
import math
math.pow(2, 3)        # 2^3
math.sqrt(16)         # Căn bậc 2
math.gcd(12, 18)      # Ước chung lớn nhất
math.lcm(12, 18)      # Bội chung nhỏ nhất (Python 3.9+)
math.factorial(5)     # 5!

# Infinity
inf = float('inf')
neg_inf = float('-inf')

# Bit manipulation
x & 1                 # Kiểm tra chẵn/lẻ
x >> 1                # Dịch phải 1 bit (chia 2)
x << 1                # Dịch trái 1 bit (nhân 2)
x & (x-1)             # Xóa bit 1 cuối cùng
```

## 5. Các hàm tiện ích quan trọng

```python
# Enumerate
for idx, val in enumerate(arr):
    print(idx, val)

# Zip
for a, b in zip(arr1, arr2):
    print(a + b)

# List comprehension
squares = [x**2 for x in range(10)]
evens = [x for x in arr if x % 2 == 0]

# Lambda functions
sort_by_second = sorted(pairs, key=lambda x: x[1])
```
