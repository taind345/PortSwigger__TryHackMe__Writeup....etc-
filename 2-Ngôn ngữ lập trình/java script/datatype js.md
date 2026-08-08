đại khái có các kiểu dữ liệu chính cần học như 
![[Pasted image 20260712224154.png|459]]
[[baitap datatype js]]
### Number 
-java script coi tất cả số thực ,số nguyên, đều là *number*
-ví dụ:
```js
let a = 10;
let b = 3.14;
let c = -100;

let num=3 ;
num=40 ;
console.log(num) ;
let num2=3.2 ;
console.log(typeof num2) ;
```
![[Pasted image 20260712224659.png]]
### String 
-dùng *let* để khai báo biến string
```js
let ten="Tai" ;
let ho="Nguyen" ;
console.log(ten+" "+ho) ;
// => Tai Nguyen
```
### boolean
-kiểu boolean chỉ có hai giá trị *true/false*
```js
let login =true ;
let username="admin" ;
console.log(typeof login) ;
//==> boolean
```

### undefined 
-là biến chưa gán giá trị thôi :))
```js
let age;
console.log(age);
// -> undefined
```
### null
-giá trị *null* khác với *undefined*
```js
let age ;
let hihi =null ;
console.log(age) ;
// -> undefined
```

### object 
-Cái thằng này nó là *object* như kiểu trong java thôi, nó là 1 lớp đối tượng

```js 
let thongtinSV ={
	name : "Tai" ,
	age : 20 ,
	hometown : "ThaiBinh" ,
};

// sua thong tin
thongtinSV.name="TaiNguyen"
thongtinSV.age= 20
console.log(thongtinSV.name)
```

### arrays
-nó là mảng thôi , ko có j đặc sắc 
-ở đây cần nhớ thêm *push* và *pop*
```js
let number =[1,2,3] ;
console.log(number[0]) ;

let fruit=["tao", "chuoi" , "xoai"]
fruit.push("dudu");
fruit.pop();
```

### Kiến thức Pentest

Bạn sẽ gặp rất nhiều đoạn như:

```js
const user = {
    username: "admin",
    role: "administrator"
};
```

```js
const data = [
    "apple",
    "orange"
];
```

```js
const json = {
    token: "...",
    email: "...",
    id: 100
};
```

### Tóm tắt

|Kiểu|Ví dụ|
|---|---|
|Number|`10`|
|String|`"Tai"`|
|Boolean|`true`|
|undefined|`let x;`|
|null|`let x = null;`|
|Object|`{name:"Tai"}`|
|Array|`[1,2,3]`|

