### object 
-phần này mình đã học sơ qua trong [[datatype js#object]]
-tiếp tục học về object 

``` js
const dienThoai = {
    // Properties
    hang: "Apple",     
    gia: 20000000,     

    // Method
    goiDien: () => {   
        console.log("Đang kết nối...");
    }
};
// Cách sử dụng:
console.log(dienThoai.hang); // In ra: "Apple"
dienThoai.goiDien();         // In ra: "Đang kết nối..."
```
-> -**properties** : là thuộc tính, là biến thôi --> miêu tả đặc điểm của object 
-**method**: là hàm bên trong object --> miêu tả hành động của object 

### arrays

-phần này mình cũng đã học trong [[datatype js#arrays]] , ở đây có push(), pop()
-ok bây giờ sẽ học về **map** , **filter()**, **for each()**, trong arrays
- **map()**: cho ra mảng mới, ví dụ như mình muốn gấp đôi các phần tử của mảng chẳng hạn :)
``` js
const giamoi =giatien.map(gia=>gia*2)
// ben trong map() la mot ham
// gia=>gia*2 la mot ham tra ve gia tri gap doi
//cac phan tu cua giamoi deu di qua ham gia()
```
  -> *các phần tử lần lượt đi qua hàm giá ()*, *map() giúp các phần tử đi qua gia() và trả về phần tử mới*
  
- **`filter()`** - hàm này giúp <u>lọc phần tử </u>trong mảng 
```js
    // Chỉ lấy những giá lớn hơn 15
    const giaCao = giaTien.filter(gia => gia > 15); 
    // Kết quả: [20, 30]
```
--> *các phần tử lần lượt đi qua hàm giá * , fillter() lọc các phần tử theo gia()

- **`forEach()` - Duyệt qua:** Lặp qua từng phần tử <u>để làm một việc gì đó</u> (như in ra màn hình). **Không** tạo ra mảng mới.
```js
    giaTien.forEach(gia => console.log(`Giá là: ${gia}`));
```


==> *3 hàm trên đều có dạng (hàm=>{}). ta có thể  custom hàm bên trong tùy thích với 3 hàm này. *

-cuối cùng luyện tập thôi 
[[bai tapjs#bài tập object và arrays]]
*sai lầm khi mình đã hiểu cái object này là class :D , thực ra nó chỉ là object thôi là định nghĩa luôn cho một vật, chứ không phải là một cái khuôn như class*
