==>[[note-docker-pipeline#2-container]]
# 2-What is a containersation

![[Pasted image 20260922213523.png]]Trong thuật ngữ điện toán, **containerisation** (container hóa) là quá trình đóng gói một ứng dụng cùng các tài nguyên cần thiết (như các **libraries** và **packages**) vào trong một gói duy nhất gọi là **container**. Quá trình này giúp ứng dụng có tính di động rất cao (**portable**) và có thể vận hành trơn tru mà không gặp rắc rối.

Các ứng dụng hiện đại thường phức tạp và phụ thuộc vào các **frameworks** cùng **libraries** phải được cài đặt sẵn trên thiết bị trước khi ứng dụng có thể chạy. Những **dependencies** (thành phần phụ thuộc) này có thể:

* Rất khó cài đặt tùy thuộc vào môi trường chạy ứng dụng (một số hệ điều hành thậm chí còn không hỗ trợ chúng!).
* Gây khó khăn cho developer trong việc chẩn đoán và tái hiện lỗi (**replicate faults**), vì nguyên nhân có thể bắt nguồn từ môi trường của ứng dụng chứ không phải do bản thân code của ứng dụng!
* Thường xuyên xung đột (**conflict**) lẫn nhau. Ví dụ: việc phải cài nhiều phiên bản Python khác nhau để chạy các ứng dụng khác nhau là một cơn đau đầu cho người dùng; một ứng dụng có thể hoạt động tốt trên phiên bản Python này nhưng lại lỗi trên phiên bản khác.

Các nền tảng container hóa giải quyết triệt để vấn đề này bằng cách đóng gói các dependencies lại với nhau và "cô lập" (**isolating**) môi trường của ứng dụng *(lưu ý: đừng nhầm lẫn khái niệm này với "cô lập bảo mật - security isolation" trong ngữ cảnh này)*. Nếu thiết bị hỗ trợ **containerisation engine** (công cụ thực thi container), người dùng có thể chạy ứng dụng và nhận được cùng một hành vi/kết quả hoạt động giống hệt nhau ở mọi nơi.

Trong ảnh chụp màn hình ở trên, chúng ta có thể thấy ba ứng dụng cùng môi trường của chúng (chẳng hạn như dependencies) được đóng gói cùng nhau và không tương tác trực tiếp với máy tính vật lý — mà tương tác thông qua **containerisation engine** (trong trường hợp này là **Docker**).

Chúng ta sẽ bàn sâu hơn về cách các container cô lập lẫn nhau sau, nhưng hiện tại, điều quan trọng là phải hiểu tính năng cô lập này là đặc tính cốt lõi của container.

Tuy nhiên, cần lưu ý rằng các nền tảng container tận dụng tính năng **namespace** của **kernel** (nhân hệ điều hành) — tính năng cho phép các tiến trình (**processes**) truy cập vào tài nguyên của hệ điều hành mà không thể can thiệp hay tương tác với các processes khác.

Sự cô lập do **namespaces** mang lại bổ sung thêm một lợi thế lớn về bảo mật: nếu một ứng dụng bên trong container bị xâm nhập (**compromised**), thông thường (trừ khi dùng chung namespace), các container khác sẽ không bị ảnh hưởng.

Các giải pháp thay thế như máy ảo (lab machines) sẽ đòi hỏi phải cài đặt nguyên một hệ điều hành hoàn chỉnh chỉ để chạy ứng dụng (gây tiêu tốn một lượng lớn dung lượng ổ đĩa và các tài nguyên tính toán khác như CPU và RAM).

> [!NOTE]
> Container là một tiến trình (process) được cô lập, dùng chung kernel với hệ điều hành mẹ và được đóng gói sẵn toàn bộ code cùng thư viện cần thiết để chạy.

# 2-introducting docker
Tôi hứa sẽ giới thiệu thật ngắn gọn và súc tích. Docker là một nền tảng container hóa (containerisation platform) mã nguồn mở, toàn diện và tương đối dễ tiếp cận, ít rắc rối. Hệ sinh thái của Docker cho phép các ứng dụng (dưới dạng các **images** — chúng ta sẽ tìm hiểu kỹ hơn ở room sau) được triển khai, quản lý và chia sẻ một cách dễ dàng.

Hoạt động trên cả Linux, Windows và macOS, Docker là lựa chọn thông minh để vận hành các ứng dụng. Ứng dụng có thể được phát hành dưới dạng các "images" và chia sẻ cho người khác. Tất cả những gì bạn cần làm là kéo (**pull** / tải về) image đó về và khởi chạy nó bằng Docker.

Docker áp dụng công nghệ container hóa để cô lập các ứng dụng vào các container thông qua một thành phần gọi là **Docker Engine**. Về bản chất, Docker Engine là một API chạy trên hệ điều hành máy chủ (host OS), đóng vai trò cầu nối liên lạc giữa hệ điều hành và các container nhằm truy cập vào tài nguyên phần cứng của hệ thống (như CPU, RAM, card mạng và ổ đĩa).

Nhờ kiến trúc này, Docker Engine rất linh hoạt và cho phép bạn thực hiện những thao tác như:

* Kết nối các container lại với nhau (ví dụ: một container chạy ứng dụng web giao tiếp với một container khác chạy cơ sở dữ liệu).
* Xuất (**export**) và nhập (**import**) các ứng dụng (images).
* Sao chép, truyền tải file qua lại giữa hệ điều hành máy chủ và container.

Docker sử dụng cú pháp định dạng **YAML** để cho phép lập trình viên chỉ định cụ thể cách một container được build cũng như những gì sẽ được thực thi bên trong. Đây là nguyên nhân then chốt giúp Docker có tính di động (**portable**) cao và rất dễ debug: bạn chỉ cần chia sẻ file cấu hình này, ứng dụng sẽ được build và vận hành đồng nhất trên bất kỳ thiết bị nào có cài đặt Docker Engine.

Docker Engine cũng hỗ trợ điều phối (**orchestration**) các container, đồng nghĩa với việc nhiều container có thể được khởi tạo như một nhóm và giao tiếp trực tiếp với nhau (chẳng hạn: container chạy web server có thể kết nối với container chạy database). Chúng ta sẽ tìm hiểu sâu hơn về tính năng này trong các room tiếp theo.

> [!NOTE] ví dụ
>  **Sử dụng cú pháp YAML (`docker-compose.yml`) để Docker Engine khởi chạy và kết nối 2 container (một Web App và một Database) với nhau.**
> 
> 
> 
> ### Tình huống
> 
> Bạn cần triển khai một trang web WordPress kết nối với cơ sở dữ liệu MySQL. Thay vì phải cài đặt thủ công MySQL, PHP, Web Server lên máy tính, bạn chỉ cần tạo **một file duy nhất** tên là `docker-compose.yml`.
> 
> ### 1. Khai báo bằng cú pháp YAML (`docker-compose.yml`)
> 
> ```yaml
> version: '3.8'
> 
> services:
>   # Container 1: Chạy Database MySQL
>   db:
>     image: mysql:8.0                     # Kéo image MySQL từ Docker Hub về
>     restart: always
>     environment:
>       MYSQL_ROOT_PASSWORD: mysecretpassword
>       MYSQL_DATABASE: my_database
> 
>   # Container 2: Chạy Ứng dụng Web (WordPress)
>   web:
>     image: wordpress:latest              # Kéo image WordPress từ Docker Hub về
>     restart: always
>     ports:
>       - "8080:80"                        # Mở cổng 8080 trên máy thật để người dùng truy cập
>     environment:
>       WORDPRESS_DB_HOST: db              # Kết nối trực tiếp tới container database bằng tên "db"
>       WORDPRESS_DB_PASSWORD: mysecretpassword
>       WORDPRESS_DB_NAME: my_database
> 
> ```
> 
> 
> 
> ### 2. Cách Docker Engine xử lý bên dưới
> 
> Khi bạn gõ lệnh:
> 
> ```bash
> docker compose up -d
> 
> ```
> 
> Docker Engine sẽ tự động thực hiện toàn bộ các cơ chế đã nêu:
> 
> 1. **Pull Images (Tải ứng dụng):** Kiểm tra máy bạn đã có image `mysql:8.0` và `wordpress:latest` chưa; nếu chưa, nó tự động tải về từ kho lưu trữ.
> 2. **Khởi tạo và cô lập Container:** Tạo ra 2 tiến trình độc lập, được cô lập môi trường (mỗi container có filesystem và thư viện riêng).
> 3. **Kết nối mạng (Networking & Orchestration):** Docker Engine tự tạo một mạng ảo nội bộ (Internal Virtual Network) và gắn cả 2 container này vào.
> * Container `web` không cần biết IP phức tạp của máy chủ, nó chỉ cần gọi thẳng tên miền nội bộ `db` là Docker DNS sẽ tự điều hướng tới đúng container MySQL.
> 
> 
> 1. **Truy cập:** Bạn chỉ cần mở trình duyệt gõ `http://localhost:8080` là trang web đã sẵn sàng hoạt động.
> 
> 
> 
> ### Giá trị thực tế của ví dụ này
> 
> * **Tính di động (Portability):** Bạn gửi file `docker-compose.yml` này cho bất kỳ ai (dù họ dùng Windows, macOS hay Linux Ubuntu), họ chỉ cần gõ đúng một lệnh `docker compose up -d` là hệ thống sẽ khởi chạy giống hệt 100% như trên máy bạn mà không lo xung đột môi trường.

# 3-How does container work
Nếu những điều trước đó vẫn chưa đủ thuyết phục, thì dưới đây là một tóm lược rõ ràng hơn: Docker là một phương thức linh hoạt (agile), tiện lợi và toàn diện để triển khai ứng dụng. Hãy cùng tìm hiểu chi tiết qua các đề mục bên dưới.
### Docker miễn phí
Hệ sinh thái Docker là mã nguồn mở và miễn phí sử dụng. Dù có các gói dịch vụ thương mại dành cho doanh nghiệp, bạn hoàn toàn có thể tải về, sử dụng, tạo mới, khởi chạy và chia sẻ các image mà không tốn chi phí nào.

### Docker có tính tương thích cao
Nền tảng Docker tương thích với Linux, macOS và Windows. Nhờ vào cơ chế hoạt động của container hóa, chỉ cần thiết bị hỗ trợ Docker Engine, bạn có thể chạy bất kỳ container nào mà không phải bận tâm về ứng dụng bên trong hay các thành phần phụ thuộc (dependencies).

### Docker hiệu quả & Tối giản
Docker là giải pháp cô lập ứng dụng hiệu quả vượt trội so với các phương án thay thế như máy ảo (lab machines). Nguyên nhân là do Docker Engine chạy và tương tác trực tiếp với hệ điều hành máy chủ (host OS), đồng thời các container không cần phải chạy một hệ điều hành hoàn chỉnh riêng biệt. Chẳng hạn, các container có thể dùng chung một base OS image tối giản, nghĩa là bạn chỉ cần lưu trữ image đó một lần duy nhất.

Một image Ubuntu tối giản chỉ nặng khoảng 100MB, lưu một lần nhưng dùng lại được nhiều lần. So sánh với image Ubuntu Server dùng cho máy ảo (VM), dung lượng sau khi cài mới đã ngốn khoảng 1GB.

> **Kiểm tra kích thước của Docker image "ubuntu":**

```bash
ubuntu@thm:~$ docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       latest    27941809078c   4 weeks ago   77.8MB
ubuntu@thm:~$

```

### Dễ dàng bắt đầu tiếp cận

Tài liệu hướng dẫn dành cho lập trình viên của Docker được biên soạn rất chi tiết, đi kèm nhiều bài viết, ví dụ thực tế và các câu hỏi được giải đáp rộng rãi trên Internet. Hầu như bất kỳ tác vụ nào bạn muốn làm với Docker thì đều đã có người từng hỏi hoặc thực hiện trước đó.

Cú pháp để bắt đầu với Docker rất trực quan và dễ tiếp thu. Bạn có thể khởi chạy container đầu tiên chỉ trong chốc lát (đặc biệt là nhờ sự hỗ trợ của các docker image dựng sẵn cho đủ mọi loại ứng dụng đã được cộng đồng phát hành).

### Dễ dàng chia sẻ với người khác

Lợi ích lớn của Docker là tính di động (portability). Docker sử dụng các "image" để lưu trữ các chỉ dẫn quy định cách một container được build (tương tự như một cuốn sách hướng dẫn từng bước).

Các image này có thể được xuất ra, chia sẻ và tải lên các kho lưu trữ (repositories) công khai hoặc riêng tư như Docker Hub hay GitHub. Một image có thể chạy trên bất kỳ môi trường nào hỗ trợ Docker Engine, miễn là cú pháp cấu hình hợp lệ.

### Tinh gọn và gia tăng bảo mật

Các Docker image này rất tinh gọn. Bạn sẽ thường thấy trong một container thiếu vắng nhiều công cụ hay gói phần mềm tiện ích vốn quen thuộc trên hệ điều hành thông thường. Dù thoạt nhìn có vẻ là một bất lợi, nhưng thực chất điều này mang lại:

* Khả năng đóng gói container chính xác theo đúng những gì lập trình viên mong muốn, không dư thừa.
* Mức độ bảo mật cao hơn: việc biết chính xác những gì đang chạy bên trong container giúp giảm thiểu nguy cơ các package không cần thiết dính lỗ hổng bảo mật (vulnerabilities) gây rủi ro cho toàn hệ thống.

### Chi phí vận hành tiết kiệm hơn

Chạy container thường là lựa chọn tiết kiệm chi phí hơn nhiều so với việc chạy máy ảo. Điều này thể hiện rõ rệt nhất trong các môi trường Cloud (đám mây), nơi tài nguyên CPU, RAM và dung lượng ổ đĩa có giá khá đắt.

Bạn hoàn toàn có thể chạy mượt mà vài container trên một gói Cloud VPS chỉ $5, trong khi cấu hình đó không thể gánh nổi một máy ảo. Điều này xuất phát từ hai lý do:

* Chạy máy ảo đòi hỏi phần cứng hỗ trợ ảo hóa (hardware virtualization) — tính năng thường chỉ xuất hiện trên các gói dịch vụ cao cấp đắt đỏ của các nhà cung cấp đám mây.
* Máy ảo tiêu tốn rất nhiều RAM và dung lượng đĩa do phải vận hành một hệ điều hành riêng biệt đè lên máy vật lý.

# 4-
Namespaces essentially segregate system resources such as processes, files and memory away from other namespaces.  

Every process running on Linux will be assigned two things:  

- A namespace
- A process identifier (PID)

Namespaces are how containerisation is achieved! Processes can only "see" other processes that are in the same namespace - no conflicts in theory. Take Docker, for example, every new container will be running as a new namespace, although the container may be running multiple applications (and in turn, processes).

Let's prove the concept of containerisation by comparing the number of processes there are in a Docker container that is running a web server versus the host operating system at the time:

![an image depicting the large amount of processes running within a normal operating system](https://cdn-images.tryhackme.com/user-uploads/5de96d9ca744773ea7ef8c00/room-content/ca9a3fd100bc4f0f9709d62925678cbc.png)  

Put simply, the process with an ID of 0 is the process that is started when the system boots. Process numbers increment and must be started by another process, so naturally, the next process ID will be #1. This process is the systems `init` , for example, the latest versions of Ubuntu use `systemd`. Any other process that runs will be controlled by `systemd` (process #1).

We can use process #1's namespace on an operating system to escalate our privileges. Whilst containers are designed to use these namespaces to isolate from each other, they can instead coincide with the host computer's processes... This gives us a nice opportunity to escape!

![an image depicting the limited amount of processes running within a container](https://cdn-images.tryhackme.com/user-uploads/5de96d9ca744773ea7ef8c00/room-content/8dc65b64a94dcd264dfddf8feca7af8f.png)