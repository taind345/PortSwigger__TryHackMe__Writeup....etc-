
### 1. Chỉ định Mục tiêu (Target)
- **1 IP:** `nmap 192.168.1.1`
- **Nhiều IP:** `nmap 192.168.1.1 192.168.1.2`
- **Cả dải mạng (Subnet):** `nmap 192.168.1.0/24`
- **Từ file text:** `nmap -iL targets.txt`
### 2. Chỉ định Cổng (Port)
- **Port cụ thể:** `nmap -p 22,80,443 <IP>`
- **Khoảng Port:** `nmap -p 1-1000 <IP>`
- **Tất cả 65,535 Ports:** `nmap -p- <IP>`
- **Top các port phổ biến:** `nmap --top-ports 100 <IP>`
### 3. Các cờ thu thập thông tin (Discovery)
- `-sV`: Quét phiên bản phần mềm (Version detection).
- `-O`: Quét hệ điều hành (OS detection).
- `-sC`: Chạy các script mặc định (Nmap Scripting Engine - NSE) để tìm lỗ hổng cơ bản.
- `-A`: Quét tổng hợp (Gộp chung của `-sV`, `-O`, `-sC` và Traceroute).
### 4. Loại hình Quét (Scan Types)
- `-sS`: TCP SYN Scan (Quét ẩn danh một nửa, phổ biến và nhanh nhất, cần quyền `sudo`).
- `-sT`: TCP Connect Scan (Hoàn thành chuỗi bắt tay 3 bước, dễ bị log lại).
- `-sU`: UDP Scan (Quét các dịch vụ UDP như DNS, SNMP - _Thường khá chậm_).
### 5. Tối ưu Tốc độ (Performance)
- `-T<0-5>`: Tốc độ quét. Mặc định là `-T3`. Thực chiến thường dùng **`-T4`** (Nhanh nhưng vẫn ổn định).
- `--min-rate <số>`: Ép Nmap gửi tối thiểu số gói tin mỗi giây (VD: `--min-rate 1000` giúp quét 65,535 ports cực nhanh).
- `-v`: Verbose (Hiển thị ngay kết quả port mở khi đang quét mà không cần đợi xong 100%).
### 6. Xuất File Báo cáo (Output)
- `-oN file.txt`: Lưu định dạng text bình thường.
- `-oG file.gnmap`: Lưu định dạng Grepable (Dễ dàng dùng lệnh `grep`, `awk` để lọc).
- `-oX file.xml`: Lưu định dạng XML.
- **`-oA name`**: Lưu ra cả 3 định dạng trên cùng lúc (Khuyên dùng khi đi dự án thật).
### 🚀 CÁC COMBO THỰC CHIẾN (NÊN LƯU LẠI)
**Combo 1: Quét nhanh tổng quan (Tìm đường vào)**
Bash
```
nmap -sC -sV -T4 -oA initial_scan <IP>
```
**Combo 2: Quét toàn bộ 65,535 Ports (Tốc độ cao)**
Bash
```
nmap -p- --min-rate 1000 -T4 -v <IP>
```
**Combo 3: Dò tìm lỗ hổng bằng Script chuyên sâu (Vuln Scan)**
Bash
```
nmap -p 80,443 --script vuln <IP>
```
**Combo 4: Quét UDP (Tìm các dịch vụ ẩn)**
Bash
```
sudo nmap -sU --top-ports 100 -T4 <IP>
```