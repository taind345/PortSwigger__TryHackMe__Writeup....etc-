Dưới đây là tóm tắt kiến thức Docker căn bản, được chia nhóm ngắn gọn để bạn dễ nắm bắt nhanh nhất:
[[note_docker]]
### 1. 3 Khái niệm cốt lõi nhất

- **Image (Hình ảnh):** Là "bản thiết kế" chứa code, thư viện, và môi trường chạy. Nó là file chỉ đọc (read-only).
    
- **Container (Thùng chứa):** Là ứng dụng đang chạy thực tế được tạo ra từ Image. Môi trường này hoàn toàn cách ly với máy chủ.
    
- **Dockerfile:** Là file text chứa một danh sách các câu lệnh. Docker sẽ đọc file này để tự động build ra một Image.
    

### 2. Thành phần quản lý hệ thống

- **Volume:** Dùng để lưu trữ dữ liệu vĩnh viễn. Nếu không dùng Volume, mọi dữ liệu (như database) sẽ mất trắng khi Container bị xóa.
    
- **Network:** Mạng ảo giúp các Container khác nhau có thể "nói chuyện" được với nhau (ví dụ: Web container gọi đến Database container).
    
- ==**Docker Compose**==: Công cụ giúp bạn cấu hình và chạy nhiều Container cùng lúc chỉ bằng một lệnh thông qua file ==`docker-compose.yml`==.
    

---

### 3. Các nhóm lệnh (Command) dùng nhiều nhất

#### **Quản lý Image:**

- `docker pull <tên_image>` : Tải image từ mạng (Docker Hub) về máy.
    
- `docker build -t <tên> .` : Build một image mới từ Dockerfile ở thư mục hiện tại.
    
- `docker images` : Liệt kê các image đang có trong máy.
    
- `docker rmi <image_id>` : Xóa image.
    

#### **Quản lý Container:**

- `docker run -d -p 8080:80 <tên_image>` : Tạo và chạy container ngầm (`-d`), nối port 8080 của máy thật vào port 80 của container (`-p`).
    
- `docker ps` : Xem các container đang chạy (thêm `-a` để xem cả những cái đã tắt).
    
- `docker stop <container_id>` : Tạm dừng container.
    
- `docker rm <container_id>` : Xóa hẳn container (phải stop trước khi xóa).
    

#### **Tương tác & Debug:**

- `docker exec -it <container_id> sh` (hoặc `bash`) : "Chui" vào terminal của một container đang chạy để gõ lệnh.
    
- `docker logs <container_id>` : Xem file log để bắt lỗi nếu container bị crash.
    
- `docker stats` : Xem Container nào đang ăn bao nhiêu RAM, CPU.
    

#### **Dành cho Docker Compose:**

- `docker-compose up -d` : Chạy toàn bộ cụm container cấu hình trong file yaml.
    
- `docker-compose down` : Tắt và xóa toàn cụm.