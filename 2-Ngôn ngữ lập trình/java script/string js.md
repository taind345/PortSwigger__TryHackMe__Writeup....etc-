-phần này mình sẽ học về các cách xử lý xâu trong js
- **includes('siuuu')** : tìm kiếm chữ siuuu có tồn tại trong chuỗi ko
``` js
let s="siuuuu" ;
if(s.includes("7 cho")){
	console.log("ton tai 7 cho trong xau s)
}
```
- **indexOf('siuu')** : trả về vị trí của từ siuuu
``` js
"anh 7 siuu khi ghi ban".indexOf('siuuu')
//-> tra ve 6
```
-  **slice(bắt_đầu, kết_thúc)** Cắt lấy một khúc. **Đặc biệt:** Hỗ trợ số âm để đếm ngược từ cuối chuỗi lên (rất hay dùng).
```js
"0912345678".slice(-3) //-> `"678"` (Lấy 3 số cuối).
```
- **`substring(bắt_đầu, kết_thúc)`:** Giống `slice` nhưng không hiểu số âm. _Lời khuyên:_ Cứ dùng `slice` cho linh hoạt.

- **`replace('cũ', 'mới')`:** Thay thế chữ. (Chỉ thay thế điểm trùng khớp đầu tiên. Muốn thay hết thì dùng `replaceAll()`).
``` js
"Chào anh".replace("anh", "chị") //-> `"Chào chị"`
```

- **`split('ký_tự')`:** Cắt nát **Chuỗi -> Mảng** dựa vào một điểm nhận diện.
    ``` js
    "Cam,Táo,Lê".split(",") //-> ["Cam", "Táo", "Lê"]
    ```
          
- **`join('ký_tự')`:** Gom **Mảng -> Chuỗi** _(Lưu ý: Đây thực chất là hàm của Array, nhưng luôn đi thành cặp với split)._
```js
["HTML", "CSS"].join(" - ") // -> `"HTML - CSS"`
```


bài tập 
[[bai tapjs#bài tập xử lý xâu]]
