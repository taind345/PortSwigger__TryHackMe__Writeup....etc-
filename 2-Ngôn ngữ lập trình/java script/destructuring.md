## 3. Destructuring

Object:

```js 
let user = {
    name: "Tai",
    age: 21
};

let { name, age } = user;

console.log(name);
console.log(age);
```

Thay vì:

```
user.name
user.age
```

Array:

``` js
let numbers = [10, 20];
let [a, b] = numbers;
```

```
a = 10
b = 20
```

## 4. Spread `...`

*Mở rộng dữ liệu:*

``` js
let a = [1, 2];
let b = [...a, 3, 4];

console.log(b);
```

```
[1, 2, 3, 4]
```

Object:

```js
let user = {
    name: "Tai"
};

let newUser = {
    ...user,
    age: 21
};
```

## 5. Rest `...`
Gom nhiều giá trị thành Array:
```js
function sum(...numbers) {
    console.log(numbers);
}

sum(1, 2, 3);
//[1,2,3]
//-> giá trị của  nhiều tham số được gọm lại thành arrays number 
```

### *-->Ghi nhớ*
**Spread:** mở ra.
```js
//...array → các phần tử
arr=[1,2,3,5];
//..arr == 4 phần tử 1, 2 ,3,5
```
**Rest:** gom lại.
```js
//...args → một array
như ví dụ trên --> 3 số 1, 2,3 được gom thành 1 mảng
```
*--> như vậy dấu ... sẽ có vai trò khác nhau nếu sau nó là 1 <u>arrays</u> hay 1 <u>argument</u>*
