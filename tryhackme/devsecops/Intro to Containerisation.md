==>[[note-22-9#2-container]]
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

