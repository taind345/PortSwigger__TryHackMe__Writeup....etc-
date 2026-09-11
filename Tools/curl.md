Đi làm pentest hay cày lab thì nhớ ngần này cờ của curl là đủ xài:
Ok, để tao giải thích nguồn gốc mấy chữ viết tắt của cờ `curl` cho mày dễ nhớ:

- **-X** — e**X**ecute method? Thực ra là **Request method**, nhưng người ta hay nhớ là "X" vì nó chỉ định method tùy ý (GET, POST...).
- **-d** — **D**ata (dữ liệu gửi trong body).
- **-H** — **H**eader (thêm header tùy chỉnh).
- **-b** — **B**iscuit? Không, **B**rowser cookie? Gốc là **B** (cookie) — gửi cookie lên server.
- **-c** — **C**ookie jar — lưu cookie server trả về vào file.
- **-L** — **L**ocation — tự đi theo header `Location` khi redirect (301/302).
- **-i** — **I**nclude — in kèm HTTP header trong response.
- **-I** — **I**nclude header only — chỉ lấy header (HEAD request).
- **-k** — **K**ill SSL check — bỏ qua kiểm tra chứng chỉ (insecure).
- **-x** — Pro**x**y — gửi request qua proxy (ví dụ Burp).
- **-v** — **V**erbose — in chi tiết quá trình gửi/nhận.
- **-s** — **S**ilent — chế độ im lặng, không hiện progress.
- **-o** — **O**utput — ghi kết quả ra file thay vì in ra màn hình.

Cứ nhớ theo nghĩa tiếng Anh là bám được ngay. 😎


