sau khi gom hết lab lại mình thấy...
nên học từ dễ tới khó thế này mới đỡ ngợp

### 0. nền
- [[lys thuyet SSRF]]
-> đọc trước để hiểu server fetch hộ là sao
- [[định nghĩa SSRF]]
-> bản định nghĩa gốc
- [[intro to ssrf]]
-> bản intro + 4 vector thao túng URL
- [[0-roadmap SSRF]]
-> map tổng

### 1. [[Lab-Basic SSRF against localhost]]
-> lab vỡ lòng, đổi stockApi thành localhost/admin rồi xóa carlos

### 2. [[Lab-SSRF against backend system]]
-> nâng 1 nấc, quét dải 192.168.0.x tìm admin ẩn port 8080

### 3. [[Lab-Blind SSRF OAST]]
-> đổi gió sang blind, chèn Collaborator vào Referer rồi hóng callback

### 4. [[Lab-SSRF blacklist bypass]]
-> localhost bị chặn, bypass bằng 127.1 + double encode admin

### 5. [[Lab-SSRF open redirect bypass]]
-> whitelist chặn host lạ, mượn /product/nextProduct để redirect vào internal

### 6. [[Lab-SSRF whitelist bypass]]
-> khoai nhất, parser confusion với @ và %2523 để lừa whitelist

> [!NOTE]
> cách mình học:
> - làm theo đúng thứ tự trên
> - mỗi lab xong tự viết lại request thật
> - kẹt thì quay về [[lys thuyet SSRF]] check payload

# LAB nâng cao
- [[Lab-Blind SSRF Shellshock]] -> blind hoàn toàn, nhét `() { :; };` vào User-Agent, confirm bằng sleep rồi exfil về collaborator.
