thứ tự này là do mình tự xếp từ dễ tới khó.
-> nên mình học theo đúng list này là đỡ ngợp nhất.

### 0. lý thuyết nền
- [[lys thuyet XSS]] -> file tổng hợp của mình, đọc trước.
- [[Xss là gì]] -> định nghĩa gốc từ portswigger.
- [[reflect XSS]] -> flow reflected.
- [[stored xss]] -> flow stored, entry/exit point.
- [[DOM based Xss]] -> source/sink, jQuery, AngularJS.
- [[CSP]] + [[Dangling Markup]] -> khi script bị chặn thì làm gì.
- [[phòng chống xss]] + [[xss-filter bypass]] -> encode + bypass.
- [[các cách phát hiện lỗ hổng XSS]] -> quy trình test 5 bước.

==> [[0-XSS portswigger lab]]
### 1. lab dễ: chèn thẳng không filter
- [[Lab-Reflected XSS basic]] -> search phản chiếu luôn, bắn `<script>` là ăn.
- [[Lab-Stored XSS basic]] -> comment lưu lại, ai mở cũng dính alert.
- [[Lab-DOM document.write]] -> source `location.search`, sink `document.write`.
- [[Lab-DOM innerHTML]] -> source `location.search`, sink `innerHTML`, phải dùng event handler.

### 2. lab trung bình: có encode / filter
- [[Lab-Reflected attribute encoded]] -> input chui vào `value="..."`, ngoặc nhọn bị encode nên phải ở lại trong thẻ.
- [[Lab-XSS filter bypass event handler]] -> `script` + `onerror` bị chặn, đổi sang `svg` + event lạ.

### 3. lab khó: CSP chặn script
- [[Lab-Dangling markup]] -> CSP chặn hết script nên dùng thẻ chưa đóng để trộm CSRF token.

-> học xong 1 lab nào thì quay lại [[lys thuyet XSS]] đối chiếu là nhớ lâu nhất.

# LAB nâng cao
- [[Lab-AngularJS sandbox escape]] -> thấy `{{7*7}}` render 49 là dính template, dùng `{{$on.constructor('alert(1)')()}}` để thoát sandbox. Nhớ check version Angular trước.


