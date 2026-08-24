### API trong code nhìn như nào
Chuẩn mẹ rồi. Đứng ở góc độ thằng dùng, gọi API bản chất đéo khác gì mày gọi một cái hàm, nhưng thay vì chạy trực tiếp trên RAM của máy mày thì nó chạy xuyên qua mạng Internet (thông qua HTTP Request).

Để tao vạch code ra cho mày xem. API nó có 2 mặt: thằng xài và thằng tạo.

Góc độ 1: Thằng xài API (Client)
Giả sử mày viết một đoạn code cần lấy dữ liệu thời tiết. Mày đéo cần tự đo, chỉ việc gọi API của dịch vụ thời tiết. Bằng JavaScript, nó trông như này:

```javascript
// Mày bắn 1 request (lời gọi) lên server của người ta
fetch('https://api.weather.com/v1/hanoi?apikey=mat_khau_cua_may')
  .then(response => response.json()) // Bọn nó trả về 1 cục dữ liệu chuẩn JSON
  .then(data => {
    // Mày bóc data ra xài
    console.log("Nhiệt độ hiện tại là: " + data.nhiet_do);
  });

```

Nhìn ở đây, API đúng nghĩa là một cái link URL kèm thông số để mày chọc vào lấy dữ liệu.

Góc độ 2: Thằng tạo ra API (Server)
Thế ở đầu bên kia, ruột cái API trông như nào? Giả sử mày làm backend bằng chính thằng Express.js vừa học ở trên. Tạo API thực chất là mày định nghĩa một cái đường dẫn để hứng cái request của tụi client.

```javascript
const express = require('express');
const app = express();

// Mày tạo ra 1 cái API endpoint (cái cổng)
app.get('/api/user/info', (req, res) => {
  // Khi có thằng client nào chọc vào đường dẫn này, code của mày sẽ chạy 
  // (ví dụ chui vào database lấy thông tin)
  const thongTin = {
    ten: "Tài",
    truong: "PTIT",
    nganh: "An toàn thông tin"
  };

  // Xong mày ném cục data đó trả về cho nó dưới dạng JSON
  res.json(thongTin);
});

// Mở server cổng 3000 chờ tụi nó gọi
app.listen(3000); 

```

Tóm lại cho dễ ngấm:
Phía thằng gọi: API là một cú bắn HTTP (fetch/curl/requests) ra ngoài.
Phía thằng nhận: API là một đoạn code đứng canh ở một đường dẫn URL nhất định, thấy ai gọi vào thì xử lý rồi nhổ data ra.

### các ứng dụng của API
Bỏ mẹ cái ví dụ nhà bếp 3 xu ấy đi. Nói ngôn ngữ system thực tế cho mày dễ thấm.

API (Application Programming Interface) bản chất nó là cái cầu nối, hay cái cổng giao tiếp để 2 hệ thống phần mềm hoàn toàn khác biệt có thể nói chuyện và truyền data cho nhau. Tụi nó đéo cần biết ruột gan code bên trong của thằng kia viết bằng cái mẹ gì, cứ giao tiếp chuẩn theo quy tắc của API là hiểu.

Ví dụ system 1: Thanh toán Shopee bằng MoMo
Khi mày mua hàng trên Shopee và chọn trả bằng ví MoMo, con app Shopee đéo thể nào có quyền thò tay vào database của MoMo để trừ tiền mày được.
Lúc này MoMo chìa ra một cái API thanh toán. Shopee chỉ việc đẩy request qua cái API đó: "Ê MoMo, tao có mã đơn hàng X, mày trừ thằng này 100k hộ tao".
MoMo nhận data qua API, tự xử lý các bước bảo mật, trừ tiền, xong quăng lại phản hồi qua API cho Shopee: "Tao trừ xong rồi đấy, mày chốt đơn cho nó đi".

Ví dụ system 2: Nút Đăng nhập bằng Google
Mày code một con web và muốn có nút Đăng nhập bằng Google. Đương nhiên Google đéo bao giờ đưa cục database chứa mật khẩu của user cho mày tự check.
Nó sẽ cấp cho web của mày một cái API. Khi user bấm nút, web mày gọi API: "Ê Google, xác thực hộ tao thằng này". Google tự hiện bảng đăng nhập, check xong xuôi sẽ báo lại qua API: "Check ok rồi, info thằng này là A, email là B, cho nó vào đi".

Tóm lại: API là cái cổng an toàn để các hệ thống độc lập nhờ vả và ném data cho nhau. Chốt thế hiểu chưa?