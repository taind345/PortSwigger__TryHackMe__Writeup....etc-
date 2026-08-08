Có 3 cách. Với mục tiêu học Pentest, mình khuyên theo thứ tự sau.

### Cách 1 (Khuyến nghị): Node.js
**Bước 1**
Cài Node.js.
Sau khi cài, mở Terminal:
```bash
node -v
```
Nếu hiện:
```text
v24.x.x
```

---
 **Bước 2**
Tạo file:
```text
lesson1.js
```
Ví dụ:
```javascript
let name = "Tai";

console.log(name);
```
---
**Bước 3**
Mở Terminal tại thư mục chứa file:
```bash
node lesson1.js
```
Kết quả:
```text
Tai
```
---
### Cách 2: Chrome DevTools (Rất phù hợp khi học Pentest)
Nhấn:
```
F12
```

↓

```
Console
```

↓

Gõ:

```javascript
let a = 10;

a
```

Enter.

Bạn sẽ thấy:

```
10
```

Đây là nơi bạn sẽ dùng rất nhiều khi làm PortSwigger.

---
### Cách 3: VS Code + Live Server (Dùng khi học DOM)

Tạo:

```
index.html
```

```html
<!DOCTYPE html>
<html>
<body>

<script src="lesson1.js"></script>

</body>
</html>
```

Chạy bằng Live Server.

Cách này sẽ dùng ở chương DOM.

### Mình khuyên bạn

Vì mục tiêu là **Pentest**, hãy cài:

- ✅ Node.js
    
- ✅ VS Code
    
- ✅ Chrome DevTools
    
Ba công cụ này là đủ cho hơn 90% quá trình học JavaScript.

