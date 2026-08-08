### trim o dia chua fefora
**Có, Fedora có sẵn và tự động bật cơ chế TRIM cho ổ SSD.**
Fedora xử lý TRIM thông qua 2 cơ chế hoạt động song song:
- **Với định dạng Btrfs (mặc định của Fedora):** Sử dụng tùy chọn mount `discard=async`. Hệ thống tự động gửi lệnh TRIM ngầm theo thời gian thực ngay khi xóa tệp tin mà không ảnh hưởng đến hiệu năng.
- **Với các định dạng khác (ext4, xfs...):** Fedora bật sẵn dịch vụ `fstrim.timer` của systemd, tự động quét và thực hiện TRIM toàn bộ các phân vùng SSD định kỳ **1 lần/tuần**.
**Các lệnh kiểm tra nhanh:**
- **Kiểm tra trạng thái TRIM định kỳ:**
    Bash
    ```
    systemctl status fstrim.timer
    ```
- **Chạy TRIM thủ công ngay lập tức:**
    Bash
    ```
    sudo fstrim -av
    ```
![[Pasted image 20260728233950.png]]

---
### trim o D
Để TRIM riêng cho phân vùng/ổ đĩa tên **DATa** trên Fedora, bạn thực hiện như sau:
 **Bước 1: Xác định đường dẫn mount (gắn đĩa)**
Mở Terminal và tìm nơi ổ đĩa đang được hệ thống gắn vào:
Bash
```
lsblk -f
```
Tìm dòng có nhãn (**LABEL**) là `DATa` và nhìn sang cột **MOUNTPOINTS**. Thông thường trên Fedora, đường dẫn sẽ dạng:
`/run/media/ten-user/DATa` hoặc `/mnt/DATa`

**Bước 2: Chạy lệnh TRIM cho thư mục đó**
Chạy lệnh `fstrim` trỏ trực tiếp tới đường dẫn vừa tìm được:
Bash
```
sudo fstrim -v /run/media/ten-user/DATa
```
_(Hãy thay `/run/media/ten-user/DATa` bằng đường dẫn thực tế trên máy bạn)._
> **Lưu ý:**
> 
> - Ổ **DATa** phải đang được mở (Mount) thì hệ thống mới TRIM được. Nếu chưa thấy đường dẫn mount, bạn chỉ cần mở ứng dụng **Files** (Tệp) và nhấn vào ổ **DATa** để kích hoạt nó trước.
>     
> - Nếu đây là ổ chuẩn NTFS/exFAT (dùng chung với Windows), lệnh trên vẫn hoạt động bình thường trên các bản Kernel Fedora mới.
>
![[Pasted image 20260728233819.png]]
