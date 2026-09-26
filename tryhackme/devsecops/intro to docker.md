# 2 - Các lệnh Docker (Docker commands)

Lệnh `docker run` tạo và khởi chạy các container từ các image. Đây là nơi các chỉ thị từ Dockerfile (cũng như các tham số đầu vào do chúng ta truyền lúc runtime) được thực thi. Vì vậy, đây là một trong những cú pháp cơ bản đầu tiên bạn cần học.

Cú pháp của lệnh hoạt động như sau: `docker run [OPTIONS] IMAGE_NAME [COMMAND] [ARGUMENTS...]`  
Các tùy chọn nằm trong dấu ngoặc vuông `[...]` là không bắt buộc để khởi chạy một container.

Docker container có thể được chạy với nhiều tùy chọn khác nhau — tùy thuộc vào mục đích sử dụng container của chúng ta. Phần này sẽ giải thích một số tùy chọn phổ biến nhất mà bạn có thể cần dùng.

### Đầu tiên, Chạy một Container đơn giản

Hãy nhớ lại cú pháp cần thiết để chạy một Docker container: `docker run [OPTIONS] IMAGE_NAME [COMMAND] [ARGUMENTS...]`. Trong ví dụ này, tôi sẽ cấu hình container để chạy:

- Một image có tên là "helloworld"
- Chế độ "tương tác" (Interactive) bằng cách truyền cờ `-it` trong phần `[OPTIONS]`. Cờ này cho phép chúng ta tương tác trực tiếp với container.
- Tôi sẽ khởi tạo một shell bên trong container bằng cách truyền `/bin/bash` vào phần `[COMMAND]`. Tham số này là nơi bạn đặt các lệnh muốn thực thi bên trong container (chẳng hạn như một tệp script, ứng dụng hoặc một shell!).

Vì vậy, để thực hiện các yêu cầu trên, câu lệnh sẽ như sau: `docker run -it helloworld /bin/bash`

Một terminal hiển thị một container đang được khởi chạy ở chế độ "interactive" (tương tác):

```shell-session
cmnatic@thm-intro-to-docker:~$ docker run -it helloworld /bin/bash
root@30eff5ed7492:/#
```

Chúng ta có thể xác minh rằng mình đã khởi chạy thành công shell vì dấu nhắc lệnh (prompt) sẽ đổi sang một tài khoản người dùng và hostname khác. Hostname của một container chính là Container ID (có thể tìm thấy bằng lệnh `docker ps`). Ví dụ, trong terminal ở trên, username và hostname của chúng ta là `root@30eff5ed7492`.

### Chạy Container... (Tiếp tục)

Như đã đề cập trước đó, Docker container có thể được chạy với nhiều tùy chọn khác nhau. Mục đích của container và các chỉ thị được thiết lập trong Dockerfile (chúng ta sẽ tìm hiểu ở phần sau) sẽ quyết định những tùy chọn nào cần dùng khi chạy container. Để bắt đầu, tôi đã tổng hợp một số tùy chọn phổ biến nhất vào bảng bên dưới:

> [!NOTE]
> Các cờ (flags/options) phổ biến của lệnh `docker run`

| [OPTION] | Giải thích (Explanation)                                                                                                                                                                                                                                                         | Chỉ thị Dockerfile liên quan | Ví dụ (Example)                                                    |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- | ------------------------------------------------------------------ |
| `-d`     | Tham số này yêu cầu container chạy ở chế độ "detached" (chạy ngầm). Điều này có nghĩa là container sẽ chạy dưới nền (background).                                                                                                                                                | N/A                          | `docker run -d helloworld`                                         |
| `-it`    | Tham số này gồm hai phần: "i" nghĩa là chạy tương tác (interactive), và "t" chỉ định Docker cấp phát một pseudo-TTY (mở shell) bên trong container. Ta dùng tùy chọn này nếu muốn tương tác trực tiếp với container sau khi khởi chạy.                                           | N/A                          | `docker run -it helloworld`                                        |
| `-v`     | Viết tắt của "Volume", chỉ định Docker mount (gắn kết) một thư mục hoặc tệp từ hệ điều hành máy chủ (host OS) vào một vị trí bên trong container. Vị trí lưu trữ các tệp này được định nghĩa trong Dockerfile.                                                                   | VOLUME                       | `docker run -v /host/os/directory:/container/directory helloworld` |
| `-p`     | Tham số này yêu cầu Docker liên kết (bind/map) một cổng trên hệ điều hành máy chủ với một cổng đang được mở (expose) trong container. Bạn sẽ dùng tùy chọn này nếu chạy một ứng dụng hoặc dịch vụ (như web server) trong container và muốn truy cập vào nó thông qua địa chỉ IP. | EXPOSE                       | `docker run -p 80:80 webserver`                                    |
| `--rm`   | Tham số này yêu cầu Docker tự động xóa bỏ container ngay sau khi container hoàn thành tác vụ được giao và dừng lại.                                                                                                                                                              | N/A                          | `docker run --rm helloworld`                                       |
| `--name` | Tham số này cho phép đặt một tên thân thiện, dễ nhớ cho container. Khi chạy container mà không có tùy chọn này, Docker sẽ tự đặt tên ngẫu nhiên gồm 2 từ. Ta có thể dùng tùy chọn này để đặt tên container theo ứng dụng mà nó đang chạy.                                        | N/A                          | `docker run --name helloworld`                                     |

Đây chỉ là một số tham số phổ biến khi chạy container. Hầu hết các tham số cần dùng sẽ phụ thuộc vào cách container được xây dựng. Tuy nhiên, các tham số như `--rm` và `--name` sẽ chỉ dẫn cho Docker cách quản lý vòng đời của container khi chạy. Một số tham số khác bao gồm (nhưng không giới hạn ở!):

- Chỉ định card mạng (network adapter / network) mà container nên sử dụng.
- Quyền hạn/khả năng (capabilities) mà container được phép truy cập. Nội dung này được đề cập trong room "[Docker Rodeo](https://tryhackme.com/room/dockerrodeo)" trên TryHackMe.
- Lưu trữ một giá trị vào biến môi trường (environment variable).

Nếu bạn muốn khám phá thêm về các tham số này, hãy tham khảo [Tài liệu về Docker run](https://docs.docker.com/engine/reference/run/).

### Liệt kê các Container đang chạy (Listing Running Containers)

Để liệt kê các container đang chạy, chúng ta có thể sử dụng lệnh `docker ps`. Lệnh này sẽ hiển thị danh sách các container hiện đang chạy — ví dụ như sau:

Terminal hiển thị danh sách các container đang chạy cùng thông tin của chúng:

```shell-session
cmnatic@thm:~/intro-to-docker$ docker ps
CONTAINER ID   IMAGE                           COMMAND        CREATED        STATUS      PORTS     NAMES                                                                                      
                             
a913a8f6e30f   cmnatic/helloworld:latest   "sleep"   1 months ago   Up 3 days   0.0.0.0:8000->8000/tcp   helloworld
cmnatic@thm:~/intro-to-docker$
```

Lệnh này cũng hiển thị các thông tin chi tiết về container, bao gồm:

- Container ID (ID của container)
- Lệnh mà container đang thực thi (Command)
- Thời điểm container được tạo (Created)
- Container đã chạy được bao lâu (Status/Uptime)
- Các cổng được ánh xạ (Ports mapped)
- Tên của container (Name)

**Mẹo (Tip):** Để liệt kê **tất cả** các container (kể cả những container đã dừng/tắt), bạn có thể dùng lệnh `docker ps -a`:

Terminal hiển thị danh sách TẤT CẢ các container và thông tin của chúng:

```shell-session
cmnatic@thm:~/intro-to-docker$ docker ps -a
CONTAINER ID   IMAGE                             COMMAND                  CREATED             STATUS     PORTS    NAMES                                                                                  
00ba1eed0826   gobuster:cmnatic                  "./gobuster dir -url…"   an hour ago   Exited an hour ago practical_khayyam
```


# 4 -Intro to Dockerfile

Dockerfile đóng vai trò cốt lõi trong Docker. Dockerfile là một tệp văn bản có định dạng chuẩn, về bản chất đóng vai trò như một bản hướng dẫn chi tiết quy định những gì container cần làm và cuối cùng lắp ráp (assemble) thành một Docker image.

Bạn sử dụng Dockerfile để chứa các câu lệnh mà container sẽ thực thi trong quá trình build. Để bắt đầu làm quen với Dockerfile, chúng ta cần nắm được một số cú pháp và chỉ thị cơ bản. Dockerfile được định dạng theo cấu trúc sau:

`INSTRUCTION argument`

Đầu tiên, hãy điểm qua một số chỉ thị quan trọng:

| Chỉ thị (Instruction) | Mô tả (Description)                                                                                                                            | Ví dụ (Example)                                                                             |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| `FROM`                    | Chỉ thị này thiết lập giai đoạn build (build stage) cũng như chọn base image (hệ điều hành gốc). Mọi Dockerfile đều phải bắt đầu bằng chỉ thị này. | `FROM ubuntu`                                                                                   |
| `RUN`                     | Chỉ thị này sẽ thực thi các câu lệnh bên trong container và tạo ra một lớp (layer) mới.                                                            | `RUN whoami`                                                                                    |
| `COPY`                    | Chỉ thị này sao chép tệp từ hệ thống máy chủ cục bộ vào thư mục làm việc trong container (cú pháp tương tự lệnh `cp`).                             | `COPY /home/cmnatic/myfolder/app/`                                                              |
| `WORKDIR`                 | Chỉ thị này thiết lập thư mục làm việc (working directory) của container (tương tự như dùng lệnh `cd` trên Linux).                                 | `WORKDIR /` <br>(thiết lập về thư mục gốc của filesystem trong container)                       |
| `CMD`                     | Chỉ thị này xác định lệnh mặc định sẽ chạy khi container khởi động (thường dùng để chạy một service hoặc application).                             | `CMD /bin/sh -c script.sh`                                                                      |
| `EXPOSE`                  | Chỉ thị này dùng để báo hiệu cho người chạy container biết cổng nào nên được xuất bản (publish/map) khi khởi chạy container.                       | `EXPOSE 80`<br><br>(báo cho người chạy container publish cổng 80, ví dụ: `docker run -p 80:80`) |

Bây giờ chúng ta đã hiểu các chỉ thị cốt lõi tạo nên một Dockerfile, hãy xem một ví dụ thực tế. Trước hết, tôi sẽ giải thích những gì tôi muốn container này thực hiện:

1. Sử dụng hệ điều hành “Ubuntu” (phiên bản 22.04) làm nền tảng (base).
2. Thiết lập thư mục làm việc là thư mục gốc (`/`) của container.
3. Tạo tệp văn bản “helloworld.txt”.

```yml
# ĐÂY LÀ DÒNG CHÚ THÍCH (COMMENT)
# Sử dụng Ubuntu 22.04 làm hệ điều hành gốc của container
FROM ubuntu:22.04

# Thiết lập thư mục làm việc về thư mục gốc của container
WORKDIR / 

# Tạo tệp helloworld.txt
RUN touch helloworld.txt
```

Hãy nhớ rằng, các lệnh bạn có thể chạy qua chỉ thị `RUN` sẽ phụ thuộc vào hệ điều hành bạn khai báo trong chỉ thị `FROM`. (Trong ví dụ này, tôi đã chọn Ubuntu. Cần lưu ý rằng các hệ điều hành được sử dụng trong container thường rất tối giản (minimal). Nghĩa là đừng kỳ vọng mọi câu lệnh đều có sẵn ngay từ đầu — ngay cả những lệnh quen thuộc như *curl*, *ping*, v.v. cũng có thể cần phải cài đặt thủ công).

### Xây dựng Container đầu tiên của bạn (Building Your First Container)

Khi đã có Dockerfile, chúng ta có thể tạo một image bằng lệnh `docker build`. Lệnh này yêu cầu một số thông tin:

1. Bạn có muốn tự đặt tên cho image hay không (chúng ta sẽ dùng tham số `-t` (tag)).
2. Tên bạn sẽ đặt cho image.
3. Vị trí của Dockerfile bạn muốn dùng để build.

Tôi sẽ đưa ra kịch bản và sau đó giải thích câu lệnh tương ứng. Giả sử chúng ta muốn build một image — hãy điền các thông tin cần thiết:

1. Chúng ta sẽ tự đặt tên cho nó, vì vậy sẽ dùng tham số `-t`.
2. Tên image ta muốn đặt là `helloworld`.
3. Dockerfile nằm ngay trong thư mục làm việc hiện tại (`.`).

Dockerfile mà chúng ta sẽ build như sau:

```yml
# Sử dụng Ubuntu 22.04 làm hệ điều hành gốc của container
FROM ubuntu:22.04

# Thiết lập thư mục làm việc về thư mục gốc của container
WORKDIR / 

# Tạo tệp helloworld.txt
RUN touch helloworld.txt
```

Câu lệnh build sẽ như sau: `docker build -t helloworld .` (dấu chấm `.` báo cho Docker tìm kiếm Dockerfile trong thư mục làm việc hiện tại). Nếu nhập lệnh chính xác, ta sẽ thấy Docker bắt đầu quá trình build image:

Terminal hiển thị quá trình build image "helloworld":

```shell-session
cmnatic@thm:~$ docker build -t helloworld .
Sending build context to Docker daemon  4.778MB
Step 1/3 : FROM ubuntu:22.04
22.04: Pulling from library/ubuntu
2b55860d4c66: Pull complete
Digest: sha256:20fa2d7bb4de7723f542be5923b06c4d704370f0390e4ae9e1c833c8785644c1
Status: Downloaded newer image for ubuntu:22.04
 ---> 2dc39ba059dc
Step 2/3 : WORKDIR /
 ---> Running in 64d497097f8a
Removing intermediate container 64d497097f8a
 ---> d6bd1253fd4e
Step 3/3 : RUN touch helloworld.txt
 ---> Running in 54e94c9774be
Removing intermediate container 54e94c9774be
 ---> 4b11fc80fdd5
Successfully built 4b11fc80fdd5
Successfully tagged helloworld:latest
cmnatic@thm:~$
```

Tuyệt vời! Quá trình build đã thành công. Bây giờ hãy dùng lệnh `docker image ls` để kiểm tra xem image đã được tạo hay chưa:

Sử dụng lệnh "docker image ls" để xác nhận image đã được build thành công:

```shell-session
cmnatic@thm:~$ docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
helloworld   latest    4b11fc80fdd5   2 minutes ago   77.8MB
ubuntu       22.04     2dc39ba059dc   10 days ago     77.8MB
cmnatic@thm:~$
```

**Lưu ý:** Bất kỳ hệ điều hành gốc nào bạn khai báo trong chỉ thị `FROM` của Dockerfile cũng sẽ được tải về. Đó là lý do tại sao chúng ta thấy có hai image:

1. `helloworld` (image do chúng ta vừa build).
2. `ubuntu` (hệ điều hành gốc được dùng làm nền tảng trong image của chúng ta).

Bây giờ bạn đã có thể sử dụng image này để khởi chạy container. Hãy tham khảo lại phần "Chạy một Container đơn giản" để nhớ lại cách khởi động container.

### Nâng cấp Dockerfile của chúng ta (Levelling up Our Dockerfile)

Hãy nâng cấp Dockerfile lên một mức độ thực tế hơn. Cho đến nay, container của chúng ta mới chỉ tạo ra một tệp — điều đó chưa thực sự hữu ích! Trong Dockerfile tiếp theo, tôi sẽ:

1. Sử dụng Ubuntu 22.04 làm hệ điều hành gốc cho container.
2. Cài đặt web server “apache2”.
3. Thiết lập kết nối mạng. Vì đây là một web server, chúng ta cần có khả năng kết nối tới container qua mạng. Tôi sẽ thực hiện việc này bằng chỉ thị `EXPOSE` và yêu cầu container mở cổng *80*.
4. Chỉ định container khởi chạy dịch vụ “apache2” ngay khi container khởi động. Các container không có trình quản lý dịch vụ như `systemd` (đây là thiết kế có chủ ý — việc chạy nhiều ứng dụng trong cùng một container là bad practice / thực hành kém. Ví dụ, container này chỉ dành cho web server apache2 — và duy nhất apache2 mà thôi).

```yml
# ĐÂY LÀ DÒNG CHÚ THÍCH (COMMENT)
FROM ubuntu:22.04

# Cập nhật kho APT để đảm bảo cài đặt phiên bản apache2 mới nhất
RUN apt-get update -y 

# Cài đặt apache2
RUN apt-get install apache2 -y

# Khai báo container mở cổng 80 để cho phép kết nối đến web server
EXPOSE 80 

# Chỉ định lệnh chạy service apache2 khi container khởi động
CMD ["apache2ctl", "-D","FOREGROUND"]
```

Để tham khảo, lệnh build image này sẽ là `docker build -t webserver .` (giả sử Dockerfile nằm trong cùng thư mục nơi bạn chạy lệnh). Sau khi khởi chạy container với các tùy chọn phù hợp (`docker run -d --name webserver -p 80:80 webserver`), chúng ta có thể truy cập vào địa chỉ IP của máy cục bộ trên trình duyệt!

![Trang mặc định của apache2 dùng để xác nhận dịch vụ đang hoạt động](https://cdn-images.tryhackme.com/user-uploads/5de96d9ca744773ea7ef8c00/room-content/95f7f4b43b7cdf6079a71a7a3e19f937.png)  

Web server đã hoạt động! Hiện tại, Apache2 đang hiển thị trang mặc định vì chúng ta chưa thêm các tệp web của riêng mình vào container.

### Tối ưu hóa Dockerfile 

Xây dựng Docker thực sự là một nghệ thuật — và nó không chỉ dừng lại ở việc viết Dockerfile cho chạy được! Trước tiên, chúng ta cần tự hỏi tại sao việc tối ưu hóa Dockerfile lại quan trọng? Các Dockerfile cồng kềnh thường rất khó đọc, khó bảo trì và chiếm dụng nhiều dung lượng lưu trữ không cần thiết! Bạn có thể giảm kích thước của một docker image (và rút ngắn thời gian build!) bằng một số cách sau:

1. **Chỉ cài đặt các gói thiết yếu:** Điểm tuyệt vời của container là chúng hầu như trống rỗng ngay từ đầu — chúng ta có toàn quyền quyết định những gì thực sự cần thiết.
2. **Xóa các tệp bộ nhớ đệm (cache files):** Chẳng hạn như APT cache hoặc tài liệu hướng dẫn được cài kèm theo các công cụ. Code bên trong container chỉ được thực thi một lần duy nhất (lúc build!), nên không cần lưu giữ những thứ này cho các lần sau.
3. **Sử dụng base image tối giản trong chỉ thị `FROM`:** Mặc dù các OS dành cho container như Ubuntu đã khá gọn nhẹ, hãy cân nhắc sử dụng phiên bản được lược bỏ sâu hơn (ví dụ: `ubuntu:22.04-minimal`). Hoặc sử dụng Alpine Linux (kích thước có thể chỉ vỏn vẹn 5.59MB!).
4. **Giảm thiểu số lượng layer (lớp):** Tôi sẽ giải thích chi tiết điều này bên dưới.

Mỗi chỉ thị (ví dụ: `FROM`, `RUN`, v.v.) được thực thi trong một layer riêng biệt. Càng nhiều layer thì thời gian build càng lâu! Mục tiêu là giữ cho số lượng layer càng ít càng tốt. Ví dụ, hãy thử nối các lệnh `RUN` lại với nhau bằng toán tử `&&`:

**Trước khi tối ưu (Before):**

```yml
FROM ubuntu:latest
RUN apt-get update -y
RUN apt-get upgrade -y
RUN apt-get install apache2 -y
RUN apt-get install net-tools -y
```

Terminal hiển thị 5 layer của một Dockerfile đang được build:

```shell-session
cmnatic@thm:~$ docker build -t before .
--omitted for brevity--
Step 2/5 : RUN apt-get update -y
 ---> Using cache
 ---> 446962612d20
Step 3/5 : RUN apt-get upgrade -y
 ---> Running in 8bed81c695f4
--omitted for brevity--
cmnatic@thm:~$
```

**Sau khi tối ưu (After):**

```yml
FROM ubuntu:latest
RUN apt-get update -y && apt-get upgrade -y && apt-get install apache2 -y && apt-get install net-tools
```

Terminal hiển thị chỉ còn 2 layer của Dockerfile đang được build:

```shell-session
cmnatic@thm:~$ docker build -t after .
Sending build context to Docker daemon   4.78MB
Step 1/2 : FROM ubuntu
 ---> 2dc39ba059dc
Step 2/2 : RUN apt-get update -y && apt-get upgrade -y && apt-get install apache2 -y && apt-get install net-tools
 ---> Running in a4d4943bcf04
--omitted for brevity--
cmnatic@thm:~$
```

![Hình minh họa cho thấy các lệnh đã được nén thành hai layer.](https://cdn-images.tryhackme.com/user-uploads/5de96d9ca744773ea7ef8c00/room-content/743dd22bcf1f3a709ee6288d519d5930.png)  

Hãy chú ý ở đây rằng quy trình build hiện chỉ còn 2 bước (tương ứng với 2 layer, giúp việc build nhanh hơn rất nhiều). Đây chỉ là một ví dụ nhỏ của Dockerfile nên thời gian build khác biệt chưa quá lớn, nhưng với các Dockerfile lớn hơn nhiều — việc giảm số lượng layer sẽ mang lại sự cải thiện hiệu năng vượt trội trong quá trình build.

> [!NOTE] giải thích về layer trong image
> 
Trong Docker, **layer (lớp)** là một tầng hệ thống tệp tin chỉ đọc (read-only) đại diện cho những thay đổi (thêm, sửa, xóa file) được tạo ra bởi một câu lệnh trong Dockerfile.
>
> Khi bạn build một image, Docker không tạo ra một khối dữ liệu liền mạch duy nhất mà xếp chồng nhiều layer này lên nhau thông qua kỹ thuật **Union File System** (phổ biến nhất hiện nay là `Overlay2`).
> 
> ---
> 
> ### 1. Cách layer được hình thành qua ví dụ Dockerfile
> 
> Mỗi dòng lệnh làm thay đổi dữ liệu filesystem (chủ yếu là `FROM`, `COPY`, `ADD`, `RUN`) sẽ tạo ra một layer mới chứa phần chênh lệch (delta) so với layer trước đó:
> 
> ```dockerfile
> # Layer 1: Kéo base OS image (chứa cấu trúc thư mục Linux cơ bản)
> FROM alpine:3.18             
> 
> # Layer 2: Cài gói curl (chỉ chứa các file binary và thư viện mới của curl)
> RUN apk add --no-cache curl  
> 
> # Layer 3: Copy mã nguồn vào (chỉ chứa file app.py)
> COPY app.py /app/app.py      
> 
> # Không tạo layer dữ liệu mới, chỉ lưu metadata cấu hình khởi chạy
> CMD ["python3", "/app/app.py"] 
> 
> ```
> 
> Khi build, Docker xếp các layer này chồng lên nhau:
> 
> * **Layer 3 (Top - Read-only):** Chứa file `/app/app.py`.
> * **Layer 2 (Middle - Read-only):** Chứa binary `curl`.
> * **Layer 1 (Base - Read-only):** Hệ điều hành tối giản Alpine.
> 
> Khi bạn chạy lệnh `docker run`, Docker sẽ đặt thêm một tầng mỏng duy nhất có quyền ghi/đọc gọi là **Container Layer (Read-Write layer)** lên trên cùng. Mọi file bạn tạo, sửa khi app đang chạy chỉ nằm trên tầng mỏng này; các layer bên dưới của Image hoàn toàn bất biến.
> 
> ---
> 
> ### 2. Hai giá trị cốt lõi của kiến trúc Layer
> 
> #### Tận dụng bộ nhớ đệm (Build Cache)
> 
> Nếu bạn sửa code trong `app.py` và chạy lại `docker build`:
> 
> * Docker nhận thấy Layer 1 (Alpine) và Layer 2 (`apk add curl`) không hề thay đổi.
> * Nó sẽ tái sử dụng ngay hai layer này từ bộ nhớ đệm (`Using cache`) mà không cần tải lại Alpine hay cài lại curl từ đầu. Quá trình build chỉ mất chưa đầy 1 giây cho Layer 3.
> 
> #### Tiết kiệm ổ cứng và băng thông mạng
> 
> Nếu trên máy bạn có 10 ứng dụng khác nhau đều dùng `FROM alpine:3.18`, Docker chỉ lưu trữ layer của Alpine **đúng 1 lần duy nhất** trên đĩa cứng. Khi bạn tải (push/pull) image lên Docker Hub, Docker chỉ truyền tải những layer mới mà máy nhận chưa có.
> 
> ---
> 
> ### 3. Kinh nghiệm tối ưu hóa Layer khi viết Dockerfile
> 
> * **Gộp các câu lệnh `RUN` bằng dấu `&&`:** Mỗi lệnh `RUN` tạo ra 1 layer riêng. Nếu bạn tải gói ở lệnh 1 và xóa cache ở lệnh 2, dung lượng của gói đó vẫn bị kẹt vĩnh viễn ở Layer 1. Hãy gộp lại:
> ```dockerfile
> # TỐT: Gom vào 1 layer duy nhất, xóa rác ngay trước khi đóng layer
> RUN apt-get update && apt-get install -y nginx && rm -rf /var/lib/apt/lists/*
> 
> ```
> 
> 
> * **Sắp xếp thứ tự thông minh (Tối ưu Cache):** Đặt các chỉ dẫn ít thay đổi (cài đặt runtime, thư viện dependencies) ở phía trên; đặt các chỉ dẫn thường xuyên thay đổi (mã nguồn ứng dụng `COPY . .`) ở gần cuối cùng. Điều này giúp tránh làm mất hiệu lực của cache (cache invalidation) mỗi khi sửa code.

# 5 -Intro to Docker Compose

Trước tiên, hãy tìm hiểu Docker Compose là gì và tại sao nó đáng để học. Cho đến nay, chúng ta mới chỉ tương tác với từng container riêng lẻ. **Tóm lại, Docker Compose cho phép nhiều container (hoặc ứng dụng) tương tác với nhau khi cần thiết trong khi vẫn hoạt động cô lập với nhau.**

Có thể bạn đã nhận thấy một vấn đề với Docker từ đầu đến giờ: **các ứng dụng thường đòi hỏi nhiều dịch vụ bổ trợ để hoạt động, điều mà chúng ta không thể nhồi nhét vào trong một container duy nhất**. Ví dụ: các website động hiện đại thường sử dụng các dịch vụ như cơ sở dữ liệu (database) và máy chủ web (web server). Trong khuôn khổ bài này, chúng ta sẽ coi mỗi ứng dụng là một “microservice”.

Mặc dù chúng ta có thể khởi chạy nhiều container hoặc “microservice” riêng lẻ rồi kết nối chúng lại, nhưng việc làm thủ công từng cái một là rất rườm rà và kém hiệu quả. Docker Compose cho phép chúng ta quản lý và khởi chạy các “microservice” này như một “dịch vụ” (service) thống nhất duy nhất.

Hình minh họa dưới đây cho thấy cách các container được triển khai cùng nhau bằng Docker Compose so với Docker đơn lẻ:

![Một khối màu xanh (đại diện cho một máy tính) với nhãn docker, bị cô lập với một nhóm các khối màu xanh khác.](https://cdn-images.tryhackme.com/user-uploads/5de96d9ca744773ea7ef8c00/room-content/c7c8b38dc06c22207134fb6f58d036a0.png)

Trước khi thực hành Docker Compose, hãy điểm qua các yếu tố cơ bản khi sử dụng:

1. Chúng ta cần cài đặt Docker Compose (nó không đi kèm sẵn với Docker theo mặc định ở một số phiên bản). Việc cài đặt nằm ngoài phạm vi bài lab này vì nó thay đổi tùy theo hệ điều hành. Bạn có thể tham khảo [tài liệu cài đặt Docker Compose tại đây](https://docs.docker.com/compose/install/).
2. Chúng ta cần một tệp `docker-compose.yml` hợp lệ — chúng ta sẽ tìm hiểu ngay sau đây.
3. Hiểu biết cơ bản về cách sử dụng Docker Compose để build và quản lý container.

Một số lệnh Docker Compose thiết yếu được tóm tắt trong hình bên dưới:

![[Pasted image 20260923165104.png]]
![](https://cdn-images.tryhackme.com/user-uploads/5de96d9ca744773ea7ef8c00/room-content/4f236c94f79474475f2dc5df15146c91.png)

*Hình minh họa cho thấy hai container được triển khai dưới dạng một dịch vụ kết hợp. Hai container này **có thể** giao tiếp qua lại với nhau.*

### Nhập môn tệp docker-compose.yml 

Một tệp duy nhất để quản lý tất cả. Cấu trúc định dạng của tệp `docker-compose.yml` khác với Dockerfile. Điều quan trọng cần nhớ là YAML bắt buộc phải thụt đầu dòng (quy chuẩn tốt nhất là thụt vào 2 khoảng trắng (spaces) và phải nhất quán trên toàn bộ file!). Trước khi tạo tệp, hãy làm quen với một số chỉ thị mới cần biết:

| Chỉ thị (Instruction)          | Giải thích (Explanation)                                                                                                                                                                                                       | Ví dụ (Example)                             |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------- |
| `version`                      | Được đặt ở đầu file, dùng để xác định phiên bản Compose mà tệp docker-compose.yml này hướng tới.                                                                                                                               | `'3.3'`                                     |
| `services`                     | Chỉ thị này đánh dấu phần bắt đầu định nghĩa các container sẽ được quản lý.                                                                                                                                                    | `services:`                                 |
| `name` (thay bằng tên dịch vụ) | Nơi bạn định nghĩa container và cấu hình của nó. "name" cần được thay bằng tên thực tế của container bạn muốn tạo, ví dụ: "webserver" hoặc "database".                                                                         | `webserver`                                 |
| `build`                        | Chỉ thị này xác định đường dẫn thư mục chứa Dockerfile cho container/service này (bạn cần dùng chỉ thị này hoặc chỉ thị `image`).                                                                                              | `./webserver`                               |
| `ports`                        | Chỉ thị này ánh xạ và xuất bản cổng từ máy host vào cổng được expose của container (tùy thuộc vào image/Dockerfile).                                                                                                           | `'80:80'`                                   |
| `volumes`                      | Chỉ thị này liệt kê các thư mục trên hệ điều hành host sẽ được mount vào bên trong container.                                                                                                                                  | `'./home/cmnatic/webserver/:/var/www/html'` |
| `environment`                  | Chỉ thị này dùng để truyền các biến môi trường (lưu ý: không an toàn tuyệt đối cho secret), ví dụ: mật khẩu, tên người dùng, cấu hình múi giờ, v.v.                                                                            | `MYSQL_ROOT_PASSWORD=helloworld`            |
| `image`                        | Chỉ thị này xác định container sẽ được tạo từ image có sẵn nào (bạn cần dùng chỉ thị này hoặc `build`).                                                                                                                        | `mysql:latest`                              |
| `networks`                     | Chỉ thị này xác định các mạng (network) mà container sẽ tham gia. Một container có thể thuộc nhiều mạng khác nhau (ví dụ: web server chỉ có thể kết nối với một database, nhưng database có thể kết nối với nhiều web server). | `ecommerce`                                 |

***Lưu ý**: Đây chỉ là một số chỉ thị cơ bản. Hãy xem [tài liệu compose file](https://docs.docker.com/compose/compose-file/) để biết toàn bộ các chỉ thị có thể dùng.*

Với những kiến thức trên, hãy cùng xem tệp docker-compose.yml đầu tiên. Tệp `docker-compose.yml` này giả định kịch bản sau:

1. Chúng ta sẽ chạy một máy chủ web (đặt tên là `web`) từ kịch bản đã đề cập trước đó.
2. Chúng ta sẽ chạy một máy chủ cơ sở dữ liệu (đặt tên là `database`) từ kịch bản trước.
3. Máy chủ web sẽ được build bằng Dockerfile của nó, nhưng máy chủ cơ sở dữ liệu sẽ dùng một image đã có sẵn (MySQL).
4. Các container sẽ được kết nối mạng để giao tiếp với nhau (mạng này có tên là `ecommerce`).
5. Cấu trúc thư mục của chúng ta như sau:
   - `docker-compose.yml`
   - `web/Dockerfile`

Dưới đây là nội dung của tệp `docker-compose.yml` (nhắc lại: cực kỳ quan trọng là phải chú ý đến việc thụt đầu dòng):

```yml
version: '3.3'
services:
  web:
    build: ./web
    networks:
      - ecommerce
    ports:
      - '80:80'


  database:
    image: mysql:latest
    networks:
      - ecommerce
    environment:
      - MYSQL_DATABASE=ecommerce
      - MYSQL_USERNAME=root
      - MYSQL_ROOT_PASSWORD=helloword
    
networks:
  ecommerce:

```

---

# 6-Giới thiệu về Docker Socket (Intro to Docker Socket)

Phần này sẽ **giải thích cách Docker tương tác giữa hệ điều hành máy chủ và container.** Khi bạn cài đặt Docker, sẽ có hai chương trình được cài đặt:

1. **Docker Client**
2. **Docker Server** (hay Docker Daemon)

Docker hoạt động theo mô hình Client/Server. Cụ thể, hai chương trình này giao tiếp với nhau để tạo nên nền tảng Docker hoàn chỉnh. Docker thực hiện việc liên lạc này thông qua một cơ chế gọi là **socket**. Socket là một tính năng thiết yếu của hệ điều hành cho phép dữ liệu được truyền thông qua lại.

Ví dụ, khi sử dụng một chương trình chat, có thể có hai socket:

1. Một socket để lưu trữ tin nhắn mà bạn đang gửi đi.
2. Một socket để lưu trữ tin nhắn mà người khác đang gửi cho bạn.

Chương trình sẽ tương tác với hai socket này để lưu trữ hoặc lấy dữ liệu bên trong chúng! Một socket có thể là một kết nối mạng (network connection) hoặc được biểu diễn dưới dạng một tệp (file socket / unix domain socket). Điều quan trọng cần biết về socket là chúng cho phép **Giao tiếp liên tiến trình** (IPC - Interprocess Communication). Điều này đơn giản có nghĩa là các tiến trình khác nhau trên hệ điều hành có thể giao tiếp, trao đổi dữ liệu với nhau!

Trong ngữ cảnh của Docker, Docker Server thực chất đóng vai trò như một API. Docker Server sử dụng API này để **lắng nghe (listen)** các yêu cầu, trong khi Docker Client sử dụng API để **gửi (send)** các yêu cầu.

Ví dụ, hãy xem xét lệnh này: `docker run helloworld`. Docker Client sẽ gửi yêu cầu đến Docker Server để chạy một container sử dụng image "helloworld". Mặc dù cách giải thích này khá cơ bản, nhưng đó chính là nguyên lý cốt lõi trong cách Docker vận hành.

Hãy xem sơ đồ sau để thấy quy trình này diễn ra trên thực tế:

![Minh họa luồng tương tác của Docker thông qua tệp docker.sock trên hệ điều hành](https://cdn-images.tryhackme.com/user-uploads/5de96d9ca744773ea7ef8c00/room-content/7ef5f80912c890645b102b28a23b9b8b.png)

Điều thú vị là nhờ cơ chế này, chúng ta có thể tương tác với Docker Server bằng các công cụ như lệnh `curl` hoặc các công cụ phát triển API như Postman. Việc sử dụng chúng nằm ngoài phạm vi của bài lab này, nhưng tôi sẽ minh họa cách giao tiếp với Docker Server bằng Postman để liệt kê tất cả các image hiện có trên hệ điều hành:

![Danh sách các Docker image trên hệ điều hành được lấy thông qua Postman](https://resources.cmnatic.co.uk/TryHackMe/rooms/docker-rodeo/dockerregistry/catalog1.png)

Cuối cùng, điều tối quan trọng cần lưu ý là máy chủ chạy Docker có thể được cấu hình để tiếp nhận các lệnh được gửi từ một thiết bị khác qua mạng. Đây là một **lỗ hổng bảo mật cực kỳ nguy hiểm** nếu không được cấu hình bảo mật đúng cách, vì kẻ tấn công có thể từ xa dừng, khởi động hoặc can thiệp trực tiếp vào các Docker container và thậm chí leo quyền kiểm soát toàn bộ máy chủ. Dẫu vậy, vẫn có những trường hợp sử dụng mà tính năng này của Docker cực kỳ hữu ích! Chúng ta sẽ tìm hiểu sâu hơn về nội dung này trong các bài lab tiếp theo!