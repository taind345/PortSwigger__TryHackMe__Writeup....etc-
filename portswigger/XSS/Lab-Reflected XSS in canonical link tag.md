![[Pasted image 20261021083245.png]]

Trang chủ phản chiếu tham số URL vào thẻ `<link rel="canonical" href='...'>`. Ứng dụng **escape `<` và `>`** nên không thể inject tag mới, **nhưng không escape dấu nháy đơn `'`** → có thể thoát khỏi attribute `href` và thêm attribute tùy ý vào chính thẻ `<link>`.


HTML sinh ra có dạng:
```html
<link rel="canonical" href='https://LAB-ID.web-security-academy.net/?INPUT'>
```

- `href` được bao bằng **nháy đơn `'`**.
- ==Input nằm ngay bên trong `href`.==
- `'` không bị lọc → **điểm break-out**.
![[Pasted image 20261021172350.png]]
## Payload

```
'accesskey='x'onclick='alert(1)
```

URL-encode khi đưa lên thanh địa chỉ:

```
%27accesskey=%27x%27onclick=%27alert(1)
```

URL đầy đủ:

![[Pasted image 20261021172138.png]]



## Cú pháp sau khi render

```html
<link rel="canonical" href='https://LAB-ID.web-security-academy.net/?' accesskey='x' onclick='alert(1)'>
```

Đối chiếu từng phần:

| Payload              | Tác dụng                                                    |
| :------------------- | :---------------------------------------------------------- |
| `'` đầu tiên         | Đóng `href` sớm                                             |
| `accesskey='x'`      | Gán phím tắt **X** cho thẻ `<link>`                         |
| `onclick='alert(1)'` | Handler chạy khi element bị "click"                         |
| `'` cuối             | Khớp với dấu `'` đóng `href` gốc → HTML cân bằng, không lỗi |

## Tại sao khai thác được

- ==`accesskey` là **global attribute**== — hoạt động trên mọi element, kể cả `<link>` (element vốn không hiển thị).
- Khi ==user bấm tổ hợp phím tắt, **Chrome** kích hoạt **click event**== trên element được gán `accesskey` → `onclick` chạy → `alert(1)`.
- Lab cung cấp sẵn các tổ hợp phím cho simulated victim:
  - **Windows:** `ALT+SHIFT+X`
  - **macOS:** `CTRL+ALT+X`
  - **Linux:** `Alt+X`
- Chỉ **Chrome** giải được vì các browser khác không hỗ trợ `accesskey` trên `<link>` (đúng như note của lab).

> [!NOTE]
> 1. **Escape không đầy đủ = lỗ hổng.** Escape `<` `>` mà bỏ quên `'` (hoặc `"`) là sai lầm kinh điển. Mỗi ngữ cảnh (HTML text, attribute, JS, URL…) cần bộ escape riêng.
> 2. **Không cần tag mới vẫn XSS được.** Chỉ cần thêm attribute vào thẻ có sẵn là đủ nếu attribute đó kích hoạt được hành vi.
> 3. **`accesskey` là vũ khí ngầm.** Nó biến bất kỳ element nào (kể cả vô hình) thành "nút bấm phím tắt" — kết hợp với `onclick` tạo ra XSS không cần tương tác chuột, chỉ cần gõ phím.
> 4. **Phụ thuộc browser.** Luôn kiểm tra compatibility — đây là lý do lab ghi rõ "chỉ Chrome".


![[Pasted image 20261021172155.png]]