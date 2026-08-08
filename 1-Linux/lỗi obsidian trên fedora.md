### Loi excalidraw
Lỗi không thể dán (paste) hình ảnh từ bộ nhớ đệm (clipboard) vào Excalidraw trên Obsidian khi dùng Fedora là một vấn đề rất phổ biến. Có hai nguyên nhân chính gây ra tình trạng này trên hệ thống của bạn: [1, 2]

1. ==Xung đột máy chủ hiển thị Wayland (Phổ biến nhất):== Obsidian được xây dựng trên nền tảng Electron. Khi chạy ở chế độ XWayland mặc định trên Fedora, nó bị hạn chế quyền truy cập dữ liệu hình ảnh trực tiếp từ clipboard của hệ thống Wayland bảo mật cao. [3]
2. ==Hạn chế của menu chuột phải trong Excalidraw:== Plugin Excalidraw đôi khi bị lỗi không bắt được dữ liệu hình ảnh khi bạn click chuột phải rồi chọn "Paste". [1, 2]

Bạn hãy áp dụng các giải pháp khắc phục triệt để dưới đây:

#### Giải pháp 1: Ép Obsidian chạy thuần Wayland (Sửa lỗi triệt để)

Để Obsidian có toàn quyền đọc clipboard hình ảnh của Fedora GNOME, bạn cần thêm tham số kích hoạt Wayland cho ứng dụng này.

- Nếu bạn cài Obsidian từ kho phần mềm Flatpak (mặc định):  
    Mở Terminal và chạy lệnh sau để cấp quyền truy cập thiết bị hiển thị Wayland:
    
    ```bash
    flatpak override --user --socket=wayland md.obsidian.Obsidian
    ```
    
- Nếu bạn cài Obsidian bằng định dạng AppImage:  
    Mỗi lần khởi chạy Obsidian từ Terminal, hãy thêm các cờ (flags) sau vào sau đường dẫn:
    
    ```bash
    ./Obsidian-x.x.x.AppImage --enable-features=UseOzonePlatform --ozone-platform=wayland
    ```
    

#### Giải pháp 2: Sử dụng phím tắt thay vì chuột phải

Menu chuột phải của Excalidraw trong Obsidian thường chỉ nhận văn bản (text), không nhận tệp hình ảnh thô từ bộ nhớ tạm. [1, 2]

- Cách làm: Sau khi chụp ảnh màn hình hoặc bấm sao chép ảnh, bạn hãy click chuột trái vào một vùng trống trên bảng vẽ Excalidraw, sau đó nhấn tổ hợp phím `Ctrl + V` thay vì dùng menu chuột phải. [1, 2, 4]

#### Giải pháp 3: Khắc phục lỗi lưu file của Obsidian trên Linux

Do bạn vừa đổi đường dẫn kho dữ liệu sang ổ D (`/mnt/DATA_D`), đôi khi Obsidian gặp lỗi phân quyền khi cố tạo file ảnh tạm.

- Thay vì copy-paste trực tiếp, bạn hãy dùng tính năng Insert Image (Biểu tượng hình bức tranh trên thanh công cụ của Excalidraw).
- Chọn hình ảnh trực tiếp từ máy tính để Excalidraw tự nạp vào. Cách này giúp tránh hoàn toàn lỗi clipboard và đảm bảo file ảnh được lưu trữ an toàn trong Vault của bạn. [5, 6, 7]

Bạn thử đổi sang nhấn `Ctrl + V` hoặc chạy lệnh kích hoạt Wayland xem Excalidraw đã nhận ảnh chụp màn hình chưa nhé!



### loi trust author moi lan mo file
-cái này là do xóa đi tải lại, éo hiểu sao nó lại lỗi 


Mở Terminal và dán lệnh này vào:

Bash

```
flatpak override --user --filesystem=/mnt/DATA_D md.obsidian.Obsidian
```

_(Tôi thiết lập cấp quyền đọc/ghi cho toàn bộ thư mục gốc `/mnt/DATA_D` thay vì đường dẫn dài. Việc này giúp tránh lỗi cú pháp do có khoảng trắng trong tên thư mục `0_Obsidian notebook`, đồng thời phòng ngừa lỗi nếu sau này bạn tạo thêm Vault mới trên ổ cứng này)._

**Bước cuối:** Đóng hoàn toàn Obsidian rồi mở lại, bấm **Trust** một lần cuối. Cấu hình sẽ được lưu thành công vào ổ đĩa và ứng dụng sẽ không bao giờ hỏi lại nữa.
### lỗi crash khi vào mục font của obsidian
Sự cố "trước đây bình thường, nay đột nhiên lỗi" trên Linux hầu hết bắt nguồn từ các bản cập nhật nền ngầm trong khoảng thời gian đó. Có 3 nguyên nhân chính:
1. **Fedora cập nhật `xdg-desktop-portal`:** Hộp thoại chọn font không phải của Obsidian mà do Fedora cung cấp qua cơ chế Portal của Flatpak. Bản cập nhật hệ thống gần đây có thể làm xung đột cổng giao tiếp này với môi trường Wayland, gây treo (lag) và crash.
2. **Obsidian/Electron cập nhật:** Bản thân gói Flatpak của Obsidian trên Flathub vừa được đẩy lên phiên bản mới, lõi Electron mới có thể bị lỗi tương thích phần cứng đồ họa (GPU) trên máy bạn.
3. **Tràn/Lỗi Font Cache:** Trong 2 tuần qua nếu bạn cài thêm font mới vào hệ thống, hoặc cache font của Flatpak bị lỗi, việc bấm gọi hộp thoại sẽ khiến app cố load toàn bộ danh sách font bị hỏng, dẫn đến tràn RAM và văng ứng dụng.
**Cách khắc phục dứt điểm (Không cần sửa tay file json):**
**Cách 1: Xóa và tạo lại bộ đệm font của Flatpak**
Mở Terminal chạy lệnh sau để dọn dẹp cache font:
```
flatpak run --command=fc-cache md.obsidian.Obsidian -f -v
```

**Cách 2: Ép dùng X11 qua Flatseal (Hiệu quả nhất với lỗi Crash UI)**
1. Mở ứng dụng **Flatseal** -> chọn **Obsidian**.
2. Cuộn đến phần **Socket**.
3. **Tắt** công tắc `Wayland windowing system`.
4. Đảm bảo **bật** công tắc `X11 windowing system` và `Fallback to X11 windowing system`.
    Khởi động lại Obsidian. Lệnh này ép app chạy qua giao thức cũ ổn định hơn, triệt tiêu hoàn toàn lỗi crash khi gọi hộp thoại hệ thống.