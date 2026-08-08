# 🧠 <mark style="background:#40a9ff">SSRF Bypass Whitelist Filter</mark> 
#ssrf 
**Nguyên lý cốt lõi**: ==Bộ lọc kiểm tra URL bằng so khớp chuỗi đơn giản (regex, starts‑with, contains)==, trong <mark style="background:#d3f8b6">khi backend dùng trình phân tích URL đầy đủ (theo RFC). </mark>Sự khác biệt này cho phép ta “đánh lừa” filter.
[[whitelist bypass]]
![[whitelist bypass]]
## 🔧 Các kỹ thuật bypass chính

| Kỹ thuật               | Cách hoạt động                                                                                                                                    | Ví dụ                                                                                                              |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **1. Userinfo (`@`)**  | Đặt domain hợp lệ vào phần credentials, host thật nằm sau `@`. Backend coi phần trước `@` là userinfo.                                            | `https://trusted-host@evil.com` → filter thấy `trusted-host`, backend request đến `evil.com`                       |
| **2. Fragment (`#`)**  | Phần sau `#` (fragment) bị backend bỏ qua khi gửi request. Đặt domain hợp lệ vào fragment để qua filter.                                          | `https://evil.com#trusted-host` → filter thấy `trusted-host` trong URL, backend chỉ lấy `evil.com`                 |
| **3. DNS subdomain**   | Lợi dụng cấu trúc tên miền: tạo subdomain chứa tên domain hợp lệ, trỏ về IP của attacker. Hiệu quả nếu filter dùng `contains` hoặc `starts-with`. | `https://trusted-host.evil.com` → filter nhận dạng `trusted-host`, DNS phân giải về IP attacker                    |
| **4. URL Encoding**    | Mã hóa các ký tự quan trọng (`@`, `#`, `.`) để filter không nhận ra. Backend tự động giải mã trước khi phân tích.                                 | `https://trusted-host%40evil.com` → filter thấy `%40` (không phải `@`), backend decode thành `@` → host `evil.com` |
| **5. Double Encoding** | Nếu filter giải mã một lần nhưng backend giải mã đệ quy (2 lần), ta mã hóa hai lớp.                                                               | `.` → `%2E` → `%252E`; filter decode 1 lần ra `%2E` (an toàn), backend decode tiếp ra `.`                          |
| **6. Kết hợp**         | Kết hợp các kỹ thuật trên để tăng hiệu quả.                                                                                                       | `https://trusted-host%252Eevil.com%40evil.com/admin`                                                               |

## 🛡️ Phòng thủ hiệu quả
- **Phân tích URL chuẩn**: dùng thư viện parse (không regex tự viết).
- **Decode hoàn toàn** trước khi kiểm tra hostname.
- **So khớp chính xác** hostname, không dùng contains / starts‑with.
- **Vô hiệu hóa userinfo & fragment**: loại bỏ `@`, `#` khỏi URL đầu vào.
- **Phân giải DNS & lọc IP**: chặn private/nội bộ, đề phòng DNS rebinding.

✍️ **Ghi nhớ**: Tận dụng **sự không nhất quán** giữa hai tầng xử lý – nơi filter nhìn thấy một URL “sạch” nhưng backend lại hiểu thành host của attacker.



# 🔀 <mark style="background:#40a9ff">Bypass SSRF Filter qua Open Redirect</mark>

## Ý tưởng cốt lõi
Khi bộ lọc SSRF chỉ cho phép các domain nằm trong **whitelist**, attacker lợi dụng một **open redirect** trên chính domain được phép đó để “nhảy” đến đích cấm<mark style="background:#fff88f">. Backend tự động follow redirect mà không kiểm tra lại đích đến, do đó filter ban đầu bị vô hiệu hóa</mark>.

## 📦 Cách thức hoạt động
1. Ứng dụng cho phép gọi `stockApi=http://allowed.com/...`
2. Attacker tìm thấy một endpoint trên `allowed.com` có tham số redirect (ví dụ: `?path=...`) mà không kiểm tra đích đến.
3. Attacker gửi:
   ```
   stockApi=http://allowed.com/redirect?url=http://192.168.0.68/admin
   ```
4. Filter kiểm tra: host là `allowed.com` → **Hợp lệ**.
5. Backend thực hiện request đến `http://allowed.com/redirect?url=...`.
6. Server `allowed.com` trả về HTTP 302 với `Location: http://192.168.0.68/admin`.
7. Backend (nếu có follow redirect) sẽ tự động gửi request tiếp theo đến `http://192.168.0.68/admin`.
8. Kẻ tấn công truy cập được tài nguyên nội bộ.

## 🔧 Điều kiện khai thác thành công
- ==Ứng dụng backend **tự động follow redirect**== (hầu hết thư viện HTTP như ==`curl`, `requests`, `HttpURLConnection` đều làm vậy trừ khi tắt)==.
- ==Domain trong whitelist có một **open redirect**== (có thể nằm ở query string, path, header tùy ứng dụng).
- Redirect không bị lọc hay chặn bởi một tầng bảo vệ khác.

## 🧩 Ví dụ minh họa
**Request tấn công** (POST body):
```
stockApi=http://weliketoshop.net/product/nextProduct?currentProductId=6&path=http://192.168.0.68/admin
```
- `weliketoshop.net` nằm trong whitelist.
- `path` là tham số của open redirect trỏ đến IP nội bộ.

## 🛡️ Phòng thủ
- **Tắt follow redirect** ở phía backend khi thực hiện request do người dùng kiểm soát URL.
- Nếu cần follow, **kiểm tra domain của mỗi redirect** (phân giải DNS, so khớp whitelist) trước khi follow.
- **Giới hạn số lần redirect** (tránh vòng lặp).
- **Chặn redirect đến private IP** (dùng tính năng như `curl` với `--resolve`, hoặc tự phân giải).

✍️ **Ghi nhớ**: Open Redirect không chỉ nguy hiểm cho phishing mà còn là “chìa khoá” để vượt qua whitelist SSRF. Hãy coi open redirect trên miền tin cậy như một lỗ hổng nghiêm trọng.

## PAYLOAD  tham số redirect
```text
redirect_uri
redirect_url
redirect
url
uri
next
continue
returnTo
return_to
return_url
return
dest
destination
target
goto
go_to
to
endpoint
forward
out
view
path
dir
directory
image_url
file_url
fetch
load
feed
```