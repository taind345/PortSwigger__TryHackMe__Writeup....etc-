#portswinger 
# cách nhận biết
- [ ] đoán url
- [ ] dựa vào mã java script trả về
- [ ] Một số ứng dụng kiểm soát truy cập dựa vào thông tin lưu ở phía người dùng
	- như cookie
	- ...
 - [ ] leo thang đặc quyền ngang
# khai thác
- [ ] khai thác trên url 
- [ ] dùng burp để  tìm ra tham số trên url--> từ đó thay tham số--> đề vào được amin login--> từ đó xem đc password
## 1. Unprotected Functionality
Lỗi: Giấu URL quản trị thay vì phân quyền hệ thống (Security by obscurity). Hậu quả: Dễ bị lộ URL qua mã nguồn client (JavaScript).
## 2. Parameter-based Access Control
Lỗi: Server tin tưởng dữ liệu client gửi lên (Cookie, URL param như ?admin=true). Hậu quả: Hacker dễ dàng tự sửa đổi tham số để nâng quyền.
## 3. Leo thang đặc quyền ngang
Lỗi: Khai thác IDOR. Đổi tham số ID tài khoản (ví dụ: ?id=123 thành ?id=124). Hậu quả: Trộm dữ liệu của người dùng cùng cấp. Dùng GUID thay vì số vẫn có rủi ro bị lộ.
## 4. Leo thang từ Ngang sang Dọc
Chiến thuật: Dùng lỗ hổng leo thang ngang (như IDOR) nhưng nhắm mục tiêu vào tài khoản Admin. Hậu quả: Chiếm được tài khoản Admin, đạt toàn quyền kiểm soát hệ thống (trở thành leo thang dọc).