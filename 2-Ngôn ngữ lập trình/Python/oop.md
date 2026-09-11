OOP Python cho người đã biết C++ thì chủ yếu là thay đổi cú pháp và triết lý. Dưới đây là tất cả những gì bạn cần dùng hằng ngày, ngắn gọn hết mức có thể.

---

## 1. Định nghĩa class & `__init__`
```python
class Dog:
    # Class attribute (biến static trong C++)
    species = "Canis familiaris"

    # Constructor (KHÔNG overload được, dùng default args)
    def __init__(self, name, age=0):
        # Instance attributes (phải gán qua self)
        self.name = name
        self.age = age

    # Method (luôn có self là tham số đầu tiên)
    def bark(self):
        print(f"{self.name} says woof!")
```

- Không có `private/public` thật sự; quy ước ==`_tên` là protected==, ==`__tên` là name-mangling== (tránh đụng độ khi kế thừa).
- ==`self` tương đương `this`== trong C++ nhưng **phải viết tường minh**.

---

## 2. Tạo đối tượng & truy cập
```python
d = Dog("Buddy", 3)       # không cần new
print(d.name)             # truy cập trực tiếp thuộc tính
d.bark()                  # tự động truyền d -> self
# Class attribute truy cập qua class hoặc instance
print(Dog.species)        # "Canis familiaris"
```

---

## 3. Kế thừa & `super()`
```python
class Beagle(Dog):
    def __init__(self, name, age, nose_power):
        super().__init__(name, age)   # gọi constructor cha
        self.nose_power = nose_power

    # Ghi đè (override) method
    def bark(self):
        super().bark()    # gọi method cha
        print("and then howls!")
```
- Đa kế thừa được hỗ trợ, thứ tự MRO xác định bằng C3 linearization. Dùng `super()` để duyệt đúng chuỗi kế thừa.

---

## 4. Phương thức đặc biệ==t (dunder methods)== – "Operator Overloading"
Dùng để định nghĩa hành vi của object với toán tử có sẵn, in ấn, so sánh,...

```python
class Vector2D:
    def __init__(self, x, y):
        self.x, self.y = x, y

    # In đẹp cho người dùng (str() hoặc print())
    def __str__(self):
        return f"({self.x}, {self.y})"

    # Biểu diễn rõ ràng cho developer (repr()), dùng trong debug
    def __repr__(self):
        return f"Vector2D({self.x}, {self.y})"

    # Cộng (+)
    def __add__(self, other):
        return Vector2D(self.x + other.x, self.y + other.y)

    # So sánh bằng (==)
    def __eq__(self, other):
        if not isinstance(other, Vector2D):
            return NotImplemented
        return self.x == other.x and self.y == other.y

    # Nhỏ hơn (<) – đủ để sort
    def __lt__(self, other):
        return (self.x, self.y) < (other.x, other.y)

    # Có thể định nghĩa __len__, __getitem__, __setitem__ để object hoạt động như container
```

**Các dunder phổ biến:** `__init__, __str__, __repr__, __eq__, __lt__, __le__, __hash__` (nếu dùng trong set/dict), `__len__, __getitem__, __iter__`.

---

## 5. `@property` (getter/setter kiểu Python)
```python
class Circle:
    def __init__(self, radius):
        self._radius = radius   # quy ước protected

    @property
    def radius(self):
        return self._radius

    @radius.setter
    def radius(self, value):
        if value < 0:
            raise ValueError("Radius must be non-negative")
        self._radius = value

    @property
    def area(self):
        return 3.14159 * self._radius ** 2   # tính toán mỗi khi truy cập

c = Circle(5)
print(c.radius)   # gọi getter
c.radius = 10     # gọi setter
print(c.area)     # truy cập như attribute, nhưng là property computed
```
- Không cần viết `getX/setX` rõ ràng, có thể bắt đầu với attribute thường, sau chuyển sang `@property` mà không đổi interface.

---

## 6. `@staticmethod` & `@classmethod`
```python
class MyMath:
    @staticmethod
    def add(a, b):       # không cần self hay cls
        return a + b

    @classmethod
    def from_string(cls, data_str):   # cls là class hiện tại (giống self nhưng là class)
        # Thường dùng làm factory method
        return cls(*map(int, data_str.split(',')))

# Dùng:
print(MyMath.add(3,4))           # như hàm thường
obj = MyMath.from_string("1,2")
```
- `@staticmethod`: giống hàm bình thường, nằm trong namespace của class.
- `@classmethod`: nhận class làm đối số thứ nhất (`cls`), thường dùng trong kế thừa để tạo đúng kiểu subclass.

---

## 7. Hủy đối tượng & `__del__`
```python
class TempFile:
    def __init__(self, name):
        self.file = open(name, 'w')
    def __del__(self):
        self.file.close()    # dọn dẹp khi object bị garbage-collected
```
- Hiếm khi cần dùng, Python có garbage collector. Ưu tiên `with` (context manager) với `__enter__/__exit__`.

---

## 8. Mẹo dùng thực tế trong CP / dự án nhỏ
- **Dataclasses** (Python 3.7+): tự sinh `__init__, __repr__, __eq__` giúp giảm boilerplate.
  ```python
  from dataclasses import dataclass
  @dataclass
  class Point:
      x: int
      y: int
  ```
- **Enum** (thay cho `enum class` C++):
  ```python
  from enum import Enum, auto
  class Color(Enum):
      RED = auto()
      GREEN = auto()
  ```
- **Duck typing**: không cần abstract base class, chỉ cần object có method cần thiết. Nếu bắt buộc kiểm tra interface, dùng `abc.ABC` với `@abstractmethod`.

---

## 9. So sánh nhanh Python vs C++

| C++                            | Python                                        |
| ------------------------------ | --------------------------------------------- |
| `class A { ... };`             | `class A: ...` (thụt lề)                      |
| `A::A(int x) : x(x) {}`        | `def __init__(self, x): self.x = x`           |
| `virtual void f() = 0;`        | `@abstractmethod def f(self): pass` (cần ABC) |
| `std::cout << obj;`            | `print(obj)` → dùng `__str__`                 |
| `obj->method()`                | `obj.method()` (dấu `.`)                      |
| `public`, `private`            | quy ước `_` và `__`                           |
| copy constructor, destructor   | `__del__`, thường dùng `copy.copy`            |
| overload toán tử: `operator+`  | `__add__`                                     |
| static member: `static int x;` | class attribute (bên ngoài `__init__`)        |
| `static void func();`          | `@staticmethod`                               |
| `virtual` tự động khi override | luôn tự động override, không cần từ khóa      |

Chỉ cần nắm vững các điểm trên, bạn hoàn toàn đủ sức code OOP Python "đúng chuẩn" và gọn gàng. Nếu cần mình có thể mở rộng bất kỳ phần nào.