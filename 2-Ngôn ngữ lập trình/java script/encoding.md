### 1. URL Encoding
*phần này nói về url encdoing mà mình đã hay gặp trong việc trong việc server xử lý đầu vào*
URL không thể chứa tùy ý ký tự như:
```
space
&
?
=
```
Nên JavaScript encode chúng.

```js 
encodeURIComponent("hello world")

//==> hello%20world
```

Giải mã:

```js 
decodeURIComponent("hello%20world")
//hello world
```

Pentest:

```
let input = location.search;
```

Nếu URL chứa dữ liệu đặc biệt, cần hiểu nó đã được encode hay chưa.
### 2. Base64
```js
btoa("hello")
//aGVsbG8=
```

```js
atob("aGVsbG8=")
// hello 
```

**Base64 không phải encryption.**

Pentest thường gặp:

```
eyJ1c2VyIjoiYWRtaW4ifQ==
```

Có thể thử:

```
atob("eyJ1c2VyIjoiYWRtaW4ifQ==")
```

---

### 3. **URLSearchParams**
Phân tích query string:
```js 
let params = new URLSearchParams(
    "?user=Tai&role=admin"
);

params.get("user"); // Tai
params.get("role"); // admin
```

Sơ đồ:

```
?user=Tai&role=admin
        ↓
URLSearchParams
        ↓
params.get("user")
        ↓
Tai
```
**-pramgrams có kiểu dữ liệu là gì ?**
	-> kiểu dữ liệu là OBJECT URLSearch pagrams :
				URLSearchParams
				├── .get()
				├── .set()
				├── .delete()
				└── .has()
     Nó là một **object đặc biệt có sẵn các method để xử lý query parameters**.
**-Object này giúp tách các tham số trong url**
	-> Ví dụ URL:	https://example.com/search?name=Tai&age=21	
	Phần sau dấu `?` là **query parameters**:
		name=Tai
		age=21
ví dụ thực tế:
```js 
let params = new URLSearchParams(location.search);
let name = params.get("name");
console.log(name);
```
