DNS record (bản ghi DNS) có thể hiểu đơn giản là một dòng thông tin hướng dẫn lưu trên DNS server, dùng để chỉ dẫn cho máy tính và trình duyệt biết cách xử lý khi ai đó gõ một tên miền (domain) trên internet.
Nếu coi hệ thống DNS là một cuốn danh bạ điện thoại:
- Tên miền (ví dụ: google.com) là tên người trong danh bạ.
- DNS record chính là thông tin chi tiết được ghi bên cạnh tên đó (*như số điện thoại, địa chỉ nhà, email...*).
![[Pasted image 20260808045201.png|419]]
Máy tính không thể tự kết nối trực tiếp bằng tên chữ như tryhackme.com mà chỉ có thể giao tiếp qua các dãy số IP. Khi bạn gõ một tên miền, DNS record sẽ giúp máy tính biết chính xác IP của máy chủ chứa trang web đó là gì, hoặc thư từ gửi đến tên miền đó sẽ chuyển về máy chủ email nào.

*Một số loại DNS record phổ biến nhất bao gồm:*
- Record A: Trỏ tên miền về một địa chỉ IPv4 cụ thể của máy chủ web.
- Record AAAA: Tương tự record A, nhưng trỏ về địa chỉ IPv6.
- Record CNAME: Trỏ một tên miền này sang một tên miền khác (tạo tên biệt danh).
- Record MX: Chỉ định máy chủ email nào chịu trách nhiệm nhận thư cho tên miền.
- Record TXT: Chứa các chuỗi văn bản dùng cho nhiều mục đích khác nhau, phổ biến nhất là để xác minh quyền sở hữu tên miền hoặc cấu hình bảo mật chống giả mạo email.
 *=> nếu ví dns record như một thông tin ghi kèm tên trong danh bạ, thì có các dạng như điện thoại, địa chỉ nhà, quê quán,....*

### ví dụ![[Pasted image 20260808051021.png]]
khi mà mình truy cập vào cái domain này=> thì trình duyệt của mình sẽ nhìn vào các *dns record* để biết xem cần truy cập vào địa chỉ ip nào