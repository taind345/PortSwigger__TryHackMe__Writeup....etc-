This lab contains a DOM-based cross-site scripting vulnerability in a AngularJS expression within the search functionality.

AngularJS is a popular JavaScript library, which scans the contents of HTML nodes containing the `ng-app` attribute (also known as an AngularJS directive). When a directive is added to the HTML code, you can execute JavaScript expressions within double curly braces. This technique is useful when angle brackets are being encoded.

To solve this lab, perform a cross-site scripting attack that executes an AngularJS expression and calls the `alert` function.
*đọc thêm lý thuyết ở đây*   [[DOM based Xss#Những sink nguy hiểm từ thư viện phổ biến]]

<u>Bài này mình xem lời giải</u>
### nhận biết lỗ hổng
-search trong *source* ko thấy
-dùng burpsite; sau đó nhập vào thanh tìm kiếm, thấy server trả về mã js 
*-->* lý do mình ko tìm được là do , "ng-app" nằm trong body của html , nên khi mình mò trong script nó ko ra, đọc lại phần lsy thuyết bên trên để hiểu hơn.[[ng-app]]
![[Pasted image 20260728173918.png]]
thẻ body ng-app nhận input từ search, do đó khi ta chèn input {{script}} thì câu lệnh bên trong sẽ thực thi
![[Pasted image 20260728174101.png]]



*thoát khỏi sandbox của agular js để thực thi hàm alert()*
[[ng-app#📘 Lý thuyết Biểu thức AngularJS bị "sandbox" (hộp cát)]]
-> khi ta gọi hàm bên dưới , nó sẽ == Function(alert(1))
![[Pasted image 20260728180005.png]]
>Không em. **`print` không phải object**, cũng không có sẵn trong scope của AngularJS.  
Muốn thoát sandbox, phải móc vào **object có sẵn** trong scope (như `$on`, `this`, `toString`...) để lấy `constructor`.

payload
```js
{{$on.constructor('alert(1)')()}}

== {{ Function(alert(1))() }}
```


### hiểu thêm về các object trong angular js
 <u>📘 Lý thuyết: Scope trong AngularJS</u>
- **Scope** là object nơi AngularJS lưu các biến/hàm mà view (`{{ }}`) có thể truy cập.
- Mỗi ứng dụng có `$rootScope`, các controller tạo scope con kế thừa từ nó.
- Khi AngularJS tính `{{ expression }}`, nó chạy trong scope hiện tại.
- Mọi object JavaScript đều kế thừa `toString`, `constructor` từ `Object.prototype` → luôn có sẵn.
- `$on`, `$watch`... là các method được AngularJS gắn vào scope → cũng luôn có.

<u> 🧪 Ví dụ</u>
```js
// Scope mặc định có sẵn:
$rootScope.$on          // method đăng ký sự kiện
this.toString           // method từ Object.prototype
this.constructor        // trỏ tới Function constructor
```
➡️ Vì vậy payload `{{ toString.constructor('alert(1)')() }}` thoát sandbox được.

<u>📝 Bài tập</u>
1. Mở console trên lab AngularJS, gõ `angular.element(document.body).scope()` để xem toàn bộ scope object.
2. Tìm ít nhất 2 method khác (ngoài `$on`, `toString`) có sẵn trong scope và thử tạo payload khai thác tương tự.
3. Giải thích vì sao không cần khai báo vẫn dùng được `this` trong `{{ }}`.

👉 Làm xong báo thầy nhé.