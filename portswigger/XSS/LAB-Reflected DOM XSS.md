
Bài lab này minh họa một lỗ hổng DOM kiểu reflected (phản chiếu). Lỗ hổng dạng này xảy ra khi máy chủ xử lý dữ liệu từ yêu cầu rồi trả lại chính dữ liệu đó trong nội dung phản hồi. Một đoạn script trên trang sau đó xử lý dữ liệu được trả về một cách thiếu an toàn, và cuối cùng đưa dữ liệu vào một vị trí nguy hiểm (dangerous sink).

Để hoàn thành bài lab, bạn cần tạo một đoạn mã chèn (injection) sao cho gọi được hàm `alert()`

