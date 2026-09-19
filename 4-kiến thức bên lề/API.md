### API trong code nhìn như nào
Chính xác. Đứng ở góc độ người dùng, gọi API bản chất tương tự như việc bạn gọi một hàm trong lập trình, nhưng thay vì chạy trực tiếp trên RAM của máy bạn thì nó gửi yêu cầu xuyên qua mạng Internet (thông qua HTTP Request) đến máy chủ khác rồi nhận kết quả trả về.

Để dễ hình dung qua code, API có 2 khía cạnh chính: phía sử dụng (Client) và phía cung cấp (Server).

**Góc độ 1: Phía sử dụng API (Client)**
Giả sử bạn viết một đoạn code cần lấy dữ liệu thời tiết. Bạn không cần tự đo đạc nhiệt độ, chỉ việc gọi API của dịch vụ thời tiết. Bằng JavaScript, cú pháp sẽ như sau:

```javascript
// Gửi một request (lời gọi) lên server của dịch vụ
fetch('https://api.weather.com/v1/hanoi?apikey=mat_khau_cua_ban')
  .then(response => response.json()) // Phía dịch vụ trả về dữ liệu chuẩn JSON
  .then(data => {
    // Trích xuất dữ liệu để sử dụng
    console.log("Nhiệt độ hiện tại là: " + data.nhiet_do);
  });
```

Ở góc độ này, API đóng vai trò là một URL kèm các tham số để ứng dụng gửi yêu cầu và nhận dữ liệu về.

**Góc độ 2: Phía cung cấp API (Server)**
Ở đầu bên kia, cấu trúc của API trên server như thế nào? Giả sử bạn xây dựng backend bằng Express.js. Tạo API thực chất là định nghĩa một endpoint (đường dẫn) để tiếp nhận các request từ phía client:

```javascript
const express = require('express');
const app = express();

// Tạo ra một API endpoint (cổng giao tiếp)
app.get('/api/user/info', (req, res) => {
  // Khi client gọi vào đường dẫn này, code xử lý sẽ chạy 
  // (ví dụ: truy vấn cơ sở dữ liệu để lấy thông tin)
  const thongTin = {
    ten: "Tài",
    truong: "PTIT",
    nganh: "An toàn thông tin"
  };

  // Trả dữ liệu về cho client dưới định dạng JSON
  res.json(thongTin);
});

// Khởi chạy server ở cổng 3000 để lắng nghe request
app.listen(3000); 
```

**Tóm lại:**
- Phía gọi (Client): API là một HTTP request (fetch, axios, curl, python requests...) gửi ra ngoài.
- Phía nhận (Server): API là một đoạn mã lắng nghe tại một URL nhất định, khi nhận được request hợp lệ sẽ xử lý nghiệp vụ và trả về dữ liệu tương ứng.

### Các ứng dụng thực tế của API
Hãy nhìn vào các bài toán hệ thống thực tế để thấy rõ vai trò của API:

API (Application Programming Interface) bản chất là cầu nối hay chuẩn giao tiếp giúp hai hệ thống phần mềm độc lập có thể trao đổi dữ liệu an toàn với nhau. Hai hệ thống không cần biết chi tiết mã nguồn nội bộ của nhau được viết bằng ngôn ngữ hay công nghệ gì, chỉ cần tuân thủ đúng định dạng và quy tắc của API là có thể làm việc cùng nhau.

**Ví dụ 1: Thanh toán Shopee bằng ví MoMo**
Khi bạn mua hàng trên Shopee và chọn thanh toán bằng MoMo, ứng dụng Shopee không thể và không được phép can thiệp trực tiếp vào cơ sở dữ liệu của MoMo để trừ tiền.
Thay vào đó, MoMo cung cấp một API thanh toán an toàn. Shopee chỉ cần gửi yêu cầu qua API: "MoMo hãy trừ 100.000đ cho đơn hàng mã số X của người dùng này".
MoMo tiếp nhận dữ liệu qua API, tự thực hiện các bước xác thực, kiểm tra số dư, trừ tiền, rồi gửi phản hồi kết quả về cho Shopee: "Giao dịch thành công, Shopee có thể hoàn tất đơn hàng".

**Ví dụ 2: Tính năng Đăng nhập bằng Google (OAuth / SSO)**
Khi bạn phát triển một website và muốn có nút "Đăng nhập bằng Google", Google chắc chắn sẽ không bao giờ cung cấp database chứa thông tin mật khẩu của người dùng cho website của bạn.
Google sẽ cấp một chuẩn API xác thực. Khi người dùng bấm nút, website gọi API của Google để yêu cầu xác thực. Google hiển thị giao diện đăng nhập bảo mật cho người dùng, sau khi người dùng xác thực thành công, Google sẽ phản hồi qua API: "Xác thực thành công, thông tin tài khoản là Tên A, Email B, bạn có thể cấp quyền truy cập".

**Tóm lại:** API chính là cổng kết nối an toàn và tiêu chuẩn giúp các hệ thống phần mềm độc lập giao tiếp và tích hợp dữ liệu với nhau một cách tin cậy.