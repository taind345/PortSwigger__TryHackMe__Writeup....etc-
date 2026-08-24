Đm mày ngợp là phải. Bọn lab nó đang dọn cho mày cái buffet thập cẩm, mỗi CVE nó ném vào mặt mày đại diện cho một lỗ hổng đặc trưng trên một hệ sinh thái (stack) khác nhau. Từ MERN (Node.js), Django (Python) đến LAMP (Apache/PHP). Nó cố tình dàn trải để mày nếm thử đủ mùi vị của giới web pentest đấy, chứ đi làm thực tế một con web đéo bao giờ ôm cả cái đống hổ lốn này.

Nhưng để đi đường dài với mảng offensive security và tự tay đào được CVE mới (0-day), mày đéo thể cưỡi ngựa xem hoa thế này được. Nghe tao phác cái chiến lược cày cuốc đây:

1. Đéo học vẹt payload, hãy móc cái **Root Cause** ra
Đừng cố nhớ cái mã CVE hay chuỗi payload `.%2e/`. Mày phải nhìn ra bản chất cốt lõi. Thằng JS chết vì cơ chế kế thừa ngu học của object (Prototype). Thằng Python Django hẹo vì dev lười xài ORM mà đi nối chuỗi SQL. Thằng Apache ngã ngửa vì lỗi thứ tự ưu tiên khi decode chuỗi. Nhìn vào nguyên lý, đéo nhìn hiện tượng. Khi hiểu bản chất, mày gặp một framework lạ hoắc mày vẫn biết cách test.
><u>-rút ra cái logic lỗi ? tức là </u>

2. Đào sâu 1 đến 2 Stack cốt lõi
Biển kiến thức rộng vãi lồn, tham là thâm. Mày định hướng đấm mạnh mảng web security thì <u>chọn mẹ 1-2 framework phổ biến (như MERN hoặc PHP/Laravel) mà cày đến tận đáy</u>. Phải hiểu ruột gan kiến trúc của nó, biết bọn dev hay lười và code ẩu ở chỗ nào thì mới bới ra bug được.

3. Chuyển sang hệ chơi Whitebox (Đọc source code)
Hack blackbox (đứng ngoài mò mẫm) thì chỉ nhai lại được đồ thừa thôi. Muốn nặn ra CVE cho riêng mình thì bắt buộc phải đọc hiểu source code. Áp dụng ngay tư duy của mấy môn như công nghệ phần mềm vào mà vẽ luồng. Phân tích data flow từ lúc user nhập input, chui vào router, đi qua middleware, chọc xuống database xem có đoạn nào đéo bọc filter không. Dấu vết 0-day nó nằm ở đấy.

4. Đu bám các bản vá (Patch Diffing)
<u>Mẹo của bọn săn bug là lân la lên Github của mấy project open-source. Rình mấy cái commit chúng nó ghi là "fix security issue".</u> Nhìn vào đoạn code tụi nó vừa vá, rồi lật ngược tư duy xem lúc chưa vá thì chọc vào kiểu đéo gì. Cày trò này não nảy số cực nhanh.

Lúc đi phỏng vấn thực tập hay vào việc thực tế, người ta ném cho mày cục source code bắt tìm lỗ hổng, chứ đéo ai rảnh đi hỏi mày thuộc mã CVE nào.

Thế giờ mày định cày chết bỏ cái stack nào trước để bắt đầu hành trình săn bug, Node.js, Python hay PHP để tao vạch đường tiếp cho?

### khó ko
Đm hỏi câu thực tế đấy. Sự thật mất lòng là game này càng ngày càng khó vãi lồn, đéo có chuyện dễ ăn như chục năm trước đâu con ạ.

Thứ nhất, hệ thống bây giờ nó chặt chẽ hơn rất nhiều. Ngày xưa gõ bừa cái dấu nháy đơn vào ô login là nổ SQL Injection, cào bừa cũng ra lỗi. Giờ thì mơ đi, các framework đời mới nó bọc tận răng rồi, tự động lọc sạch mấy cái trò XSS với SQLi cơ bản. Bọn công ty còn ốp thêm tường lửa (WAF), rồi tool quét lỗi tự động tích hợp mẹ vào quy trình đẩy code. Mấy cái lỗi rác rưởi (low-hanging fruit) bị tuyệt chủng gần hết cmnr.

Thứ hai, đúng là fix lỗi này lòi ra lỗi khác, nhưng bản chất lỗi nó đã tiến hóa rồi. Hệ thống bây giờ nó phức tạp vãi cứt, băm nát ra thành microservices, chằng chịt API gọi chéo nhau, rồi nhồi hết lên Cloud. Lỗi bây giờ nó đéo nằm ở cú pháp code đơn thuần nữa, mà nó nằm ở lỗ hổng logic và cấu hình ghép nối.

Nghĩa là sao? Code chạy đéo sai dòng nào, máy scan cũng đéo phát hiện ra, nhưng tư duy thiết kế luồng chạy bị ngu. Ví dụ mày đổi số ID trên đường link là xem được sao kê ngân hàng của thằng khác (IDOR), hoặc API giỏ hàng cho nhập số lượng âm để hệ thống tự thối lại tiền. Máy móc đéo thể scan được mấy cái này, chỉ có con người dùng não lươn lẹo mới nghĩ ra được kịch bản để lách.

Tóm lại là rào cản bước chân vào ngành giờ cao hơn hẳn. Đéo còn đất diễn cho mấy thằng "thợ gõ" chỉ biết bật tool quét quét nữa đâu. Muốn nặn ra CVE thì mày phải đào thật sâu vào ruột kiến trúc phần mềm, đọc source code (whitebox) và hiểu hệ thống ngang ngửa thằng tạo ra nó.

Thế giờ nhận ra độ chua của nó rồi, mày vẫn muốn đâm đầu vào cày cuốc hay tính quay xe sang hướng khác?
### pentest thực tế
Đm mày tỉnh đấy! Suy nghĩ thế là khôn.

Thực ra đéo ai bắt mày ra trường phải thành dị nhân đi rình mò 0-day hay săn CVE mới có lương cao đâu. Cái mảng Vulnerability Research hay Bug Bounty nó dành cho mấy thằng quái vật, não to, thích chơi hệ cày cuốc độc lập. Nghề đấy thỉnh thoảng nổ được quả đậm nhưng bấp bênh vãi lồn, có khi mút ngón tay cả tháng đéo ra bug.

Còn con đường Pentest Web mày vừa nói mới là cái cần câu cơm chuẩn bài, ổn định và ra tiền đều đặn nhất hiện nay. Đi làm doanh nghiệp thì đây là chân ái. Mày đéo cần phát minh ra lỗ hổng mới cho nhân loại, mày chỉ cần đi tìm cái ngu của thằng dev đang code con web đó.

Cụ thể đi làm Pentest nó nhàn não hơn mày nghĩ nhiều:

Thứ nhất là bắt lỗi logic (Business Logic). Framework có xịn đến mấy mà thằng dev tự viết hàm check quyền ngu thì cũng vứt. Mày chỉ cần bật con Burp Suite lên, chặn cái HTTP request, đổi tham số ID từ 1 sang 2 mà xem được thông tin của khách hàng khác (lỗi IDOR), hay sửa giỏ hàng thành giá trị âm để web tự thối lại tiền... là mày có report ăn tiền cmnr. Mấy cái này máy móc đéo scan được, cần não người lươn lẹo như anh em mình.

Thứ hai là bới móc thư viện cũ. Các công ty xài cả đống thư viện open-source, dev đéo bao giờ rảnh đi update liên tục. Việc của mày là rà xem nó có đang chạy cái bản Apache hay Node.js cũ rích dính CVE đéo nào không (y hệt mấy bài lab nãy giờ mày cày đấy). Thấy cái là bú.

Thứ ba là chuyên trị cấu hình lởm. Đôi khi lỗi đéo nằm ở code, mà do thằng quản trị mạng mở nhầm port, phân quyền sai thư mục, hoặc để lộ nguyên cái file chứa password db trên server. Mày vạch ra được là thành công.

Nghề Pentester lương cao là vì mày giúp bọn doanh nghiệp bít lại những cái lỗ hổng thực tế, bảo vệ nồi cơm cho tụi nó trước khi bị bọn hacker mũ đen nhảy vào tống tiền. Khách hàng nó ném tiền cho mày mua sự an tâm.

Cứ bám chết lấy cái định hướng Offensive với Web Security đi, hướng này lúc đéo nào cũng khát nhân lực. Cứ xách con Burp Suite lên cày nát cái chuẩn OWASP Top 10, cuối tuần rảnh thì lên mạng giã thêm mấy giải CTF để luyện phản xạ thực chiến với trau dồi tool. Làm ngon mấy cái cơ bản đấy là ra trường đi làm Pentester vểnh râu, thu nhập đéo phải nghĩ. Đéo có gì phải ngợp hết, tới bến đi con chó!