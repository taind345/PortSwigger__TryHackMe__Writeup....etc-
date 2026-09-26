
## 1. Dùng Store của Fedora (Đồ họa - Dễ nhất)

- Tên ứng dụng: GNOME Software (hoặc Discover trên bản KDE).
- Cách dùng: Mở ứng dụng lên, gõ tìm kiếm phần mềm và nhấn Install (Cài đặt) giống như trên điện thoại.
- Nguồn ứng dụng: Lấy từ kho mặc định của Fedora và kho Flathub (nếu đã bật).

## 2. Dùng lệnh DNF (Terminal - Truyền thống của Fedora)

- Bản chất: Cài đặt các gói phần mềm chính thức dạng `.rpm` do Fedora kiểm duyệt.
- Câu lệnh cơ bản:  
    `sudo dnf install tên_ứng_dụng`
- Đặc điểm: Tốc độ tải nhanh, ứng dụng nhẹ và tối ưu riêng cho Fedora.

## 3. Dùng lệnh Flatpak / Flathub (Terminal hoặc Store)

- Bản chất: Cài đặt các ứng dụng chạy trong môi trường cô lập (sandbox), không lo xung đột hệ thống.
- Câu lệnh cơ bản:  
    `flatpak install flathub tên_định_danh_ứng_dụng`
- Đặc điểm: Luôn có phiên bản mới nhất, chạy được trên mọi bản Linux, nhưng dung lượng tải về thường nặng hơn DNF.

## 4. Dùng AppImage (Chạy ngay không cần cài đặt)

- Bản chất: Ứng dụng được đóng gói thành một file duy nhất (giống file `.exe` di động của Windows).
- Cách dùng: Tải file đuôi `.AppImage` về → Click chuột phải chọn Properties → Bật quyền Allow executing file as program → Click đúp để chạy luôn.


## 5. Tải ứng dụng từ github
Với GitHub, bản chất của nó hoàn toàn khác với các cách trên. GitHub không phải là một kho phần mềm đóng gói sẵn cho Linux<u>, mà nó là một kho chứa mã nguồn </u>(source code) do các lập trình viên tải lên.
Để cài đặt ứng dụng từ GitHub trên Fedora, bạn có 3 trường hợp từ dễ đến khó sau:

**1-Trường hợp 1: Tác giả có sẵn file cài đặt (Dễ nhất)**
Nhiều lập trình viên đóng gói sẵn ứng dụng của họ và để trong mục Releases (Bản phát hành) trên GitHub.

* Cách làm: Vào trang GitHub của ứng dụng → Tìm mục Releases bên phải → Tải về file có đuôi .rpm (cài bằng DNF), .flatpak hoặc .AppImage (chạy ngay).

**2-Trường hợp 2: Cài bằng đoạn script tự động (Vừa phải)**
Tác giả viết sẵn một đoạn mã để tự động tải và cài đặt mọi thứ cho bạn.

* Cách làm: Bạn chỉ cần copy câu lệnh dạng curl ... | bash hoặc wget ... mà tác giả ghi ở phần hướng dẫn (README.md) rồi dán vào Terminal để chạy.

**3-Trường hợp 3: Tải mã nguồn về và tự biên dịch (Khó nhất)**
Nếu không có file đóng gói sẵn, bạn phải tải "nguyên liệu thô" (mã nguồn) về và tự "nấu" (biên dịch) thành ứng dụng trên máy mình.

* Cách làm (Quy trình chung):
1. Tải mã nguồn về bằng lệnh: git clone <link_github>
   2. Di chuyển vào thư mục vừa tải: cd <tên_thư_mục>
   3. Cài đặt các công cụ biên dịch (thường là lệnh make, cmake, hoặc gcc).
   4. Chạy lệnh biên dịch và cài đặt theo hướng dẫn của tác giả (thường là make rồi đến sudo make install).
