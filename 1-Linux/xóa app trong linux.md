Trên Fedora, lệnh chính thức để xóa ứng dụng là **sudo dnf remove ten_app**. Bạn có thể thực hiện nhanh chóng qua Terminal (Dòng lệnh) hoặc Giao diện đồ họa (Software Center).
## 1. Sử dụng Terminal (Dòng lệnh)

Mở Terminal (`Ctrl` + `Alt` + `T`) và sử dụng các lệnh sau tùy theo nhu cầu:

- Xóa ứng dụng cơ bản: Xóa ứng dụng và các gói phụ thuộc không cần thiết.  
    **sudo dnf remove ten_app**
- Dọn dẹp triệt để dữ liệu thừa: Xóa toàn bộ các tệp cấu hình cũ còn sót lại của ứng dụng.  
    **sudo dnf autoremove**
- Tìm chính xác tên ứng dụng: Nếu bạn không nhớ rõ tên gói cài đặt để xóa, hãy dùng lệnh sau để tìm kiếm.  
    **dnf list installed | grep ten_app**
## 2. Gỡ các ứng dụng dạng Flatpak (Phổ biến trên Fedora)

Fedora tích hợp sâu kho ứng dụng Flatpak. Nếu ứng dụng của bạn được cài từ nguồn này, hãy dùng lệnh:

- Xóa ứng dụng Flatpak:  
    `flatpak uninstall ten_app`
- Dọn dẹp dữ liệu Flatpak thừa:  
    `flatpak uninstall --unused`

## 3. Sử dụng Giao diện Đồ họa (GUI)

Nếu bạn đang dùng phiên bản Fedora Workstation (Gnome):

1. Mở ứng dụng Software (Chợ ứng dụng có biểu tượng túi mua sắm).
2. Nhấp vào tab Installed (Đã cài đặt) ở thanh menu phía trên.
3. Tìm ứng dụng cần xóa trong danh sách.
4. Nhấn nút Uninstall (Gỡ cài đặt) bên cạnh ứng dụng đó.
