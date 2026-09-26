![[Pasted image 20260802192536.png]]
### phần path có ý nghĩa gì ??
*Toàn bộ các file mã khai thác*  đã *được tải sẵn và nằm trên ổ cứng máy tính* .Tuy nhiên, cột `Path` không hiển thị đường dẫn đầy đủ từ ổ đĩa.
Ví dụ với file `linux/remote/13853.pl`:
- Nó nằm bên trong thư mục chứa dữ liệu của Exploit-DB (thường gọi là thư mục gốc).
- Trên máy Kali Linux chuẩn, đường dẫn **thực tế đầy đủ** của nó sẽ là:
    `/usr/share/exploitdb/exploits/linux/remote/13853.pl`


### truy cập vào cái exploit ở phần /path
-truy cập vào exploit bằng các câu lệnh dưới đây
1. **Để xem nhanh nội dung (đọc code hoặc đọc hướng dẫn):**
    ```
    searchsploit -x 13853
    ```
2. **Để copy file đó ra thư mục bạn đang đứng (để sửa IP rồi chạy):**
    ```
    searchsploit -m 13853
    ```
    