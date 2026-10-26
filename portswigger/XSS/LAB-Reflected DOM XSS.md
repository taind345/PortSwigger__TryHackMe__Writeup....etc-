# Writeup: Reflected DOM XSS — PortSwigger Web Security Academy

Lab này là một case reflected DOM XSS khá kinh điển. Mình làm theo hướng mở Burp lên, bật Intercept, rồi xem server trả về gì và JS xử lý cái đó như thế nào.

## Các bước mình làm

1. Mở Burp Suite, bật **Proxy → Intercept**.
2. Quay lại lab, vào thanh search và nhập thử một chuỗi bất kỳ, ví dụ `XSS`.
3. Quay lại Burp, forward request.
4. Trong response, mình thấy chuỗi search được reflect vào một JSON response tên là `search-results`.
5. Mở file `searchResults.js` từ **Site Map** thì thấy response JSON này được đưa vào `eval()`. Đây là sink nguy hiểm.
6. Thử vài payload khác nhau thì thấy server có escape dấu `"` thành `\"`, nhưng lại **không escape dấu `\`**. Đây là điểm mấu chốt.

## Payload

Mình dùng payload:

```text
\"-alert(1)}//
```

## Giải thích

Vì server escape dấu `"` bằng cách thêm `\` phía trước, còn dấu `\` mình gửi vào thì không bị xử lý, nên khi JSON response được tạo ra, đoạn `\"` ban đầu bị biến thành `\\"`.

Kết quả response kiểu như sau:

```json
{"searchTerm":"\\"-alert(1)}//", "results":[]}
```

Lúc này cặp `\\` làm việc escape bị vô hiệu hóa. Dấu `"` phía sau trở thành dấu đóng string thật sự, nên chuỗi searchTerm bị đóng sớm. Phần `-alert(1)` nằm bên ngoài string, dấu `-` đóng vai trò toán tử để tách expression, còn `}` đóng object sớm và `//` comment phần còn lại của JSON.

Khi `eval()` xử lý response này, `alert(1)` được gọi và lab solved.

![[Pasted image 20261020230544.png]]