
### hướng giải
Để giải bài thực hành này, bạn cần ép cơ sở dữ liệu **"nói hớ"** mật khẩu thông qua thông báo lỗi. Bạn làm theo các bước sau trong **Burp Suite Repeater**:

1. **Gửi yêu cầu:** Tìm request có `TrackingId` cookie.
    
2. **Gây lỗi:** Thay `TrackingId` bằng:
    
    `' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--`
    - _Kết quả:_ Thông báo lỗi sẽ hiện tên người dùng đầu tiên (thường là `administrator`).    
1. **Lấy mật khẩu:** Sau khi biết tên, thay bằng lệnh:
    
    `' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--`
    
    - _Kết quả:_ Thông báo lỗi sẽ "vô tình" in ra mật khẩu của `administrator`.
        
2. **Đăng nhập:** Dùng mật khẩu vừa lấy được để đăng nhập vào trang web là xong.
    

**Lưu ý:** Nếu báo lỗi giới hạn ký tự, hãy xóa giá trị cũ của `TrackingId` trước khi chèn mã tấn công vào.

![[Pasted image 20260629090228.png]]
=> thông báo lỗi lộ liễu => ta biết được bảng ***users*** tồn tại

![[Pasted image 20260629104607.png]]
=> mã lỗi hiển thị luôn bảng administrator
![[Pasted image 20260629104934.png]]
=> mã lỗi hiển thị luôn mật khẩu
```markdown
?? Vai trò của CAST ở đây là gì??
-CAST khiến ép cái câu truy vấn SELECT... FROM trả về *lỗi*==> từ lỗi ta có thể biết được thông tin về cái đoạn SELECT... mà ko cần nhiều công sức mò mẫm

```

```markdown
?? có cách nào thay thế cho CAST ko ??
Có. Ngoài `CAST()`, bạn có thể dùng **các hàm ép kiểu khác** hoặc **kỹ thuật toán học** để ép máy chủ báo lỗi. Dưới đây là các phương án thay thế phổ biến:

1. Dùng `CONVERT()`

Đây là cách phổ biến nhất vì nó tương tự `CAST`.

- **Cú pháp:** `CONVERT(int, (SELECT password FROM users LIMIT 1))`
    
- **Tác dụng:** Kết quả y hệt `CAST`. Một số hệ quản trị cơ sở dữ liệu (như SQL Server) ưa chuộng lệnh này hơn.
    

2. Dùng phép tính toán học (Arithmetic Error)

Bạn không nhất thiết phải ép kiểu, bạn có thể ép máy chủ thực hiện một phép tính **bất khả thi**.

- **Cú pháp:** `AND 1=(SELECT password FROM users LIMIT 1)/0`
    
- **Cơ chế:** Chia cho số 0 (`/0`) là hành động gây lỗi "Divide by zero" (chia cho 0). Khi lỗi này xảy ra, máy chủ thường sẽ in kèm dữ liệu của truy vấn đó trong thông báo lỗi.
    

3. Dùng hàm xử lý chuỗi (String Manipulation)

Ép hệ thống thực hiện một hàm không thể xử lý trên dữ liệu đó.

- **Cú pháp:** `AND 1=(SELECT UPPER(password) FROM users LIMIT 1)` (nếu cột đó không phải kiểu chuỗi) hoặc các hàm đặc thù khác tùy loại Database.
    

Tư duy rút ra:

Mục tiêu cuối cùng của bạn không phải là dùng `CAST`, mà là **"làm cho truy vấn bị lỗi một cách có kiểm soát"**.

- Nếu `CAST` bị chặn, hãy thử **chia cho 0**.
    
- Nếu chia cho 0 bị chặn, hãy thử **ép kiểu dữ liệu sai**.
    
- **Nguyên tắc:** Bất cứ thứ gì làm máy chủ phải "đầu hàng" và in ra thông báo lỗi chứa kết quả truy vấn đều là công cụ tốt.
    

Bạn đang gặp lỗi "CAST bị chặn" hay chỉ muốn tìm thêm giải pháp để mở rộng kiến thức
```
