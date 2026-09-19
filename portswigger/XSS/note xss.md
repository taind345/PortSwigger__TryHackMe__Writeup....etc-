- [ ] dùng escape để prevent xss
- [ ] điều chỉnh payload sao cho phù hợp với cách input được handle
	- [ ] thường input nằm ở comment, feeback,...ticket
- [ ] url parameter thường là input đầu vào

khái niệm untrusted data khá hay
- [ ] XSS Hunter Express: giúp gửi về thông tin cho mình khi chèn XSS
	- [ ] netcat listen trên máy mình ==> script +kèm http request lại máy==> khi script thực thi trên trình duyệt==> mình sẽ nhận biết được do có request trả về máy mình -->[[THM_blindxss]]
-chỉ có 4 loại xss hay gặp: blind, stored và reflect, dom
-thường thì sẽ có input filter **=>** vậy phải làm như nào?
**-->**[[xss-filter bypass]]
