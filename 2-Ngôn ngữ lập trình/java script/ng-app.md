
[[ng-app draw]]  *<--nhấp vào đây*

### 📘 `ng-app` là gì?

- `ng-app` là **chỉ thị** (directive) của AngularJS.
- Khi thêm vào 1 thẻ HTML (ví dụ `<body ng-app>`), toàn bộ nội dung bên trong thẻ đó sẽ được AngularJS **biên dịch và quản lý**.
- AngularJS sẽ **thực thi mọi biểu thức** đặt trong dấu `{{ }}` như code JavaScript.
➡️ Vì vậy, nếu dữ liệu người dùng rơi vào `{{ }}`, attacker có thể chạy JS mà không cần `<script>` hay sự kiện.
---
🧪 Ví dụ
```html
<html ng-app>
<body>
  <h1>{{ 'Xin chào ' + name }}</h1>
</body>
</html>
```
- Nếu `name` được lấy từ URL và không lọc, attacker chèn:
  `?name={{constructor.constructor('alert(1)')()}}`
- AngularJS sẽ thực thi biểu thức trong `{{ }}`, gọi `constructor` để tạo hàm `alert(1)` và chạy.
---
Bài tập
Cho trang:
```html
<html ng-app>
<body>
  <p>{{ greeting }}</p>
  <script>
     var g = new URLSearchParams(location.search).get('greet');
     document.querySelector('p').innerHTML = '{{ ' + g + ' }}';
  </script>
</body>
</html>
```
1. Xác định source, sink.
2. Viết URL để hiện `alert(document.cookie)`.
3. Cách sửa: không dùng `innerHTML` kết hợp `{{ }}`.
👉 Làm xong nhắn thầy chữa.



### 📘2-Biểu thức AngularJS bị "sandbox" 
- AngularJS không cho phép gọi thẳng `alert()` vì biểu thức chạy trong một scope riêng, tách biệt với `window`.
- Muốn gọi hàm toàn cục, phải **thoát sandbox** bằng cách truy xuất đến `Function` constructor.

 🔗 Cách hoạt động
- Mọi object trong JavaScript đều có `.constructor`, thường trỏ tới `Function`.
- Từ một object bất kỳ (như `$on` – một service của Angular), lấy constructor 2 lần:  
  `$on.constructor` → `Function` → `$on.constructor.constructor` chính là `Function`.
- `Function('alert(1)')()` tạo và gọi hàm chứa `alert(1)`.

 🧪 Ví dụ so sánh
```html
<!-- Gõ vào ô search -->
{{alert(1)}}                    <!-- Không chạy, bị sandbox chặn -->
{{$on.constructor('alert(1)')()}}  <!-- Thoát sandbox, chạy alert -->
```

 📝 Bài tập nhanh
Tìm một object khác ngoài `$on` có sẵn trong scope (gợi ý: `toString` hoặc `this`) và viết payload ngắn hơn mà vẫn hiện `alert(1)`.

>tức là để thoát khỏi sandbox của angular js thì cần phải gọi hàm Function(...script..). mà để gọi tới Function thì mẹo là gọi constructor của 1 object


### 3-hiểu thêm về các object trong angular js
<u>🎒 Scope = cái túi đồ nghề</u>
- AngularJS tạo ra 1 **túi đồ nghề** (scope) cho mỗi vùng có `ng-app`.
- Trong túi có sẵn:
  - **Đồ AngularJS cho sẵn:** `$on`, `$watch`...
  - **Đồ JavaScript kế thừa:** `toString`, `constructor` (do mọi object đều có).
- Khi viết `{{ gì đó }}`, AngularJS thò tay vào túi lấy ra dùng.
➡ Vì vậy, `{{ $on }}`, `{{ toString }}` luôn có sẵn mà không cần em khai báo.
![[Pasted image 20260728182623.png|518]]
 🧪 Ví dụ
```html
<body ng-app>
  <p>{{ 'Túi có: ' + toString }}</p>
</body>
```
Hiển thị: `Túi có: function toString() { [native code] }`  
→ `toString` có sẵn trong túi.

Payload XSS:
```
{{ toString.constructor('alert(1)')() }}
```
Mượn `toString` lấy constructor, tạo hàm `alert(1)` và gọi.

<u> 📝 Bài tập</u>
1. Vào lab, mở Console (F12), gõ:
   ```js
   angular.element(document.body).scope()
   ```
   Xem trong túi có những gì.
2. Tìm 1 món đồ khác ngoài `toString`, `$on` có thể dùng để tạo payload XSS. Gợi ý: `valueOf`, `hasOwnProperty`.

![[Pasted image 20260728182545.png]]

--> đây là các *object* có thể dùng bên trong *{{....}}* . Chúng được đựng  trong *scope()*
