
### 19/9
sau khi đọc bài này [[intro to ssrf]] mình có thêm vài insight khá hay:
- [ ] có thể dùng kiểu dạng na ná path traversal để có thể thay url được request tới
- [ ] biết thêm 1 kỹ thuật là **chèn dấu &**==> thực ra có rất nhiều cách thao túng url parameter ==> dấu & là ngăn cách các biến truyền vô 1 tham số ví dụ `http://tai.com/api/thongtin?ten=TAI&tuoi=21` thì tuổi và tên là 2 tham số đầu vào cho cùng 1 enpoint
- [ ] dấu hiệu nhận biết đơn giản là server truy cập tới một url, mà có khả năng url đó có thể bị sửa đổi bằng 1 url bên ngoài hoặc url nội bộ 
	 - [ ] -> url partial, webhook? , pdf, url ảnh =>reference từ bên ngoài
- [ ] redirect trong allow list


