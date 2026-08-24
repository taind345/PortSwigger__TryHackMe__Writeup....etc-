Lỗi nằm ở dòng `open uinput failed`.

Phần mềm đã nhận diện được bảng vẽ và tín hiệu nút bấm (`PenKey 1 down/up`), nhưng không có quyền truy cập vào thiết bị `/dev/uinput` của hệ thống để tạo con trỏ chuột ảo.

Bạn xử lý theo các bước sau:

1. Thử chạy bằng quyền sudo để kiểm tra con trỏ:

```bash
sudo /usr/lib/pentablet/PenTablet.sh

```

2. Cấp quyền vĩnh viễn cho uinput để chạy được bằng tài khoản thông thường:

```bash
echo 'KERNEL=="uinput", MODE="0666"' | sudo tee /etc/udev/rules.d/99-uinput.rules
sudo modprobe uinput
sudo udevadm control --reload-rules && sudo udevadm trigger

```

Sau khi chạy lệnh cấp quyền trên, bạn tắt app đang chạy và mở lại bằng quyền user bình thường:

```bash
/usr/lib/pentablet/PenTablet.sh

```

Con trỏ sẽ di chuyển bình thường.


### auto khởi động driver
Driver của XP-Pen hoạt động dưới dạng tiến trình ứng dụng người dùng (user-space daemon) thay vì driver nhân Linux (kernel module). Vì vậy, hệ thống bắt buộc phải có tiến trình `PenTablet.sh` chạy ngầm thì tín hiệu bảng vẽ mới được xử lý.

Để không phải bật thủ công mỗi lần mở máy, bạn có thể thiết lập cho ứng dụng tự khởi động cùng hệ thống khi đăng nhập:

Tạo file tự khởi động trong thư mục cấu hình của tài khoản bằng lệnh sau:

```bash
mkdir -p ~/.config/autostart
cat << 'EOF' > ~/.config/autostart/xppentablet.desktop
[Desktop Entry]
Type=Application
Name=XP-Pen Tablet
Exec=/usr/lib/pentablet/PenTablet.sh /min
Hidden=false
NoDisplay=false
X-GNOME-Autostart-enabled=true
EOF

```

Tham số `/min` giúp phần mềm tự động thu nhỏ xuống khay hệ thống khi khởi động mà không làm phiền màn hình làm việc. Từ lần đăng nhập tiếp theo, bảng vẽ sẽ tự nhận diện ngay khi cắm vào máy.