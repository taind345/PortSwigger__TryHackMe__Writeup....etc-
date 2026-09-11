1- Mục tiêu: Viết 2 script Python để dò subdomain và dò thư mục/file ẩn trên web. Cả hai dùng chung một chiến thuật: đọc wordlist → tạo URL → gửi request → xem phản hồi.

2- Thư viện `requests` giúp gửi HTTP dễ dàng:
```python
import requests
```
Cài nếu thiếu: `pip3 install requests`

### Phần 1: Dò subdomain

**Bước 1 – Đọc wordlist:**
```python
def load_wordlist(filepath):
    try:
        with open(filepath, "r") as f:
            words = [line.strip() for line in f if line.strip()]
        print(f"[*] Loaded {len(words)} entries from {filepath}")
        return words
    except FileNotFoundError:
        print(f"[!] Error: '{filepath}' not found.")
        return []
```
Giải thích: Đọc file, bỏ dòng trống, xử lý lỗi file không tồn tại.
>*==> cái thằng này giúp mình đọc file ra 1 cái list word=[]*

**Bước 2 – Dò từng subdomain:**
```python
import requests

def enumerate_subdomains(domain, wordlist):
    found = []

    for sub in wordlist:
        url = f"http://{sub}.{domain}"
        try:
            requests.get(url, timeout=3)
            print(f"[+] Found: {url}")
            found.append(url)
        except requests.ConnectionError:
            pass
        except requests.Timeout:
            pass

    return found
```
Giải thích: Với mỗi từ trong wordlist, ghép thành `http://<từ>.<domain>`. Nếu kết nối thành công → subdomain tồn tại. Nếu lỗi → bỏ qua (không in lỗi vì sẽ rất nhiều).
>*=> đơn giản là dùng wordlist cho subdomain->gửi request(sub.domain)   -> nếu ok thì subdomain đó tồn tại*

**Bước 3 – Chạy chính:**
```python
import sys

def main():
	#kiem tra xem tham so dau vao co du 3 cai ko
    if len(sys.argv) != 3:
        print(f"Usage: python3 {sys.argv[0]} <domain> <wordlist>")
        sys.exit(1)
	# lay tung tham so dau vao
    domain = sys.argv[1]
    wordlist_path = sys.argv[2]

    print(f"[*] Starting subdomain enumeration for {domain}")
    wordlist = load_wordlist(wordlist_path)
	#thuc thi ham load_wordlist cho thang wordlist[]
    if not wordlist:
        print("[!] No words to test. Exiting.")
        sys.exit(1)
		
    results = enumerate_subdomains(domain, wordlist)
    print(f"\n[*] Enumeration complete. Found {len(results)} subdomain(s).")
	
main()
```

> [!NOTE] Title
> sys nó là 1 thư viện giúp thao tác với hệ điều hành
> -> còn arg là tham số truyền vào trên terminal

Chạy:
```bash
python3 subdomain_enum.py example.thm subdomains.txt
```
Kết quả mẫu:
```
[*] Starting subdomain enumeration for example.thm
[*] Loaded 15 entries from subdomains.txt
[+] Found: http://admin.example.thm
[+] Found: http://dev.example.thm
[*] Enumeration complete. Found 2 subdomain(s).
```
Lưu ý: Script này chỉ dùng cho domain (example.com), không dùng cho IP.

### Phần 2: Dò thư mục/file

Chiến thuật y hệt, chỉ đổi cách ghép URL và check status code:
```python
def enumerate_directories(target_url, wordlist, extension=".html"):
    found = []
	# cai thang url= "target_url/wordlist.html"
    for entry in wordlist:
        url = f"{target_url}/{entry}{extension}"
        # gửi thàng url này bằng request-> nếu ok thì duyệt-> ko thì pass
        try:
            r = requests.get(url, timeout=3)
            if r.status_code != 404:
                print(f"[+] {r.status_code} - {url}")
                found.append(url)
        except requests.ConnectionError:
            pass
        except requests.Timeout:
            pass

    return found
```
Giải thích: Ghép `target_url/từ.extension`. Nếu status code **khác 404** → có tồn tại. Đáng chú ý:
- 200 = OK
- 301 = chuyển hướng
- 403 = có tồn tại nhưng bị chặn (thông tin quý)

Chạy:
```bash
python3 dir_enum.py http://MACHINE_IP:8080 wordlist.txt
```
Kết quả:
```
[*] Starting directory enumeration for http://127.0.0.1:8080
[*] Loaded 58 entries from wordlist.txt
[+] 200 - http://MACHINE_IP:8080/index.html
[*] Enumeration complete. Found 1 valid path(s).
```
Thử port khác:
```bash
python3 dir_enum.py http://MACHINE_IP:8000 wordlist.txt
```
Kết quả:
```
[+] 200 - http://MACHINE_IP:8000/admin.html
```


3- Góc nhìn Defender: Dò subdomain/thư mục tạo ra hàng trăm request từ một IP trong thời gian ngắn, phần lớn trả 404. WAF và IDS có thể phát hiện và chặn IP. Phòng thủ: rate limiting, giám sát lưu lượng bất thường, hạn chế DNS record công khai.

4- Tóm gọn:
- Cùng một pattern, đổi URL construction là ra 2 tool khác nhau.
- Subdomain: ghép `<từ>.<domain>`, check kết nối thành công.
- Directory: ghép `<url>/<từ>.<ext>`, check status code != 404.
- Dùng `try/except` để tránh crash, dùng `sys.argv` để nhận tham số.
![[Pasted image 20260912005109.png]]