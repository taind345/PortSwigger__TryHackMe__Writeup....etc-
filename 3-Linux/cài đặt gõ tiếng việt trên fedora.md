Dưới đây là bản ghi chú (Note) hoàn chỉnh, ngắn gọn và tối ưu nhất. Bản note này đã gộp cả các bước cài đặt cốt lõi lẫn phần cấu hình tính năng tự động nhớ ngôn ngữ theo từng ứng dụng mà bạn vừa thiết lập thành công.

---

## 📝 CẨM NANG CÀI GÕ TIẾNG VIỆT TRÊN FEDORA (GNOME WAYLAND)

Tài liệu cấu hình hệ thống bộ gõ Fcitx5 + Unikey, hoạt động ổn định 100% trên môi trường Wayland của Fedora, tự động sửa lỗi ẩn bộ gõ trong Settings và hỗ trợ ghi nhớ ngôn ngữ thông minh theo từng ứng dụng.

## Bước 1: Cài đặt hệ thống Fcitx5 và các gói hỗ trợ

Mở Terminal và chạy lệnh sau để cài đặt Fcitx5, bộ gõ Unikey và tiện ích tích hợp giao diện GNOME (`kimpanel`):

```bash
sudo dnf install fcitx5 fcitx5-autostart fcitx5-unikey gnome-shell-extension-kimpanel -y
```
![[Pasted image 20260702235734.png]]
## Bước 2: Thiết lập biến môi trường bắt buộc (Sửa lỗi ứng dụng Wayland)

Chạy lệnh sau để ép tất cả các ứng dụng (Chrome, Firefox, Discord, VS Code...) nhận diện bộ gõ Fcitx5:

```bash
cat <<EOF >> ~/.bash_profile
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
EOF
```

## Bước 3: Khởi động lại hệ thống

Chạy lệnh sau để Fedora áp dụng các thiết lập môi trường mới:

```bash
reboot
```

## Bước 4: Kích hoạt kiểu gõ tiếng Việt Unikey

1. Nhấn phím Windows (Super), tìm và mở ứng dụng: Fcitx5 Configuration (hoặc gõ lệnh `fcitx5-configtool` trong Terminal).
2. Ở phần dưới cửa sổ, bỏ tích chọn ô `Only Show Current Language`.
3. Tại cột bên phải (Available Input Method), tìm chữ Unikey → Nhấp chọn nó → Bấm nút Mũi tên sang trái (`<-`) để chuyển Unikey sang cột bên trái.
4. Nhấn Apply ở góc dưới màn hình.
![[Pasted image 20260702235757.png]]

## Bước 5: Cấu hình tính năng "Ghi nhớ ngôn ngữ theo từng ứng dụng"

Để khi đa nhiệm, mỗi phần mềm tự nhớ ngôn ngữ riêng (ví dụ: Chrome giữ tiếng Việt, Terminal tự chuyển tiếng Anh):

1. Cũng tại cửa sổ Fcitx5 Configuration, chuyển sang tab Global Options ở thanh menu phía trên.
2. Tìm đến mục Share Input State (Chia sẻ trạng thái nhập liệu).
3. Bấm vào menu thả xuống và chọn giá trị là: Program (hoặc Chương trình).
4. Nhấn Apply để lưu cấu hình.
![[Pasted image 20260702235934.png]]


---

## ⌨️ Phím tắt mặc định khi sử dụng:

- Bật / Tắt tiếng Việt: Nhấn tổ hợp phím `Ctrl + Space`.
- Trạng thái bộ gõ sẽ hiển thị trực tiếp bằng biểu tượng trên thanh tác vụ (Taskbar) phía trên cùng màn hình.

---

fix lỗi gạch chân 
![[Pasted image 20260703150817.png]]
nano ~/.config/fcitx5/conf/bamboo.conf
=>sửa dòng      InlinePreedit=False
=>fcitx5 -r
