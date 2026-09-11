Trong lập trình thi đấu, **IR (Invalid Return)** — trên nhiều nền tảng còn được gọi là **NZEC (Non-Zero Exit Code)** — xảy ra khi chương trình của bạn kết thúc nhưng trả về cho hệ thống chấm một mã trạng thái khác `0`.

Dưới đây là 3 nguyên nhân phổ biến nhất kèm ví dụ thực tế:

==**1. Hàm chính không trả về 0 (Thường gặp ở C/C++)**==

Hệ thống chấm mặc định một chương trình chạy thành công (không có lỗi) phải báo cáo lại số `0`.

- **Ví dụ:** Bạn viết nhầm `return 1;`, `return -1;` hoặc dùng lệnh ngắt `exit(1);` ở cuối hàm `int main()`.
    
- **Cách sửa:** Luôn luôn chốt bằng `return 0;` ở cuối chương trình.
    

==**2. Gặp lỗi thực thi ngầm (Exception/Crash)**==

Khi chương trình đang chạy mà gặp một lỗi "chí mạng" không thể tự xử lý, nó sẽ tự sập và trả về một mã lỗi (khác 0), dẫn tới lỗi IR.

- **Ví dụ:**
    
    - ==**Chia cho 0==:** Có phép tính `x = 10 / 0`.
        
    - ==**Tràn bộ nhớ==:** Hàm đệ quy gọi lại chính nó vô hạn lần (Stack Overflow), hoặc truy cập vào vùng nhớ cấm (Segmentation Fault ở C++).
        
    - ==**Truy cập ngoài mảng:**== Khai báo mảng 5 phần tử nhưng cố tình gọi `a[10]`.
        

==**3. Lỗi thao tác Input/Output (Đặc biệt ở Python)**==

Đây là nguyên nhân phổ biến nhất khi code Python trên các trang chấm. Khi Input/Output gặp sự cố, Python ném ra Exception và lập tức thoát chương trình với mã lỗi.

- **Ví dụ:**
    
    - Cố ép ==kiểu sai==, giống hệt bài toán trước của bạn (gọi `int(" ")`).
        
    - Dùng lệnh `input()` để đọc dữ liệu khi test case đã hết (gây lỗi `EOFError` - End of File).
        

**Tóm tắt cách bắt bệnh nhanh:** Nếu dính IR, hãy kiểm tra ngay: **1. Đã `return 0` chưa? (với C++) -> 2. Có ép kiểu sai hoặc đọc dư Input không? (với Python) -> 3. Có mảng nào bị truy cập lố giới hạn không?**