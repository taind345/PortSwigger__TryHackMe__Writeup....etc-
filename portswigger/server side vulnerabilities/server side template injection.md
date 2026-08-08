
**1. Bản chất lỗ hổng**
Thay vì xử lý dữ liệu bạn nhập vào như một "đoạn văn bản vô hại", lập trình viên lại nhúng trực tiếp nó vào mã nguồn Template. Template Engine (bộ xử lý khuôn mẫu) bị đánh lừa, tưởng đó là code của hệ thống và đem ra **thực thi**. Kết quả: Kẻ tấn công chiếm quyền điều khiển máy chủ.
**2. Vòng đời khai thác (3 Bước cốt lõi)**
- **Bước 1: Phát hiện (Detect)**
    - Chèn các biểu thức toán học đặc trưng của nhiều loại engine khác nhau: `{{7*7}}`, `${7*7}`, `<%= 7*7 %>`, `*{7*7}`.
    - Nếu màn hình trả về `49`, bạn đã tìm thấy "lỗ hổng".
- **Bước 2: Nhận diện Engine (Identify)**
    - Có hàng chục loại Template Engine (Jinja2, Twig, FreeMarker...). Mỗi loại dùng một cú pháp lập trình khác nhau. Bạn không thể dùng code của Jinja2 đi đánh sập hệ thống chạy Twig.
    - Dựa vào việc server phản hồi đúng hay báo lỗi với từng loại cú pháp, bạn sẽ khoanh vùng được chính xác Engine đang sử dụng.
- **Bước 3: Khai thác (Exploit)**
    - Vượt ra khỏi phạm vi (Sandbox) của Template.
    - Mục tiêu tối thượng: Tìm cách gọi các hàm hệ thống (System calls) để đọc file nhạy cảm (LFI) hoặc chạy lệnh điều hành từ xa (RCE).
**🎯 Bài tập tư duy cho bạn:**
Trong Bước 2, để phân biệt các loại Engine, giới bảo mật thường dùng kỹ thuật thử sai. Giả sử bạn gửi payload: `{{7*'7'}}`
- Nếu kết quả trả về là `49` (7 nhân 7).
- Nếu kết quả trả về là `7777777` (chuỗi '7' được lặp lại 7 lần).
Hành vi nhân bản chuỗi `7777777` là đặc trưng của ngôn ngữ lập trình nào? Và nếu xác định được nền tảng bên dưới chạy ngôn ngữ đó, bạn sẽ nhắm tới việc gọi module (thư viện) mặc định nào của nó để có thể thực thi các lệnh hệ điều hành như `ls` hay `cat`?