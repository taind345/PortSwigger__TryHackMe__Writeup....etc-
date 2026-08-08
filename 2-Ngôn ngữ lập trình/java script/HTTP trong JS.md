## Chương 7: HTTP
Hãy hiểu bằng một ví dụ duy nhất:
```text
JavaScript
    │
    ├── fetch()
    │       └── gửi HTTP request
    │
    ├── GET
    │       └── lấy dữ liệu
    │
    ├── POST
    │       └── gửi dữ liệu
    │
    ├── Headers
    │       └── thông tin kèm request
    │
    ├── Response
    │       └── dữ liệu server trả về
    │
    ├── JSON
    │       └── định dạng dữ liệu
    │
    └── Promise
            └── kết quả sẽ có trong tương lai
```

>*lưu ý là JS hoạt động ở client* , do đó , các hành động này giúp client có thể lấy dữ liệu, request , post thông tin tới server
## 1. GET với `fetch()`
```javascript
fetch("https://api.example.com/users")
```

Ví dụ thực tế:
```javascript
fetch("https://jsonplaceholder.typicode.com/users") // request GET .../users
    .then(response => response.json()) 
    .then(data => {
        console.log(data);
    });
```
>*--> bên trong then(....) là một hàm.*
>*->fetch() gửi request tới server --> hàm bên trong then(...) định nghĩa cái request đó*
## 2. POST

```javascript
fetch("/login", {
    method: "POST",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify({
        username: "Tai",
        password: "123456"
    })
});
```

HTTP request:
```text
POST /login

Content-Type: application/json

{
    "username": "Tai",
    "password": "123456"
}
```

Pentest: đây chính là kiểu request bạn thường thấy khi test API.
>có thể thấy **fetch("url", {method :{}   ,  header: {}  , body: JSON.stringify({}))**

### -Headers
*--> có thể thấy là trường header:{} xuất hiện trong fetch()*
```javascript
headers: {
    "Content-Type": "application/json",
    "Authorization": "Bearer token"
}
```

*Headers chứa metadata:*
```text
Content-Type
Authorization
Cookie
User-Agent
```

### -JSON
<u>còn gọi là JavaScript Object:</u>
*--> có thể hiểu json là* <u>object viết dưới dạng string</u>
--> JSON là kiểu dữ liệu  giúp các thành phần như client, server, giao tiếp với nhau ??

```javascript
let user = {
    name: "Tai",
    age: 21
};

JSON.stringify(user); //->Chuyển OBJECt --> Json (text) 
//{"name":"Tai","age":21}

JSON.parse('{"name":"Tai"}'); //--> chuyển Json(text)--> OBJECT 
```

## 5. Promise
Khi gọi HTTP, kết quả **không xuất hiện ngay lập tức**.
```text
fetch()
    │
    ├── Đang gửi request
    │
    └── Promise
            │
            ├── Thành công → then()
            │
            └── Thất bại → catch()
```

Ví dụ:
```javascript
fetch("/api/user")
    .then(res => response.json())
    .then(data => {
        console.log(data);
    })
    .catch(error => {
        console.log(error);
    });
```
>*-->* cái *.then()* và *.catch()* mà mình đã học ở trên nó được gọi là **promise**
>-> dấu "*.*" trước then() do *then()* và *catch()* là phương thức của *fetch()*

**-hiểu đúng về `res` , `data`, `error` trong ví dụ trên :**
>->mấy cái trên đơn giản là <u>biến mà dev tự đặt</u> thôi
>->luồng hoạt động như sau:
>	-*response serverr* --đi qua --> *.then( hàm1)*-->kết quả -->*them(hàm 2)*-->..
>	..-->.*then(hàm n)*--><u> kết quả response được xử lý qua 7749 bước </u>


## 6. XMLHttpRequest
Đây là <u>cách cũ hơn:</u>, cái này nó thay cho *fetch()*
```javascript
let req = new XMLHttpRequest();

req.open("GET", "/accountDetails");
req.onload = function() {
    console.log(this.responseText);
};

req.send();
```

>ok đại khái là  
> - tạo biến request bằng *open()*
> - định nghĩa cái request ~ giống với *.then()* bằng *onload=hàm(...)*
> - gửi request bằng *send()*

 Trong Pentest bạn vẫn sẽ gặp nó trong:
- XSS payload
- CSRF PoC
- <u>Source code cũ</u>
- PortSwigger labs
    
Ví dụ:
```javascript
var req = new XMLHttpRequest();

req.open("GET", "/accountDetails", true);

req.withCredentials = true; // yêu cầu sever gửi kèm cookie trong respond

req.send();
```


## Bài tập

Chạy:
```javascript
fetch("https://jsonplaceholder.typicode.com/users/1")
    .then(response => response.json())
    .then(data => {
        console.log(data);
    });
```

Sau đó viết một `POST request` gửi:
```javascript
{
    username: "Tai",
    password: "123456"
}
```
đến:
```text
https://jsonplaceholder.typicode.com/posts
```

Gửi code của bạn. Mình sẽ review ngắn gọn từng phần.
```js
fetch("https://jsonplaceholder.typicode.com/users/1")
.then( response =>{ return response.json(); } )
.then( data =>{ console.log(data); } );

  

fetch("https://jsonplaceholder.typicode.com/users/1" ,
	{method : "POST" ,
	headers : {"Content-Type": "application/json"} ,
	body : JSON.stringify( {name: "Tai", pass : "123456"})
	}
);
```
![[Pasted image 20260719232714.png]]



