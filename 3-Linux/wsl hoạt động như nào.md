![[Pasted image 20261014203115.png]]

Không hẳn là chạy “bên trong” Windows theo kiểu một phần mềm thông thường, và nó cũng không hoàn toàn độc lập với Windows khi truy cập phần cứng.

Cơ chế thực tế hoạt động như sau:

### 1. Bản chất: Cả Windows và Linux đều chạy trên Hyper-V

Hyper-V là một **Type-1 Hypervisor** (bộ giám sát máy ảo chạy sát phần cứng):

* ==**Hyper-V** nằm ở tầng dưới cùng, quản lý trực tiếp CPU và RAM.==
* **Windows** chạy trong phân vùng ưu tiên cao nhất (*Root Partition*).
* **WSL 2** chạy trong một phân vùng máy ảo siêu nhẹ riêng biệt (*Child Partition*).
![[Pasted image 20261014203648.png]]

### 2. Giao tiếp với phần cứng chia làm 2 phần:

* **CPU và RAM (Trực tiếp qua Hyper-V):**
Linux Kernel của WSL 2 ==thực thi các chỉ thị mã máy trực tiếp trên CPU vật lý== (thông qua tập lệnh ảo hóa phần cứng như Intel VT-x hoặc AMD-V). Windows **không can thiệp** hay làm nhiệm vụ phiên dịch ở bước này.
* **Ổ đĩa, Mạng và GPU (Vẫn phải qua Windows):**
WSL 2 không chiếm quyền trực tiếp toàn bộ phần cứng (không phải kiểu PCIe passthrough cắm thẳng thiết bị):
* **Ổ đĩa:** Toàn bộ hệ điều hành Linux thực chất chỉ là một tệp ảo `.vhdx` nằm trên ổ đĩa do Windows quản lý. ==Khi Linux đọc/ghi, nó gửi tín hiệu qua kênh giao tiếp ảo (**VMBus**) sang Windows để xử lý I/O.==
* **GPU:** WSL 2 chia sẻ chung card đồ họa với Windows bằng cơ chế ảo hóa bán phần (*GPU-PV*). Driver `dxgkrnl` trong Linux Kernel sẽ bắt tay với driver GPU trên Windows để render hoặc tính toán AI/CUDA.
* **Mạng:** Sử dụng một card mạng ảo kết nối qua một Virtual Switch do Windows định tuyến.

> [!NOTE]
> -cpu và ram--> linux kernel dùng trực tiếp
> -gpu, đọc ghi file--> phải đi qua thằng window--> dễ hiểu thôi do

Tóm lại: WSL 2 chạy song song với Windows dưới sự điều phối của Hyper-V; nó chạy **trực tiếp** trên CPU/RAM thực tế, nhưng việc truy xuất **tài nguyên lưu trữ và thiết bị ngoại vi** vẫn phải mượn tay Windows điều phối.


![[Pasted image 20261014204442.png]]
> [!NOTE]
> ### 1. Khi chạy tool, bash, deploy trên WSL: Đi thẳng hay vòng qua Windows?
> 
> Câu trả lời phụ thuộc vào việc **bạn đang chạy tác vụ gì và đặt dữ liệu ở đâu**:
> 
> - **Xử lý tính toán (CPU & RAM): ĐI THẲNG**
>     
>     - Khi bạn chạy bash script, biên dịch code (`gcc`, `cargo`, `npm build`), train mô hình AI hay chạy container Docker... các chỉ thị mã máy này được **CPU và RAM thực thi trực tiếp** thông qua tầng ảo hóa Hyper-V.
>         
>     - Windows **không hề đứng giữa dịch lệnh hay can thiệp**, nên tốc độ tính toán CPU/RAM đạt gần như 100% tốc độ thực tế của máy.
>         
> - **Thao tác đọc/ghi file (Disk I/O):**
>     
>     - **Nếu đặt dự án trong môi trường Linux (`/home/username/...`):** Linux Kernel tự xử lý trực tiếp trên hệ thống tệp `ext4` (nằm trong ổ đĩa ảo `.vhdx`). Tốc độ cực nhanh và ổn định, **không bị phụ thuộc vào Windows**.
>         
>     - **Nếu để dự án ở ổ đĩa Windows (`/mnt/c/...`):** Lúc này mọi thao tác đọc/ghi **bắt buộc phải vòng qua Windows** (thông qua giao thức cầu nối). Tốc độ sẽ bị suy giảm rõ rệt.
>         
> 
> ### 2. Tác dụng của WSL có lớn không? Có bằng cài Linux trực tiếp (Bare-metal) không?
> 
> #### Tác dụng của WSL là **rất lớn**:
> 
> 1. **Giải quyết bài toán "Hai thế giới":** Bạn vừa dùng được toàn bộ phần mềm quen thuộc của Windows (Office, Photoshop, giải trí, đồ họa...), vừa có trọn vẹn môi trường dòng lệnh Linux chuẩn gốc để làm việc, viết code và deploy.
>     
> 2. **Thay thế máy ảo nặng nề:** So với VMware hay VirtualBox ngốn hàng chục GB RAM cố định và mất cả phút khởi động, WSL 2 khởi động trong **chưa đầy 2 giây** và có khả năng **tự động trả lại RAM** cho Windows khi nhàn rỗi.
>     
> 3. **Chạy Docker và GPU Native:** WSL 2 hỗ trợ Docker mượt mà và tận dụng trực tiếp card đồ họa rời (NVIDIA CUDA) cho AI/Machine Learning mà không cần cấu hình phức tạp.
>     
> 
> #### So sánh hiệu năng với cài Linux trực tiếp (Dual-Boot / Bare-metal):
> 
> |**Tiêu chí**|**Cài Linux trực tiếp**|**WSL 2 (Đặt code trong /home)**|**WSL 2 (Đặt code ở /mnt/c)**|
> |---|---|---|---|
> |**Tính toán CPU & RAM**|100%|**95% – 98%** (Gần như tương đương)|**95% – 98%**|
> |**Tốc độ đọc/ghi file**|100%|**90% – 95%**|**30% – 50%** (Rất chậm)|
> |**Tính toán GPU (CUDA/AI)**|100%|**~99%**|**~99%**|
> |**Độ tiện dụng**|Phải reboot máy để đổi OS|Chạy song song ngay trên Windows|Chạy song song ngay trên Windows|
> 
> > **Tóm lại:** Nếu bạn clone repo và đặt mã nguồn bên trong thư mục Linux của WSL (ví dụ: `~/project`), hiệu năng thực tế **đạt tới 95–98% so với cài Linux trực tiếp**, hoàn toàn đáp ứng xuất sắc mọi nhu cầu lập trình, thử nghiệm tool và deploy.
> > 
> > Bạn chỉ cần cài Linux trực tiếp khi cần can thiệp sâu vào phần cứng đặc thù (nạp vi điều khiển qua cổng USB chuyên dụng, viết driver kernel, can thiệp sóng Wi-Fi thô...).
> 


