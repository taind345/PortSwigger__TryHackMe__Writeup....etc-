## Giáo án tối nay (khoảng 4 tiếng)

```text
JavaScript Basic (4h)
│
├── Chương 1: Variable (20')
│   ├── let
│   ├── const
│   └── var
│
├── Chương 2: Data Types (35')
│   ├── Number
│   ├── String
│   ├── Boolean
│   ├── null
│   ├── undefined
│   ├── Object
│   └── Array
│
├── Chương 3: Operators (20')
│   ├── Arithmetic
│   ├── Comparison
│   ├── Logical
│   ├── ===
│   └── !==
│
├── Chương 4: Điều kiện (20')
│   ├── if
│   ├── else
│   └── switch
│
├── Nghỉ 10'
│
├── Chương 5: Loop (25')
│   ├── for
│   └── while
│
├── Chương 6: Function (35')
│   ├── function
│   ├── parameter
│   ├── return
│   └── callback (giới thiệu)
│
├── Chương 7: Arrow Function (25')
│
├── Chương 8: Scope (25')
│   ├── Global
│   ├── Function
│   └── Block
│
└── Tổng ôn + Quiz (25')
```

---

# Cách học

Mỗi chương đều theo đúng quy trình này.

```text
1. Khái niệm (2 phút)

↓

2. So sánh với C++ (2 phút)

↓

3. Ví dụ nhỏ (5 phút)

↓

4. Bạn làm bài tập (5~10 phút)

↓

5. Mình review

↓

6. Sang chương tiếp
```

Không được đọc liên tục 4 tiếng.

---

# Quy tắc của lớp học

❌ <u>Không copy code.</u>

❌ <u>Không đọc đáp án trước</u>.

✔ Tự chạy code.

✔ Tự sửa lỗi.

✔ Hỏi ngay khi không hiểu.

---

# Công cụ

Chỉ cần một trong các cách:

- Chrome DevTools → Console (khuyến nghị)
    
- VS Code + Code Runner
    
- Node.js
    
- JSFiddle
    
- CodePen
    

---

# Cách mình sẽ review

Ví dụ bạn gửi

```js
let a = 10;
const b = 20;
a = b;
```

Mình sẽ chấm như này

```
Điểm: 9/10

✔ Dùng let đúng.

✔ const đúng.

❌ Thiếu hiểu về immutable.

Nên nhớ:

const không làm object bất biến.

Chỉ làm biến không được gán lại.
```

Không chỉ nói đúng/sai.

---

# Sau tối nay bạn sẽ làm được

✔ Đọc được khoảng **70–80%** JavaScript cơ bản trong PortSwigger.

✔ Hiểu đa số đoạn:

```js
let

const

if

for

function

return

fetch(...)

.then(...)

=>

[]
{}
```

✔ Không còn cảm giác "JavaScript nhìn như tiếng ngoài hành tinh".

---

# Một yêu cầu đối với bạn

Trong suốt buổi học, **đừng cố ghi nhớ**.

Hãy tập trung vào **viết code**. Với nền C++ của bạn, <u>điều quyết định không phải là thuộc cú pháp </u>mà là hình thành phản xạ. Nếu <u>mỗi khái niệm bạn đều tự viết 5–10 dòng mã và chạy thử</u>, <u>bạn sẽ nhớ nhanh và bền hơn nhiều</u> so với việc chỉ đọc.

Từ bây giờ mình sẽ coi bạn là học viên trong một khóa học 1:1. Mỗi bài sẽ ngắn gọn, có bài tập, và mình sẽ không chuyển sang bài tiếp theo cho đến khi bạn thực sự nắm được bài hiện tại.
_--> tức là mình phải  tự viết code 
