
### Tóm tắt
> *nói chung là cái này là các thuộc tính giúp lấy thông tin của trình duyệt*
```text
JavaScript
    │
    ▼
Browser APIs
    │
    ├── URL hiện tại       → location
    ├── Lịch sử trình duyệt → history
    ├── Thông tin browser  → navigator
    ├── Cookie             → document.cookie
    ├── Dữ liệu lâu dài    → localStorage
    ├── Dữ liệu tạm thời   → sessionStorage
    ├── Hộp thoại          → alert / prompt
    └── Debug              → console.log
    
    
location
└── URL / dữ liệu từ URL

document.cookie
└── Cookie

localStorage
└── Dữ liệu lưu lâu dài

sessionStorage
└── Dữ liệu lưu tạm thời

navigator
└── Thông tin browser

history
└── Lịch sử và URL

console.log()
└── Debug
```
Trong quá trình học Pentest, hãy đặc biệt nhớ 4 thứ:
```text
location
document.cookie
localStorage
sessionStorage
```
Vì chúng xuất hiện rất nhiều trong **DOM XSS, client-side security và JavaScript analysis**.


### 1. **location** — URL hiện tại
Các thuộc tính  thường gặp 
```javascript
location.href
//-->https://example.com/search?q=test

console.log(location.href);
//->https://example.com/search?q=test

location.search
//->?q=test

location.hostname
location.pathname
```


 *Trong Pentest*
```javascript
location = "https://example.com"; //→ Chuyển hướng trang.

location.href = "/login"; // ->redirect 
```


Đặc biệt quan trọng khi học **DOM XSS**, vì `location` thường là một **source**:

```javascript
let data = location.search;
```

Dữ liệu từ URL được đưa vào JavaScript.
### 2. `history` — lịch sử trình duyệt

```javascript
history.back(); //quay lại trang trước 

history.forward(); // đi tới trang sau 

history.pushState({}, "", "/new-url"); // thay đổi url mà ko reload trang

history.replaceState({}, "", "/new-url"); // thay url hiện tại 
```

Pentest: thường gặp khi phân tích SPA và các ứng dụng dùng client-side routing.

---

### 3. **navigator**— thông tin về trình duyệt
```javascript
navigator.userAgent // thông tin browser

navigator.language// ngôn ngữ trình duyệt

navigator.platform// nền tảng hệ điều hành
```
Ví dụ:

```javascript
console.log(navigator.userAgent);
```

Trong Pentest, bạn có thể thấy code:

```javascript
if (navigator.userAgent.includes("Mobile")) {
    // mobile logic
}
```

→ Website kiểm tra User-Agent.
### 4. **document.cookie** ⭐

```javascript
document.cookie
//-> session=abc123; theme=dark

document.cookie = "theme=light"; // gán cookie
document.cookie = "theme=; expires=Thu, 01 Jan 1970 00:00:00 UTC";// xóa cookie khi hết hạn

```

 **Trong Pentest**
Nếu cookie:
```text
session=abc123
```
và **không có `HttpOnly`**, JavaScript có thể đọc được:
```javascript
document.cookie
```
Đây là lý do XSS có thể dẫn đến đánh cắp cookie trong một số trường hợp.
Nhưng:
```text
HttpOnly Cookie
```
→ JavaScript **không đọc được** bằng *document.cookie*.

### 5. **localStorage** ⭐

Lưu dữ liệu trong trình duyệt.

```javascript
localStorage.setItem("username", "Tai"); // -> Tai 
localStorage.getItem("username"); // đọc
localStorage.removeItem("username"); //xóa 1 key
localStorage.clear();// xóa tất cả 

```

Ví dụ:
```javascript
localStorage.setItem("token", "abc123");
localStorage.getItem("token"); // -> abc 123 
```

*trong pentest*
Nếu ứng dụng lưu:

```javascript
localStorage.setItem("token", "secret_token");
```

thì JavaScript chạy trên cùng origin có thể đọc:

```javascript
localStorage.getItem("token");
```

Đây là một điểm thường được kiểm tra khi phân tích XSS và client-side security.

### 6. **sessionStorage**

Gần giống **localStorage**.

```javascript
sessionStorage.setItem("token", "abc");
sessionStorage.getItem("token");
```

Khác nhau:

```text
localStorage
└── Tồn tại lâu dài

sessionStorage
└── Thường tồn tại trong một tab/session
```

### 7. **alert()**

Hiện thông báo:

```javascript
alert("Hello");
```
Trong XSS:
```javascript
alert(document.domain);
```
*Đây thường là payload kiểm tra XSS cơ bản.*

### 8. **prompt()**
Hiện hộp nhập dữ liệu:
```javascript
let name = prompt("Your name?");
// nhập Tai --> name =Tai

```

### 9. **console.log()**

In dữ liệu ra Console:

```javascript
console.log("Hello");

//debug
let token = "abc123";
console.log(token);
```


Trong Pentest, bạn sẽ dùng rất nhiều để theo dõi dữ liệu:
```javascript
let input = location.search;

console.log(input);
```



### -Học từ ví dụ thực tế
> *Do có vẻ khá khó hiểu khi học chay nên mình sẽ học từ ví dụ*

```text
User
 │
 ├── Nhập username
 │
 ├── Đăng nhập
 │      │
 │      ├── localStorage → lưu username
 │      ├── sessionStorage → lưu trạng thái phiên
 │      └── document.cookie → lưu session
 │
 ├── Website đọc:
 │      ├── location → URL
 │      ├── navigator → trình duyệt
 │      └── history → thay đổi URL
 │
 └── console.log → debug
```

Tạo file `index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Browser API Lab</title>
</head>
<body>
    <h1>My Website</h1>
    <input id="username" placeholder="Username">
    <button id="loginBtn">Login</button>
    <button id="logoutBtn">Logout</button>
    <p id="status"></p>
    <script>
        // =================================
        // 1. console.log()
        // =================================
        console.log("Website started");
        // =================================
        // 2. location
        // =================================
        console.log("Current URL:", location.href);

        // =================================
        // 3. navigator
        // =================================
        console.log("Browser:", navigator.userAgent);
        // =================================
        // 4. localStorage
        // =================================
        let savedUsername =
            localStorage.getItem("username");
        console.log("Saved username:", savedUsername);
        // =================================
        // 5. sessionStorage
        // =================================
        let loggedIn =
            sessionStorage.getItem("loggedIn");

        console.log("Logged in:", loggedIn);
        // =================================
        // 6. document.cookie
        // =================================

        console.log("Cookies:", document.cookie);

        // =================================
        // 7. Login
        // =================================

        document
            .getElementById("loginBtn")
            .addEventListener("click", function() {
                let username =
                    document.getElementById("username").value;
                // Lưu lâu dài
                localStorage.setItem(
                    "username",
                    username
                );

                // Lưu trong session
                sessionStorage.setItem(
                    "loggedIn",
                    "true"
                );

                // Tạo cookie
                document.cookie =
                    "session=abc123";

                console.log(
                    "User logged in:",
                    username
                );

                // Thay đổi nội dung trang
                document.getElementById("status")
                    .textContent =
                    "Hello " + username;

                // Thay đổi URL
                history.pushState(
                    {},
                    "",
                    "/dashboard"
                );
            });

        // =================================
        // 8. Logout
        // =================================
        document
            .getElementById("logoutBtn")
            .addEventListener("click", function() {
                localStorage.removeItem(
                    "username"
                );
                sessionStorage.removeItem(
                    "loggedIn"
                );

                document.cookie =
                    "session=; expires=Thu, 01 Jan 1970 00:00:00 UTC";

                console.log("User logged out");

                document.getElementById("status")
                    .textContent =
                    "Logged out";

                history.pushState(
                    {},
                    "",
                    "/login"
                );
            });

        // =================================
        // 9. alert()
        // =================================

        function showAlert() {

            alert("Hello from JavaScript");

        }
        // =================================
        // 10. prompt()
        // =================================
        function askName() {
            let name =
                prompt("What is your name?");
            console.log(
                "User entered:",
                name
            );
        }
    </script>
</body>
</html>
```

## Bài tập 1
Sau khi chạy ví dụ trên, hãy sửa code để:
1. Khi mở trang, nếu `localStorage` đã có username thì hiển thị:
    ```text
    Welcome back, Tai
    ```

2. Khi Login, thay:   
    ```javascript
    document.cookie = "session=abc123";
    ```
    bằng cookie chứa username.
    
3. Thử tự trả lời:
    
    ```text
    User input
        ↓
    username
        ↓
    localStorage
    ```
    
Trong luồng này, đâu là **Source** và đâu là **Storage**?
    
Gửi code bạn sửa. Mình sẽ review từng dòng.
```js
.....

if(localStorage.getItem("username")){
console.log(`welcomback ${localStorage.getItem("username")}`);
}
</script>
</body>
</html>
```

## Bài tập 2

Chạy đoạn code sau trong Console:

```javascript
localStorage.setItem("token", "abc123");//-> set gia tri truong "token"='abc123'

console.log(localStorage.getItem("token"));//-> lay gia tri cua truong "token"

sessionStorage.setItem("user", "Tai");//->set gia tri Tai cho "user" vao storage cua trinh duyet

console.log(sessionStorage.getItem("user"));//--> hien thi gia tri cua truong "user" luu o trinh duyet

console.log(location.href);//-> hien thi duong link href

console.log(navigator.userAgent);//->hien thi thong tin trinh duyet

console.log(document.cookie); //-> hien thi cookie cua trinh duyet
```

Sau đó trả lời:
1. `localStorage` và `sessionStorage` khác nhau thế nào?
2. `document.cookie` dùng để làm gì?
3. `location.href` chứa gì?
4. Nếu một website lưu `token` trong `localStorage`, JavaScript trên cùng origin có thể đọc nó không?
