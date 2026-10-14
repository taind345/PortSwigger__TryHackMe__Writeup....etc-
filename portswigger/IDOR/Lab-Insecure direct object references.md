![[idor-transcript.png]]
-mô tả lab: chức năng xem lại chat `/download-transcript` dùng `key` đoán được + không check quyền. Mình đổi key là đọc chat carlos.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/access-control/lab-insecure-direct-object-references

### hướng giải
- [ ] mình chat với bot rồi bấm xem transcript của mình: `/download-transcript?key=ABC...` (key của mình)
- [ ] mình thử bỏ key / đổi key bừa thì báo lỗi, nhưng mình nhớ đề bài gợi ý carlos cũng chat ở đây
- [ ] sau khi quan sát gói tin này, mình thấy key chỉ là số tăng dần / đoán được (vd brute 1..100 hoặc lấy từ JS). Mình brute key 1 lúc thì lòi ra transcript của carlos chứa password
- [ ] => không check key có thuộc về session mình không. Mình đọc được password carlos rồi login là solve

### giải
```bash
ffuf -u "https://LAB-ID.web-security-academy.net/download-transcript?key=FUZZ" -w numbers.txt -fr "Not found"
```
-sau khi quan sát kết quả này, mình thấy có 1 key trả về 200 chứa chat của carlos, trong đó có dòng `password: SECRET...`.
![[idor-transcript-carlos.png]]
-mình login `carlos:SECRET...` là solve.

> [!NOTE]
> -bài học của mình: key/reference phải random khó đoán (UUID) + vẫn phải check quyền. ID tuần tự là mời gọi brute.
