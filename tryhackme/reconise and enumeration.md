### 📘 Recon & Enumeration – Nắm bắt mục tiêu

**Nguyên tắc:** Luôn bắt đầu bằng trinh sát, không đoán mò.
###  Các bước thực hiện

| Bước                  | Lệnh/Công cụ                                | Kết quả chính                                                                           |
| --------------------- | ------------------------------------------- | --------------------------------------------------------------------------------------- |
| **Quét cổng**         | `nmap -sV -sC -p- <IP>`                     | 22(SSH), 80(HTTP Apache+PHP), 3306(MySQL), 8080(HTTP mặc định)                          |
| **Xem header HTTP**   | `curl -I <IP>`                              | Apache 2.4.58, PHPSESSID (dùng PHP session)                                             |
| **Dò thư mục ẩn**     | `gobuster dir -u <IP> -w <wordlist> -x php` | `/admin`, `/api`, `/uploads`, `/reset.php`, `/profile.php`, `/dashboard.php`, `/config` |
| **Đăng ký/đăng nhập** | Dùng form có sẵn                            | Vào được dashboard, thấy chức năng người dùng                                           |
| **Khám phá API**      | `curl <IP>/api/`                            | API tự liệt kê endpoint: `/api/user`, `/api/jobs`, `/api/applications`                  |
>*tức là dùng gobusster để dò thư mục --> dùng curl để xem các endpoint ứng với thư mục đó*

-nmap:
```shell
root@tryhackme:~# nmap -sV -sC -p- MACHINE_IP
Starting Nmap 7.80 ( https://nmap.org ) at 2026-03-27 16:40 GMT
mass_dns: warning: Unable to open /etc/resolv.conf. Try using --system-dns or specify valid servers with --dns-servers
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for MACHINE_IP
Host is up (0.00015s latency).
Not shown: 65531 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.58 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: RecruitX \xE2\x80\x94 Home
3306/tcp open  mysql   MySQL (unauthorized)
8080/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: 06:B2:88:D5:C7:67 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.54 seconds
```
-curl :
```shell
root@tryhackme:~# curl -I http://MACHINE_IP
HTTP/1.1 200 OK
Date: Fri, 27 Mar 2026 16:43:09 GMT
Server: Apache/2.4.58 (Ubuntu)
Set-Cookie: PHPSESSID=f05vg10cq16k3kq5vpurgpioqb; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Type: text/html; charset=UTF-8
```
-gobuster:
```tex
root@tryhackme:~# gobuster dir -u http://MACHINE_IP -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php -x php
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://MACHINE_IP
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 277]
/index.php            (Status: 200) [Size: 21600]
/profile.php          (Status: 302) [Size: 0] 
/login.php            (Status: 200) [Size: 15107]
/jobs.php             (Status: 200) [Size: 20288]
/uploads              (Status: 301) [Size: 314] 
/data                 (Status: 403) [Size: 277]
/admin                (Status: 301) [Size: 312] 
/test                 (Status: 200) [Size: 705]
/includes             (Status: 301) [Size: 315]
/api                  (Status: 301) [Size: 310] 
/logout.php           (Status: 302) [Size: 0] 
/config               (Status: 301) [Size: 313] 
/dashboard.php        (Status: 302) [Size: 0] 
/register.php         (Status: 200) [Size: 17384]
/reset.php            (Status: 200) [Size: 14408]
/.php                 (Status: 403) [Size: 277]
Progress: 175328 / 175330 (100.00%)
===============================================================
Finished
===============================================================
```
### 📝 Bài tập nhanh

1. Tự quét lại các cổng, dùng `-A` để lấy thêm chi tiết.
2. Dùng Gobuster quét thêm phần mở rộng `.txt`, `.bak` xem có file backup/config lộ không.
3. Thử truy cập `/api/user` mà không đăng nhập, xem có trả dữ liệu không.
4. Kiểm tra `/reset.php` có hoạt động không? Giao diện ra sao?
👉 Làm xong báo thầy kết quả, đặc biệt chú ý `/api/user` và `/reset.php` – đây là manh mối cho bước tiếp.