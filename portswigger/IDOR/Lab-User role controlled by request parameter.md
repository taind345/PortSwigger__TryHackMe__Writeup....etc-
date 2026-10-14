![[idor-role-cookie.png]]
-mô tả lab: quyền admin nằm trong cookie mình sửa được. Login `wiener:peter` rồi sửa cookie là lên admin.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/access-control/lab-user-role-controlled-by-request-parameter

### hướng giải
- [ ] mình login `wiener:peter`, sau đó quan sát cookie này, mình thấy `Admin=false`
- [ ] mình sửa thành `Admin=true` rồi GET lại `/my-account`, mình thấy hiện link admin
- [ ] => server tin cookie client gửi. Mình vào admin xóa carlos là solve

### giải
```
GET /my-account
Cookie: session=...; Admin=false -> 200 trang thường
Cookie: session=...; Admin=true -> 200 hiện Admin panel
GET /admin/delete?username=carlos -> 302
```
![[idor-role-admin-true.png]]

> [!NOTE]
> -bài học của mình: quyền phải lưu ở server (session store), không bao giờ để client tự khai role.
