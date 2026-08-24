## Console
Chạy JavaScript trực tiếp:
```js
let x = 10;
console.log(x);
```
Dùng để:
```
thử code
debug
xem biến
```
*--> cái chức năng này trong f12 giúp mình chạy code js trực tiếp để tương tác vs html trang web*
## Network
-Giúp ta <u>Quan sát HTTP request:</u>
```
Client
  │
  ├── Request
  │
  ▼
Server
  │
  └── Response
```
Bạn có thể xem:
```
GET /api/user
POST /login
Headers
Cookies
Request Body
Response
```

Đây là tab **cực kỳ quan trọng khi Pentest**.
## Sources
Xem JavaScript source code:
```
Sources
   │
   └── file.js
```
Bạn có thể đọc:
```
function login() {
    ...
}
```
## Breakpoint

Ví dụ:
```js
let username = "Tai";
let token = "abc123";
console.log(token);
```
Đặt breakpoint tại:
```
console.log(token);
```

Khi chương trình chạy đến đó:

```
Code
 │
 ▼
Breakpoint
 │
 ├── Xem token
 ├── Xem username
 └── Xem call stack
```

Chương trình **tạm dừng**.

## debugger

Bạn có thể viết:

```
let token = "abc123";

debugger;

console.log(token);
```


*Chrome sẽ tự dừng khi chạy đến debugger`*.

## Ví dụ Pentest thực tế

```js 
let token = localStorage.getItem("token");

debugger;
fetch("/api/user", {
    headers: {
        Authorization: "Bearer " + token
    }
});
```

Trong DevTools:
```
Sources
   ↓
debugger
   ↓
Xem token
   ↓
Network
   ↓
Xem request
   ↓
Authorization: Bearer ...
```

Đây là quy trình rất phổ biến:

```
Đọc JavaScript
      ↓
Đặt breakpoint
      ↓
Theo dõi biến
      ↓
Network
      ↓
Phân tích HTTP request
```


### Bài tập 
Bài tập DevTools: **theo dõi một request đăng nhập giả**.
Tạo file `index.html`:
```html
<!DOCTYPE html>
<html>
<body>

<button id="login">Login</button>

<script>
document.getElementById("login").onclick = function () {
    let username = "Tai";
    let password = "123456";

    debugger;
    fetch("https://jsonplaceholder.typicode.com/posts", {
        method: "POST",
        headers: {
            "Content-Type": "application/json"
        },
        body: JSON.stringify({
            username: username,
            password: password
        })
    })
    .then(response => response.json())
    .then(data => {
        console.log("Server response:", data);
    });
};
</script>

</body>
</html>
```

Nhiệm vụ:
1. Mở **DevTools → Sources**.
2. Tìm dòng:
    ```
    debugger;
    ```
![[Pasted image 20260721170013.png|416]]    
3. Bấm **Login**.
4. Khi code dừng:
    - Xem giá trị `username`.
    - Xem giá trị `password`.
5. Mở **Network**.
6. Tìm request `posts`.
7. Xem:
    - Request Method
    - Request Headers
    - Request Payload
    - Response
![[Pasted image 20260721170310.png|549]]
![[Pasted image 20260721170629.png|539]]- response
![[Pasted image 20260721170723.png]]

```
Click Login
    ↓
JavaScript
    ↓
debugger
    ↓
Sources: xem biến
    ↓
fetch()
    ↓
Network: xem HTTP Request
    ↓
Server Response
```
