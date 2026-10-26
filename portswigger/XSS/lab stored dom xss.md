![[Pasted image 20261020232309.png]]

Bài này ngắn, lỗi chính nằm ở chỗ filter encode dấu ngoặc nhọn không đúng cách.

Mình vào lab, kéo xuống phần comment, điền name/email gì cũng được, còn phần comment thì dán payload:

```html
<><img src=1 onerror=alert(1)>
```

Submit comment xong là `alert(1)` nổ.

## Vì sao payload này chạy?

Trang web có cố gắng chặn XSS bằng cách dùng `replace()` để encode `<` và `>`. Nhưng vì tham số đầu tiên của `replace()` là string, nó chỉ thay **lần xuất hiện đầu tiên** thôi, không thay toàn bộ.

Nghĩa là nếu mình gửi thẳng:

```html
<img src=1 onerror=alert(1)>
```

thì dấu `<` đầu tiên bị encode, tag không còn hoạt động.

Còn với payload:

```html
<><img src=1 onerror=alert(1)>
```

thì cặp `<>` đầu tiên sẽ “ăn” hết lần replace đầu tiên. Kết quả đại khái là:

```html
&lt;&gt;<img src=1 onerror=alert(1)>
```

Phần `&lt;&gt;` chỉ hiện ra như text `<>`, còn `<img src=1 onerror=alert(1)>` phía sau vẫn là HTML thật. Khi ảnh `src=1` load lỗi, `onerror` chạy và gọi `alert(1)`.

Vì đây là stored DOM XSS nên comment được lưu lại, mỗi lần render comment là payload lại được đưa vào DOM và alert có thể nổ lại.


![[Pasted image 20261020232256.png]]

