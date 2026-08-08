### 1. Tìm SUID / SGID (Leo thang đặc quyền - PrivEsc)
- **Tìm file SUID (chạy bằng quyền của Owner, thường là Root):**
    Bash
    ```
    find / -type f -perm -4000 2>/dev/null
    ```
- **Tìm file SGID:**
    Bash
    ```
    find / -type f -perm -2000 2>/dev/null
    ```
- **Tìm cả SUID và SGID cùng lúc:**
    Bash
    ```
    find / -type f \( -perm -4000 -o -perm -2000 \) 2>/dev/null
    ```
### 2. Tìm File/Thư mục có quyền Ghi (Writable)
- **Thư mục bất kỳ ai cũng ghi được (World-Writable Directory):**
    Bash
    ```
    find / -type d -writable 2>/dev/null
    # hoặc
    find / -type d -perm -o+w 2>/dev/null
    ```
- **File bất kỳ ai cũng sửa được:**
    Bash
    ```
    find / -type f -perm -o+w 2>/dev/null
    ```
### 3. Săn Flag / Mã nguồn / Credential (CTF & Post-Exploitation)
- **Tìm file chứa tên "flag" hoặc "user":**
    Bash
    ```
    find / -name "*flag*" 2>/dev/null
    ```
- **Tìm các file cấu hình chứa thông tin nhạy cảm:**
    Bash
    ```
    find / -name "*.conf" -o -name "*.config" -o -name "*.php" 2>/dev/null
    ```
- **Tìm file SSH Key tiết lộ quyền riêng tư:**
    Bash
    ```
    find / -name "id_rsa" -o -name "id_dsa" 2>/dev/null
    ```
### 4. Tìm theo Thời gian & Kích thước (Phân tích hành vi / Log)
- **File được tạo/sửa đổi trong 10 phút qua (khi vừa upload webshell hoặc chạy script):**
    Bash
    ```
    find / -type f -mmin -10 2>/dev/null
    ```
- **File bị ẩn (bắt đầu bằng dấu chấm):**
 
    ```
    find /home -name ".*" 2>/dev/null
    ```
> **Mẹo:** Cụm `2>/dev/null` ở cuối luôn cần thiết để lọc bỏ các dòng báo lỗi _Permission Denied_.