Nếu là **lập trình thi đấu (competitive programming)** thì thực ra `datetime` của Python **rất ít dùng**. 95% bài toán chỉ cần biết vài thứ sau:

```text
datetime cho thi đấu
├── datetime.strptime()
│   └── Chuyển string → thời gian
├── datetime.strftime()
│   └── Chuyển thời gian → string
├── timedelta
│   └── Cộng/trừ ngày, giờ
└── (d2 - d1).days
    └── Tính số ngày chênh lệch
```

## 1. Đọc ngày từ chuỗi

Ví dụ input:

```text
2026-06-16
```

```python
from datetime import datetime

s = input()

d = datetime.strptime(s, "%Y-%m-%d")

print(d.year)
print(d.month)
print(d.day)
```

---

## 2. Tính số ngày giữa hai mốc thời gian

```python
from datetime import datetime

d1 = datetime.strptime(input(), "%Y-%m-%d")
d2 = datetime.strptime(input(), "%Y-%m-%d")

print((d2 - d1).days)
```

Input:

```text
2026-06-01
2026-06-16
```

Output:

```text
15
```

---

## 3. Cộng/trừ ngày

```python
from datetime import datetime, timedelta

d = datetime.strptime(input(), "%Y-%m-%d")

new_d = d + timedelta(days=7)

print(new_d.strftime("%Y-%m-%d"))
```

---

## 4. Các format cần nhớ

```text
%Y  → năm (2026)
%m  → tháng (01-12)
%d  → ngày (01-31)

%H  → giờ 24h
%M  → phút
%S  → giây
```

---

## Nếu thi ICPC/CodePTIT ==thì nhớ:==

```text
datetime tối thiểu
├── from datetime import datetime, timedelta
├── datetime.strptime(s, "%Y-%m-%d")
├── datetime.strftime("%Y-%m-%d")
├── d2 - d1
└── timedelta(days=n)
```

Ngoài ra, trong thi đấu nhiều người **không dùng `datetime`** mà tự xử lý ngày tháng, vì một số OJ cũ không cho phép hoặc để tối ưu tốc độ. Nhưng nếu đề không cấm thư viện chuẩn thì 4 thứ trên là đủ dùng cho hầu hết bài toán về thời gian.