-trong phần này ta sẽ học về
![[Pasted image 20260713164736.png|284]]

### Operators 
- **Arithmetic (Toán học):** 
    - Cơ bản: *+* (cộng), *-* (trừ), * (nhân), */* (chia)., %

```js
let num = 10
let num2= 20
let num3= num/num2
```

- **Comparison (So sánh):** `>`, `<`, `>=`, `<=`.
- **Logical (Logic):** && , ||
```js
let a=1
let b =2
if(a>b){
	console.log("bigger")
}
```
- **`===` và `!==` (RẤT QUAN TRỌNG TRONG JS):**
*-mấy cái trên tương đối giống c++ , khác biệt nằm ở === và ==*
```  js
let a=5 ;
let b="5" ;
if(a==b){
	// tra ve true
	// neu dung == thi a==b
}
if(a===b){
	//tra ve false
	// cai nay tuong ung voi == ben c++
}
```


### Điều kiện 
*-giống hệt với if else hay switch case bên c++*
``` js
let role = 'admin';
    switch (role) {
        case 'admin':
            console.log("Toàn quyền hệ thống");
            break; // BẮT BUỘC phải có break để thoát vòng
        case 'user':
            console.log("Chỉ xem bài viết");
            break;
        default:
            console.log("Chưa đăng nhập");
    }
```

### Loop 
- **for:** cú pháp y hệt c++
    ``` js
    // In ra số từ 1 đến 3
    for (let i = 1; i <= 3; i++) {
        console.log("Số: " + i);
    }
    ```
    
- **while:** cũng y hệt c++

    ``` js
    let hp_boss = 100;
    // Chừng nào máu boss còn lớn hơn 0 thì tiếp tục đánh
    while (hp_boss > 0) {
        console.log("Chém 1 nhát!");
        hp_boss -= 30; // Giảm máu, nếu không có dòng này sẽ lặp vô tận (đứng máy)
    }
    ```
**Task kiểm tra nhanh (5 phút):**

Kết hợp kiến thức 3 chương trên, bạn hãy viết bằng JS:

- Dùng vòng lặp `for` chạy từ 1 đến 5.
    
- Dùng `if/else` kết hợp toán tử `%` và `===` để kiểm tra. Nếu số hiện tại chia hết cho 2 thì in ra `"[số đó] là số chẵn"`, ngược lại in `"[số đó] là số lẻ"`.

```js
for(let i=0; i<=5; i++){
	if(i%2===0){
	console.log("sochan") ;
	}
	else{
	console.log("sole") ;
	}
}
```

