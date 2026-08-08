

[[note_docker]]
Docker là một nền tảng dùng để **đóng gói, phân phối và chạy ứng dụng trong container**.

Container là một môi trường chạy nhẹ, chứa gần như mọi thứ ứng dụng cần: mã nguồn, thư viện, runtime, biến môi trường và cấu hình. Nhờ vậy, ứng dụng có thể chạy nhất quán trên máy lập trình viên, server test, production hoặc cloud.

Ví dụ đơn giản:

```bash
docker run nginx
```

Lệnh này tải và chạy web server Nginx trong một container, không cần bạn cài Nginx trực tiếp lên máy.

Khác biệt chính:

**Máy ảo** mô phỏng cả một hệ điều hành, nên nặng hơn.  
**Docker container** dùng chung kernel với hệ điều hành host, nên nhẹ hơn, khởi động nhanh hơn và dễ nhân bản hơn.

Docker thường dùng để:

- Chạy ứng dụng mà không làm bẩn máy thật.
    
- Đảm bảo “chạy được trên máy tôi” cũng chạy được trên server.
    
- Triển khai microservices.
    
- Dựng môi trường dev/test nhanh bằng `Dockerfile` và `docker-compose`.
    
- Đóng gói ứng dụng cùng dependency để deploy dễ hơn.
    

Một số khái niệm cốt lõi:

```text
Dockerfile  → công thức để build image
Image       → bản đóng gói bất biến của ứng dụng
Container   → một instance đang chạy từ image
Docker Hub  → nơi lưu và chia sẻ image
```

Ví dụ luồng cơ bản:

```bash
docker build -t my-app .
docker run -p 3000:3000 my-app
```

Nói ngắn gọn: **Docker giúp bạn đóng gói ứng dụng thành một “hộp chạy được ở mọi nơi”, miễn là nơi đó có Docker.**
![[Pasted image 20260430041929.png]]