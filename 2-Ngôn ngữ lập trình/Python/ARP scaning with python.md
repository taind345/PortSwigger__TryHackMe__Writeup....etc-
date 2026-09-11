1- Mục tiêu: Viết script Python dò các máy đang sống trong mạng nội bộ bằng **ARP**, thay vì dùng ping (ICMP).

2- Vì sao dùng ARP thay vì ping?
- Ping dùng ICMP, thường bị firewall chặn → máy sống mà không thấy.
- ARP hoạt động ở Layer 2, mọi thiết bị trong mạng **buộc phải trả lời**, không thể trốn.

3- Cách ARP hoạt động (ví dụ đời thường):
Giống cô giáo điểm danh: đọc tên IP, ai có IP đó phải giơ tay báo MAC. Không giơ tay = không tồn tại.

### Cài Scapy
```bash
sudo apt install python3-scapy
# hoặc
pip3 install scapy
```
Cần chạy `sudo` vì ARP ở tầng thấp.

### Bước 1: Import
```python
from scapy.all import *
```
Wildcard import thường bị khuyên tránh, nhưng Scapy là ngoại lệ vì nó thiết kế để dùng kiểu này.

### Bước 2: Tạo gói tin ARP
```python
def build_arp_packet(ip_range):
    broadcast_mac = "ff:ff:ff:ff:ff:ff"
    ether_layer = Ether(dst=broadcast_mac)
    arp_layer = ARP(pdst=ip_range)
    return ether_layer / arp_layer
```
Giải thích:
- `Ether(dst="ff:ff:ff:ff:ff:ff")` → gửi cho **tất cả** máy trong mạng.
- `ARP(pdst=ip_range)` → hỏi "ai có IP trong dải này?".
- Dấu `/` xếp 2 lớp lại thành 1 gói.


### Bước 3: Gửi và nhận
```python
def scan_network(ip_range, interface="eth0", timeout=2):
    packet = build_arp_packet(ip_range)
    answered, unanswered = srp(packet, timeout=timeout, iface=interface, inter=0.1, verbose=False)

    hosts = []
    for sent, received in answered:
        ip = received[ARP].psrc
        mac = received[Ether].src
        hosts.append((ip, mac))

    return hosts
```
Giải thích:
- `srp()` gửi gói ở Layer 2, trả về 2 danh sách: ai trả lời và ai không.
- Với mỗi máy trả lời, lấy IP (`psrc`) và MAC (`src`).
- `inter=0.1` → chờ 0.1s giữa các gói, tránh làm ngập mạng.
- `verbose=False` → tắt log rác.

### Bước 4: Script hoàn chỉnh
```python
from scapy.all import *
import sys

def build_arp_packet(ip_range):
    broadcast_mac = "ff:ff:ff:ff:ff:ff"
    ether_layer = Ether(dst=broadcast_mac)
    arp_layer = ARP(pdst=ip_range)
    return ether_layer / arp_layer

def scan_network(ip_range, interface="eth0", timeout=2):
    packet = build_arp_packet(ip_range)
    answered, unanswered = srp(packet, timeout=timeout, iface=interface, inter=0.1, verbose=False)

    hosts = []
    for sent, received in answered:
        ip = received[ARP].psrc
        mac = received[Ether].src
        hosts.append((ip, mac))

    return hosts

def main():
	# xử lý cái đầu vào tham số
    if len(sys.argv) < 2:
        print(f"Usage: sudo python3 {sys.argv[0]} <ip_range> [interface]")
        sys.exit(1)
	
	#lấy dải ip và interface từ tham số
    ip_range = sys.argv[1]
    interface = sys.argv[2] if len(sys.argv) > 2 else "eth0"

    print(f"[*] Scanning {ip_range} on interface {interface}...")
    hosts = scan_network(ip_range, interface)#hàm này có tác dụng gửi gói tin arp
	# in ra kết quả
    if hosts:
        print(f"\n[*] Found {len(hosts)} live host(s):\n")
        print(f"{'IP Address':<20}{'MAC Address':<20}")
        print("-" * 40)
        for ip, mac in hosts:
            print(f"{ip:<20}{mac:<20}")
    else:
        print("[!] No hosts found.")

main()
```
Chạy:
```bash
sudo python3 arp_scanner.py 10.10.10.0/24 eth0
```
>*=> tức là gửi đầu vào là dải mạng 10.10.10.x và inerface*

Kết quả mẫu:
```
[*] Found 3 live host(s):

IP Address          MAC Address
----------------------------------------
10.10.10.1          02:42:0a:0a:0a:01
10.10.10.5          02:42:0a:0a:0a:05
10.10.10.10         02:42:0a:0a:0a:0a
```
Lưu ý: `{ip:<20}` canh trái trong 20 ký tự, tạo bảng gọn gàng.


4- Góc nhìn Defender:
- ARP scan ít bị phát hiện hơn ping, nhưng vẫn có dấu hiệu: **ARP storm** (nhiều gói ARP cùng lúc).
- NIDS như Snort/Suricata phát hiện được.
- Phòng thủ: Dynamic ARP Inspection (DAI) trên switch, giám sát broadcast bất thường.

5- Tóm gọn:
- Ping yếu vì bị chặn. ARP mạnh vì bắt buộc phải trả lời.
- Scapy dùng để tạo và gửi gói ARP.
- Chạy `sudo` vì cần quyền tầng thấp.
- Kết quả là bảng IP + MAC của mọi máy sống trong mạng.