
https://share.gemini.google/D2XHMseyq9S3
### 1- Sử dụng Virtual Machine Manager 
Công cụ này cung cấp toàn quyền kiểm soát phần cứng, cấu hình mạng ảo, snapshot - cực kỳ cần thiết cho các lab an toàn thông tin (như thiết lập web security lab).
**1.Cài đặt các gói cần thiết:**
Mở terminal và chạy lệnh cài đặt nền tảng ảo hóa KVM cùng công cụ quản lý:

```
sudo dnf install @virtualization
```
**2.Khởi động service libvirtd:**
Bật và cho phép service ảo hóa chạy tự động cùng hệ thống:

```
sudo systemctl enable --now libvirtd
```
**3.Cấp quyền cho user (Tùy chọn):**
Để không phải nhập mật khẩu root mỗi lần mở máy ảo, hãy thêm user của bạn vào nhóm `libvirt`:
```
sudo usermod -aG libvirt $USER
```
**4.Tạo máy ảo:**
Mở ứng dụng **Virtual Machine Manager** từ menu. Bấm vào biểu tượng **"Create a new virtual machine"**, chọn file `.iso` (Windows, Kali Linux, Ubuntu, v.v.) và làm theo các bước thiết lập RAM/CPU.![[Pasted image 20260804190910.png]]


### 2-cài đặt máy ảo và lưu trữ nó trên ổ D
Mặc định, KVM sẽ lưu ổ cứng máy ảo ở phân vùng gốc của Linux (`/var/lib/libvirt/images/`). Để chuyển sang lưu ở ổ D, bạn phải tạo một **Storage Pool** mới:

**1.Tạo thư mục trên ổ D:**
Mở trình quản lý file, vào ổ D của bạn và tạo một thư mục riêng để chứa máy ảo (Ví dụ: đặt tên là `VM_Linux`).

**2.Thiết lập Custom Storage:**
Trong quá trình tạo máy ảo bằng virt-manager, đi tới bước cấu hình ổ cứng (bước 4), hãy đánh dấu chọn mục **"Select or create custom storage"** và bấm nút **Manage**.

**3.Tạo Storage Pool mới:**
Ở cửa sổ hiện ra, bấm vào dấu **"+"** (Add Pool) ở góc dưới bên trái:
- **Name:** Đặt tên tùy ý (ví dụ: `O_D_Pool`).
    
- **Type:** Chọn `dir: Filesystem Directory`.
    
- **Target Path:** Nhấn Browse và trỏ tới thư mục `VM_Linux` bạn vừa tạo trên ổ D.
    
- Bấm **Finish**.
    ![[Pasted image 20260804192245.png]]
**4.Tạo phân vùng cho máy ảo:**
Sau khi tạo Pool, hãy chọn Pool đó. Bấm dấu **"+"** (Add Volume) ở góc trên để tạo một file đĩa ảo mới (định dạng `.qcow2`), cấp dung lượng RAM và tiến hành cài đặt.
![[Pasted image 20260804192638.png|641]]
### 3-FIX lỗi -Phân quyền cho máy ảo đọc ghi vào ổ D
[[cách auto mount ổ D]]
#### Lỗi chắc chắn bạn sẽ gặp: "Permission Denied"
Vì ổ D của bạn là định dạng **NTFS** (dành cho Windows) và Fedora sử dụng cơ chế bảo mật **SELinux** rất nghiêm ngặt, khi bạn bấm "Start" máy ảo, 99% hệ thống sẽ báo lỗi từ chối quyền truy cập (Permission Denied).
![[Pasted image 20260804192833.png|503]]
Nguyên nhân nằm ở cơ chế phân quyền (permissions) của tham số `umask=022` và đặc thù của NTFS:
- `uid=1000,gid=1000`: Bạn đang mount ổ D sao cho tài khoản user của bạn (có ID là 1000) là chủ sở hữu của mọi file trên ổ đĩa này.
- `umask=022`: Cấu hình này cấp quyền Đọc/Ghi (Read/Write) cho chủ sở hữu (user 1000), nhưng **chỉ cấp quyền Đọc (Read-only)** cho những user khác trên hệ thống.
- **Vấn đề:** *Tiến trình ảo hóa mặc định chạy dưới quyền của một user hệ thống tên là `qemu`. Khi `qemu` cố gắng ghi dữ liệu vào file ổ cứng ảo (`.qcow2`) nằm trên ổ D, nó sẽ bị hệ thống chặn lại vì chỉ có quyền Đọc.* KVM không thể hoạt động nếu không ghi được vào đĩa ảo. (Ngoài ra, tiến trình ảo hóa cũng không thể dùng lệnh `chown` để đổi quyền file vì NTFS không hỗ trợ tính năng này của Linux).

*==>Để giải quyết vấn đề này mà vẫn giữ được sự an toàn (không phải chạy dưới quyền `root` nguy hiểm như đã phân tích), bạn có **2 cách tối ưu nhất**:

####  Cho QEMU chạy dưới quyền user của chính bạn (Khuyên dùng)
Đây là cách tốt nhất để cân bằng giữa bảo mật và tính tiện dụng. Bạn ép tiến trình ảo hóa chạy dưới tài khoản cá nhân của bạn (UID 1000) thay vì user `qemu`. Vì ổ D đã được mount cho UID 1000, máy ảo sẽ có toàn quyền truy cập mà không cần quyền `root` của hệ thống.
1. Mở terminal và sửa file cấu hình:
    ```
    sudo nano /etc/libvirt/qemu.conf
    ```
2. Tìm đến các dòng `user = "qemu"` và `group = "qemu"`, bỏ dấu `#` và đổi thành tên user đăng nhập của bạn (thường là tên bạn dùng để đăng nhập vào Fedora). Ví dụ nếu tên user của bạn là `tai`:
    ```
    user = "tai"
    group = "tai"
    ```
![[Pasted image 20260804194244.png]]
3. Tìm dòng `#security_driver = "selinux"` (hoặc tìm chữ `security_driver`), xóa dấu `#` ở đầu và sửa lại thành:

```
security_driver = "none"
```
![[Pasted image 20260804202140.png|455]]4. Lưu lại và khởi động lại dịch vụ:
    ```
    sudo systemctl restart libvirtd
    ```

### 4-tăng dung lượng cho máy ảo
việc tăng dung lượng cho máy ảo KVM (file `.qcow2`) sau khi đã cài đặt xong gồm 2 giai đoạn: **Mở rộng file ổ cứng vật lý trên Fedora (Host)** và **Mở rộng phân vùng bên trong máy ảo Kali Linux (Guest)**.

Đây là hướng dẫn an toàn và chuẩn xác nhất để dữ liệu không bị lỗi.

#### Giai đoạn 1: Mở rộng file ổ cứng trên máy thật (Fedora)

**1.Tắt hoàn toàn máy ảo:**

Đảm bảo máy ảo Kali Linux đã được Shutdown (không phải Suspend hay Save state).

**2.Mở terminal và di chuyển đến thư mục chứa ổ cứng ảo:**

Vì bạn đã lưu ổ cứng ở phân vùng DATA (ổ D), hãy dùng terminal đi đến đó (thay đổi đường dẫn cho đúng với thực tế máy bạn, dựa theo ảnh trước đó có thể là `/mnt/data/VM_LINUX/`):

```
cd /mnt/data/VM_LINUX/
```

**3.Sử dụng lệnh qemu-img để tăng dung lượng:**Ví dụ: Tăng thêm 20GB.

Gõ lệnh sau (nhớ thay tên file `kali_linux.qcow2` bằng tên file thực tế của bạn):

```
qemu-img resize kali_linux.qcow2 +20G
```

_Hệ thống sẽ báo `Image resized` là thành công. Lúc này cái "vỏ" ổ cứng đã to ra thêm 20GB._
![[Pasted image 20260823101002.png]]
#### Giai đoạn 2: Cấp phát dung lượng mới bên trong máy ảo (Kali Linux)

Mặc dù "vỏ" ổ cứng đã to ra, nhưng hệ điều hành bên trong chưa tự động nhận diện phần không gian mới này. Bạn cần gộp nó vào phân vùng hiện tại.

**1.Khởi động máy ảo Kali Linux:**
Mở Virtual Machine Manager và bật máy ảo lên.

**2.Cài đặt công cụ GParted:**
Mở terminal bên trong Kali Linux và cài đặt phần mềm quản lý phân vùng bằng giao diện trực quan:

```
sudo apt update
sudo apt install gparted -y
```

**3.Mở rộng phân vùng bằng GParted:**

1. Chạy phần mềm: Gõ `sudo gparted` trong terminal (hoặc tìm trong menu ứng dụng).
    
2. Trên giao diện, bạn sẽ thấy ổ đĩa của mình (`/dev/vda` hoặc `/dev/sda`) và một vùng xám ghi là **"unallocated" (chưa cấp phát)** - đây chính là 20GB bạn vừa thêm.
    
3. Phân vùng chính của Kali thường nằm ngay sát bên trái vùng unallocated (vd: `/dev/vda1` hoặc `/dev/vda5`).
    
4. Click chuột phải vào phân vùng chính đó, chọn **Resize/Move**.
    
5. Kéo mũi tên sang phải đến kịch kim để lấp đầy khoảng trống "unallocated".
    
6. Bấm **Resize**.
    
7. Cuối cùng, bấm dấu check màu xanh (Apply All Operations) trên thanh công cụ để thực thi thay đổi.
    
