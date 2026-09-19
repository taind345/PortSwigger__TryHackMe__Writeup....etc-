**Stored XSS (XSS lưu trữ)** xảy ra khi ứng dụng web bê nguyên dữ liệu đầu vào của kẻ tấn công đem cất vào server (thường là cất vào cơ sở dữ liệu/database). Sau đó, nó lại lôi nội dung đó ra hiển thị cho những người dùng khác mà không thèm mã hóa (escaping) tử tế, làm cho đoạn script độc hại chạy thẳng trên trình duyệt của bất kỳ vị khách nào xui xẻo bấm vào xem.

Stored XSS nguy hiểm và có sức công phá khủng khiếp hơn Reflected XSS nhiều, vì payload được "ghim" vĩnh viễn ở đó và có thể lây nhiễm cho cả đống người (từ khách vãng lai đến cả admin) theo thời gian. Kẻ tấn công có thể gài script độc hại vào một bình luận, tiểu sử cá nhân, tin nhắn, hoặc chèn vào bảng điều khiển chỉ dành cho admin; bất cứ ai (kể cả admin) lỡ dại vào xem cái trang đó, trình duyệt sẽ tự động thực thi đoạn JavaScript bị chèn. Đoạn script này có thể thó luôn session token và mượn danh người dùng để làm bậy. Mày sẽ rất hay bắt gặp lỗ hổng này nằm chình ình ở các khu vực bình luận, hồ sơ người dùng, bảng tin, phần đánh giá sản phẩm, hoặc mấy tính năng upload file có hiển thị metadata.

**Thực hành**

Truy cập vào trang Guestbook (Sổ lưu bút) ở link `[http://10.49.135.246:5000/guestbook](http://10.49.135.246:5000/guestbook)`. Tại form Comment, mày hãy dán thử cái payload test đơn giản này: `<script>alert('You are Hacked')</script>`, xong bấm Submit. Bình luận này sẽ lập tức được lưu vào hệ thống và bày ra cho tất cả những người dùng khác cùng xem.
![[Pasted image 20260919111238.png]]
F5 tải lại trang; nếu trang đó dính lỗ hổng, mày sẽ thấy ngay một hộp thoại pop-up cảnh báo nhảy xổ ra. Vì payload đã được lưu chết trong database, nên bất cứ thanh niên nào mới mò vào trang bình luận này cũng sẽ tự động chạy đoạn mã đó và dính chưởng y hệt.

**Nguyên nhân gốc rễ (Root Cause)**

Nguyên nhân là do con app này đã xuất trực tiếp dữ liệu do kẻ tấn công kiểm soát dưới dạng HTML thô (dùng mấy cái filter kiểu `{{ query|safe }}` và `{{ c.comment|safe }}`) hoặc lưu nguyên xi bình luận rác rồi in thẳng ra mà không qua bước escaping nào. Việc này giống như trải thảm đỏ cho các thẻ `<script>` (hoặc các thẻ HTML khác) thực thi trên trình duyệt của khách truy cập. Lỗi này xuất hiện vì đoạn code đã tự tay tắt mất cơ chế tự động mã hóa (automatic escaping) và ngây thơ coi mớ dữ liệu rác không đáng tin cậy kia là "đã an toàn".