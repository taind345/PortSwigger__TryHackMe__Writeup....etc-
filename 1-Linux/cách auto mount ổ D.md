### ==1.làm sao để auto mount ổ D của tôi, obsidian của tôi lấy dữ liệu từ ổ D nên mỗi lần khởi động lại máy là lại phải mount lại dữ liệu từ đầu??==



Để giải quyết vấn đề này, bạn cần ==thiết lập tính năng tự động gắn ổ đĩa (**Auto-mount**) vào hệ thống mỗi khi khởi động Fedora==. Vì đây là ổ D (thường có định dạng **NTFS** từ Windows), chúng ta cần cài đặt thêm gói hỗ trợ đọc/ghi và cấu hình file quản lý ổ đĩa `/etc/fstab`.

Bạn hãy mở **Terminal** và thực hiện lần lượt theo các bước chuẩn xác sau:
![[Pasted image 20260702234226.png]]

**Bước 1: Cài đặt driver hỗ trợ đọc/ghi ổ NTFS**

Để Fedora ghi và đọc dữ liệu Obsidian trên ổ D mượt mà, không bị lỗi phân quyền, hãy cài gói `ntfs-3g`:

bash

```
sudo dnf install ntfs-3g -y
```

Hãy thận trọng khi sử dụng mã.

**Bước 2: Tìm mã định danh (UUID) của ổ D**

Mỗi ổ đĩa trên Linux có một mã UUID cố định. Bạn hãy chạy lệnh sau để liệt kê các ổ đĩa:

bash

```
sudo blkid
```

Hãy thận trọng khi sử dụng mã.

_Bạn nhìn vào danh sách kết quả, tìm dòng có chữ `TYPE="ntfs"` hoặc `LABEL="..."` (tên ổ D của bạn). Hãy copy lại chuỗi **UUID** của ổ đó (ví dụ: `UUID="1234567890ABCDEF"`)._

**Bước 3: Tạo thư mục để gắn ổ đĩa vào (Mount Point)**

Bạn cần tạo một thư mục cố định để hệ thống đổ dữ liệu ổ D vào đó. Thông thường nên tạo trong thư mục `/mnt`:

bash

```
sudo mkdir -p /mnt/data
```

Hãy thận trọng khi sử dụng mã.

_(Bạn có thể thay chữ `data` bằng tên bất kỳ bạn thích, ví dụ: `/mnt/o_d`)._

**Bước 4: Cấu hình file hệ thống để tự động Mount khi khởi động**

Bây giờ, chúng ta sẽ ghi thông tin vào file cấu hình `/etc/fstab`. Chạy lệnh sau để mở file bằng trình chỉnh sửa Nano:

bash

```
sudo nano /etc/fstab
```

Hãy thận trọng khi sử dụng mã.

Dùng phím mũi tên di chuyển xuống **dòng cuối cùng** của file, sau đó copy và dán dòng cấu hình dưới đây vào (hãy thay mã UUID bằng mã thật của bạn ở Bước 2, và sửa lại đường dẫn thư mục nếu bạn đổi tên):

```
UUID=1234567890ABCDEF  /mnt/data  ntfs-3g  defaults,uid=1000,gid=1000,umask=022  0  0
```

- **uid=1000,gid=1000,umask=022**: Cực kỳ quan trọng đối với Obsidian, lệnh này cấp toàn quyền đọc/ghi ổ D cho tài khoản user hiện tại của bạn (`ti`), tránh lỗi Obsidian báo không có quyền tạo file.
![[Pasted image 20260702234541.png]]

**Cách lưu và thoát khỏi Nano:**

1. Nhấn tổ hợp phím **`Ctrl + O`** -> Nhấn **Enter** để lưu file.
2. Nhấn tổ hợp phím **`Ctrl + X`** để thoát ra ngoài Terminal.

**Bước 5: Thử nghiệm kiểm tra lỗi**

Trước khi khởi động lại máy, bạn **bắt buộc** phải chạy lệnh này để kiểm tra xem cấu hình có bị viết sai chính tả không (viết sai có thể khiến máy không lên nguồn):

bash

```
sudo mount -a
```

Hãy thận trọng khi sử dụng mã.

- Nếu lệnh chạy im lặng và không báo lỗi gì -> Thành công! Bạn mở ứng dụng Files (Quản lý tệp) truy cập vào thư mục `/mnt/data` sẽ thấy toàn bộ dữ liệu ổ D đã nằm ở đó.
- Nếu báo lỗi, hãy kiểm tra lại mã UUID hoặc dấu cách trong file `/etc/fstab`.

**Bước 6: Cập nhật đường dẫn trong Obsidian**

Bây giờ bạn chỉ cần mở Obsidian lên, chọn **Open folder as vault** và trỏ đường dẫn đến thư mục cố định mới là `/mnt/data/[Tên_Thư_Mục_Obsidian_Của_Bạn]`. Từ lần khởi động máy sau, ổ D sẽ tự lên và Obsidian sẽ tự động đọc được ngay lập tức.

Bạn đã lấy được chuỗi **UUID** của ổ D từ lệnh `blkid` chưa? Nếu gặp khó khăn khi đọc danh sách ổ đĩa, hãy dán kết quả lệnh đó lên đây để mình xem giúp nhé!


### ==-cách này có an toàn ko vậy??==


Cách cấu hình file `/etc/fstab` này **hoàn toàn an toàn và là phương pháp chuẩn mực nhất trên Linux** để tự động gắn ổ đĩa khi khởi động. Đây không phải là một bản "hack" mà là tính năng cốt lõi của hệ điều hành.

Tuy nhiên, để đảm bảo an toàn tuyệt đối 100% cho dữ liệu và giúp máy không bao giờ gặp sự cố khi khởi động, bạn nên lưu ý hai điểm kỹ thuật cực kỳ quan trọng sau:

1. Luôn sử dụng mã `UUID` thay vì tên ổ đĩa (`/dev/sdb...`)

Tên ổ đĩa (như `/dev/sdb1`, `/dev/nvme0n1p2`) có thể bị thay đổi ngẫu nhiên sau mỗi lần bạn cắm thêm USB hoặc ổ cứng di động. Nếu dùng tên ổ đĩa, hệ thống có thể gắn nhầm ổ.

- **Tại sao an toàn:** Mã `UUID` là định danh duy nhất của riêng phân vùng ổ D đó. Hệ thống sẽ chỉ tìm đúng phân vùng này để gắn vào, không bao giờ bị nhầm lẫn.

2. Mẹo an toàn: Thêm tham số `nofail` (Khuyên dùng)

Có một rủi ro nhỏ: Nếu một ngày nào đó bạn vô tình rút ổ cứng này ra (nếu là ổ di động), hoặc ổ D bị lỗi phần cứng, Fedora tìm không thấy ổ đĩa này lúc khởi động. Mặc định, hệ thống có thể sẽ dừng lại ở màn hình đen (Emergency Mode) để cảnh báo.

Để triệt tiêu hoàn toàn rủi ro này, bạn hãy thêm tham số `nofail` vào dòng cấu hình trong file `/etc/fstab`. Dòng lệnh an toàn tuyệt đối sẽ như sau:

text

```
UUID=1234567890ABCDEF  /mnt/data  ntfs-3g  defaults,uid=1000,gid=1000,umask=022,nofail  0  0
```

Hãy thận trọng khi sử dụng mã.

- **Tham số `nofail` có tác dụng gì?** Nó báo với Fedora rằng: _"Nếu khởi động mà thấy ổ D thì tự động gắn vào. Nếu không thấy (hoặc ổ bị rút ra), cứ bỏ qua và tiếp tục vào màn hình chính bình thường"_. Máy của bạn sẽ không bao giờ bị kẹt ở màn hình đen.

3. Quy tắc vàng: Chạy lệnh `sudo mount -a` trước khi tắt máy

Như đã nhắc ở bước trước, sau khi sửa file `/etc/fstab`, bạn **phải luôn** chạy lệnh `sudo mount -a`. Lệnh này giả lập quá trình khởi động để kiểm tra xem bạn có gõ sai chính tả hay sai cú pháp nào không. Nếu lệnh chạy im lặng không báo lỗi, bạn có thể yên tâm khởi động lại máy mà không gặp bất kỳ rủi ro nào.

Bạn đã mở được file `/etc/fstab` chưa, hoặc bạn có muốn mình kiểm tra giúp dòng lệnh cấu hình trước khi bạn lưu không?


### ==-làm như vậy có hại ổ cứng ko, và phải tốn thêm dung lượng ổ đĩa cho /mnt à??==


Bạn hoàn toàn có thể yên tâm về hai vấn đề này. Câu trả lời ngắn gọn là: ==**Không hề hại ổ cứng** và **Không tốn thêm bất kỳ dung lượng nào** của ổ đĩa cài Fedora== (`/`).

Dưới đây là giải thích chi tiết dưới góc độ kỹ thuật để bạn hiểu rõ:

1. Việc tự động mount có làm hại ổ cứng không?

- **Không hại ổ cứng:** Dù bạn dùng ổ cứng HDD truyền thống hay ổ SSD tốc độ cao, việc tự động mount **không làm tăng hao mòn phần cứng**.
- **Bản chất kỹ thuật:** Khi bạn mở máy, Fedora chỉ đọc một lượng dữ liệu cực nhỏ (vài KB) để nhận diện cấu trúc tệp tin của ổ D và tạo một đường dẫn kết nối. Nó không hề quét toàn bộ ổ đĩa hay thực hiện các thao tác ghi dữ liệu liên tục gây hại cho ổ.
- **Thậm chí còn tốt hơn:** Việc gắn cố định qua `/etc/fstab` giúp hệ thống quản lý năng lượng (ngủ đông/tắt ổ đĩa khi không dùng) hoạt động chuẩn xác hơn so với việc bạn click chuột thủ công nhiều lần trong ứng dụng Files.

2. Có tốn thêm dung lượng cho thư mục `/mnt` không?

- **Không tốn dung lượng:** Thư mục `/mnt/data` (hoặc bất kỳ thư mục nào bạn tạo ra để gắn ổ) thực chất chỉ là một **"Cánh cổng ảo" (Mount Point)**.
- **Bản chất kỹ thuật:** Thư mục này chỉ tốn vài byte trên ổ cài Fedora để lưu cái tên `/mnt/data`. Khi bạn chạy lệnh mount thành công, hệ thống Linux sẽ "bẻ lái" toàn bộ lưu lượng truy cập vào thư mục này thẳng sang phần cứng của ổ D.
- **Ví dụ trực quan:** Khi Obsidian lưu một file ghi chú dung lượng 10MB vào `/mnt/data/GhiChu.md`, toàn bộ 10MB này sẽ được ghi trực tiếp vào **ổ D (NTFS)** của bạn. Ổ đĩa hệ thống chứa Fedora không bị mất đi một byte dung lượng nào cho file đó cả.

---

Tóm lại, đây là cách làm tối ưu, an toàn và tiết kiệm tài nguyên nhất trên hệ điều hành Linux.

Bạn đã chạy lệnh `sudo blkid` để lấy mã **UUID** của ổ D chưa? Hãy gửi kết quả lên đây, mình sẽ viết sẵn dòng lệnh hoàn chỉnh (có kèm tham số an toàn `nofail`) để bạn chỉ việc copy-paste vào file cấu hình nhé!


### -lúc cần truy cập thì vào /mnt/DATA_D à



Chính xác rồi bạn! Từ giờ trở đi, bất cứ khi nào bạn muốn mở, xem, sửa hoặc xóa dữ liệu trên ổ D, bạn chỉ cần truy cập vào đường dẫn **`/mnt/DATA_D`**.

Để sử dụng một cách tiện lợi và nhanh chóng nhất trong các ứng dụng, bạn hãy áp dụng 2 mẹo nhỏ sau:

1. Cấu hình trên ứng dụng Obsidian

Khi bạn mở Obsidian lên:

1. Chọn tính năng **Open folder as vault** (Mở thư mục dưới dạng kho lưu trữ).
2. Duyệt theo đường dẫn: `Computer` (Máy tính) → `mnt` → `DATA_D` → chọn đến thư mục chứa dữ liệu ghi chú của bạn.
3. Từ các lần khởi động máy tiếp theo, Obsidian sẽ tự động ghi nhớ và truy cập thẳng vào đây mà bạn không cần phải chọn lại.

4. Mẹo tạo "Lối tắt" (Shortcut) trong ứng dụng Files để vào nhanh

Để không phải mỗi lần vào ổ D lại phải bấm qua nhiều thư mục (`/` → `mnt` → `DATA_D`), bạn hãy tạo một lối tắt ở thanh bên trái của ứng dụng quản lý tệp tin:

1. Mở ứng dụng **Files** (Quản lý tệp) mặc định của Fedora lên.
2. Bấm vào mục **Other Locations** (Vị trí khác) ở cột bên trái → Chọn **Computer** (Máy tính) → Vào thư mục **mnt** → Vào thư mục **DATA_D**.
3. Tại đây, bạn nhấn tổ hợp phím **`Ctrl + D`** (hoặc kéo thư mục `DATA_D` thả vào danh sách bên trái).
4. Hệ thống sẽ tạo một biểu tượng lối tắt tên là **DATA_D** ngay ở thanh bên cạnh. Lần sau muốn vào ổ D, bạn chỉ cần click đúng 1 phát vào biểu tượng đó là xong.

Mọi thứ hiện tại đã được thiết lập rất đồng bộ và tự động rồi. Bạn có gặp khó khăn gì trong việc trỏ dữ liệu của **Obsidian** vào thư mục mới này không?