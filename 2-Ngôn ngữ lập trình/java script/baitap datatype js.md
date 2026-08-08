## Bài 1
Khai báo các biến sau:
```js
age = 21

name = "Tai"

isStudent = true
```

Sau đó dùng `typeof` để in kiểu dữ liệu của từng biến.
==>
```js
let age =21 ;
let name= "Tai" ;
let isstudent= true ;
console.log(typeof age);
console.log(typeof name);
console.log(typeof isstudent);
```

## Bài 2

Cho đoạn mã:

```js
let a;
let b = null;
```

**Không chạy code**, hãy trả lời:

1. Giá trị của `a` là gì?
2. Giá trị của `b` là gì?
3. `a` và `b` khác nhau ở điểm nào?
		--> `undefined`, `null` , `.....`

## Bài 3
Tạo một Object tên `book` gồm:
- title
- author
- year
Sau đó in ra **title**.
==>
```js
//-Bai3
let book={
title: "nhung la thu ko gui" ,
author: "",
years: 1999 ,
};
console.log(book.title , book.years);
```
## Bài 4
Tạo Array:
```js
["Chrome","Firefox","Edge"]
```
Sau đó:
1. In phần tử thứ hai.
2. Thêm `"Safari"` vào cuối mảng.
3. In toàn bộ mảng.
```js
let arr=["Chrome","Firefox","Edge"] ;
arr.push("safari")
console.log(arr)
```
## Bài 5 (Quan trọng)
Không chạy code, hãy dự đoán kết quả:

```js
console.log(typeof 10);

console.log(typeof "10");

console.log(typeof false);

console.log(typeof null);

console.log(typeof []); ==> arrays

console.log(typeof {});  ==> object
```
Đây là bài kiểm tra rất tốt để xem bạn đã nắm được các kiểu dữ liệu và cũng sẽ giúp phát hiện một "cái bẫy" nổi tiếng của JavaScript. Chúng ta sẽ cùng phân tích sau khi bạn làm xong.