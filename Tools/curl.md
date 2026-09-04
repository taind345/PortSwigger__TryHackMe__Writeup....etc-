Đi làm pentest hay cày lab thì nhớ ngần này cờ của curl là đủ xài:

* -X: Chọn phương thức HTTP (GET, POST, PUT, DELETE). Mặc định không ghi gì là GET.
* -d: Nhét dữ liệu vào body (form data, chuỗi text, JSON). Cứ cắm cờ này là curl tự hiểu chuyển sang gửi POST.
* -H: Kẹp thêm header tùy chỉnh (Content-Type, Authorization: Bearer, User-Agent...).
* -b: Gửi cookie lên server (truyền chuỗi "name=value" hoặc trỏ tới file cookie).
* -c: Hứng cookie server nhả về rồi lưu vào file để dùng cho các request sau.
* -L: Tự động nhảy theo chuyển hướng (khi gặp mã 301, 302 redirect).
* -i: In cả HTTP header của response ra màn hình để soi mã phản hồi (status code) và cookie mới.
* -I: Chỉ lấy header (gửi HEAD request), bỏ qua phần nội dung body.
* -k: Bỏ qua kiểm tra chứng chỉ HTTPS/SSL (cực kỳ cần khi gặp lab dùng chứng chỉ tự ký hoặc lỗi cert).
* -x: Đẩy request qua proxy trung gian (tiện nhất là bắn thẳng qua Burp Suite: `-x [http://127.0.0.1:8080](http://127.0.0.1:8080)`).
* -v: Chế độ verbose, in toàn bộ chi tiết quá trình gửi/nhận để debug xem lỗi ở đâu.
* -s: Chạy ngầm, tắt thanh tiến trình download/upload cho đỡ rác màn hình terminal.
* -o: Ghi kết quả trả về ra file thay vì in thẳng ra màn hình.