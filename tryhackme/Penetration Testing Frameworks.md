>*phần này thuần lý thuyết*
# 1-Intro
hãy tưởng tượng bạn vừa nhận dự án penetration testing đầu tiên. khách hàng là một công ty thương mại điện tử quy mô vừa, muốn bạn đánh giá an toàn cho web application, internal network và nhận thức an ninh của nhân viên. mở laptop lên, bạn sẽ làm gì tiếp theo? chạy nmap ngay lập tức? kiểm tra trang đăng nhập xem có lỗi injection? hay thử phishing một nhân viên?

nếu không có phương pháp bài bản, một buổi penetration test sẽ nhanh chóng trở thành một tập hợp các bước kiểm tra ngẫu nhiên và hỗn loạn. bạn có thể bỏ sót các attack surface quan trọng, quên ghi chép tài liệu, hoặc đưa ra kết quả mà khách hàng không thể áp dụng. tệ hơn, bạn có thể kiểm thử nhầm các hệ thống nằm ngoài scope và gặp rắc rối về pháp lý. đây chính là vấn đề mà các penetration testing framework giải quyết.

**penetration testing framework**<u> là phương pháp luận có cấu trúc, hướng dẫn chuyên gia an ninh qua từng giai đoạn của dự án</u>, từ lập kế hoạch, xác định scope ban đầu cho đến exploitation, reporting và remediation validation. tương tự như một thanh tra xây dựng làm việc theo checklist tiêu chuẩn: họ không đi loanh quanh với hy vọng vô tình thấy lỗi, mà tuân theo quy trình hệ thống để đảm bảo mọi kết cấu, hệ thống điện và an toàn phòng cháy đều được đánh giá theo tiêu chuẩn. **penetration testing framework** cũng đóng vai trò tương tự trong đánh giá an toàn thông tin.
áp dụng phương pháp luận có cấu trúc <u>mang lại nhiều lợi ích</u>:
- *thoroughness*: đảm bảo tính toàn diện, không bỏ sót các khu vực quan trọng.
- *consistency*: đảm bảo tính nhất quán, giúp các tester khác nhau trong cùng đội ngũ đưa ra kết quả tương đồng.
- *compliance*: hỗ trợ tuân thủ quy định bằng cách căn chỉnh đợt đánh giá theo các tiêu chuẩn pháp lý.
- *communication*: cải thiện giao tiếp, giúp khách hàng, auditor và các bên liên quan dễ hiểu, tin tưởng vào quy trình.

hiện có nhiều penetration testing framework được sử dụng phổ biến, mỗi framework đều có triết lý, thế mạnh và trường hợp áp dụng riêng. trong room này, chúng ta sẽ tìm hiểu sâu về:
- *osstmm* (open source security testing methodology manual): phương pháp tiếp cận dựa trên chỉ số và khoa học.
- *wstg* (web security testing guide): framework chuẩn cho đánh giá web application.
- *nist sp* 800-115: hướng dẫn kỹ thuật kiểm thử và đánh giá an toàn của chính phủ mỹ.
- *ptes* (penetration testing execution standard): tiêu chuẩn thực tế chia theo từng giai đoạn, phản ánh cách thực hiện dự án thực tế.
- *issaf* (information systems security assessment framework): phương pháp luận có tầm ảnh hưởng lịch sử với mô hình đánh giá 9 bước chi tiết.

chúng ta cũng sẽ làm quen với *mitre att&ck*, cơ sở tri thức bổ trợ giúp hệ thống hóa các tactic và technique của kẻ tấn công. ngoài ra,<u> chúng ta sẽ điểm qua một số framework đáng chú ý khác như wasc threat classification, csa cloud controls matrix, mastg (mobile application security testing guide), crest penetration testing guidelines và cbest framework </u>để biết khi nào và ở đâu nên áp dụng chúng.

mục tiêu bài học:
- mô tả mục đích và cấu trúc của các penetration testing framework chính.
- so sánh các framework dựa trên scope, methodology và trường hợp sử dụng.
- chọn framework phù hợp cho một kịch bản dự án cụ thể.
- giải thích cách mitre att&ck bổ trợ cho các phương pháp pentest truyền thống.

# 2-OSTM
**tổng quan**
có thể bạn đã nghe câu "bạn không thể quản lý những gì bạn không thể đo lường". trong hầu hết các ngành kỹ thuật, việc định lượng là điều hiển nhiên: một kỹ sư kết cấu không đánh giá một cây cầu "an toàn" dựa trên cảm tính. tuy nhiên trong penetration testing, <u>các phát hiện thường được trình bày dưới dạng mô tả định tính</u> mang tính chủ quan. <u>open source security testing methodology manual (osstmm) được tạo ra để thay đổi điều đó.</u>

được phát triển bởi institute for security and open methodologies (isecom), osstmm (hiện ở phiên bản 3) áp dụng phương pháp luận khoa học vào kiểm thử an toàn thông tin. <u>đặc trưng nổi bật của nó là ưu tiên metrics hơn opinions: </u>thay vì đưa ra đánh giá rủi ro chủ quan, <u>osstmm tạo ra kết quả có thể định lượng, kiểm chứng</u> và lặp lại được.
osstmm tổ chức kiểm thử xung quanh 5 kênh an ninh (security channels), thể hiện triết lý rằng an toàn thông tin không chỉ là vấn đề của mạng:
- human security (humsec): nhận thức và các lỗ hổng liên quan đến yếu tố con người.
- physical security (physsec): kiểm soát truy cập vật lý, từ thẻ từ đến hành vi tailgating.
- wireless communications (specsec): wi-fi, bluetooth, rfid và các tín hiệu điện từ khác.
- telecommunications (comsec): hệ thống điện thoại, voip, fax và hạ tầng modem.
- data networks (datasec): các dịch vụ mạng, firewall và giao thức tầng ứng dụng.

một tổ chức có thể có quy tắc cấu hình mạng hoàn hảo, nhưng nếu kẻ tấn công có thể bám đuôi (tailgate) vào phòng server (physsec) hoặc dùng social engineering để reset credential (humsec), thì các cơ chế kiểm soát mạng đó sẽ trở nên vô nghĩa. 5 kênh này đảm bảo không có khía cạnh nào bị bỏ sót.
<u>trọng tâm của phương pháp định lượng trong osstmm là risk assessment values</u> (ravs). ravs đo lường sự cân bằng giữa tổng attack surface (mức độ bộc lộ/exposure) và các cơ chế kiểm soát bảo vệ nó. chỉ số rav dương thể hiện rủi ro còn lại; rav tiệm cận 0 cho thấy các cơ chế kiểm soát phù hợp tốt với mức độ bộc lộ. đầu ra dạng số này giúp hai tester khi đánh giá cùng một mục tiêu sẽ đưa ra kết quả tương đồng, tương tự như hai kỹ sư cùng đo một dầm thép sẽ tính ra sức chịu tải tương đương.

**các giai đoạn thực thi**
chu kỳ kiểm thử của osstmm gồm 4 giai đoạn. hãy cùng đi qua từng giai đoạn qua kịch bản: đội của bạn đang đánh giá external network cho finvault corp, một công ty dịch vụ tài chính, với scope là dải ip 10.0.113.0/24 và customer portal tại portal.finvault-corp.thm.
- phase 1: induction (thu thập & xác minh)
    bao gồm enumeration và verification. bạn thu thập thông tin những gì đang tồn tại và xác nhận chúng có thực. tại finvault, bạn truy vấn dns, kiểm tra certificate transparency logs, và phát hiện các subdomain như vpn.finvault-corp.thm và mail.finvault-corp.thm. sau đó bạn xác minh từng asset đang hoạt động và phản hồi. đầu ra là danh mục asset đã được xác nhận của môi trường mục tiêu.
- phase 2: interaction (tương tác & định lượng)
    bao gồm qualification và quantification. bạn chủ động probe các asset đã xác minh và đánh giá mức độ liên quan. tại finvault, bạn kết nối tới từng dịch vụ, fingerprint công nghệ và định lượng exposure: 12 dịch vụ có thể truy cập từ bên ngoài trên 8 host, trong đó 4 dịch vụ chấp nhận kết nối không cần xác thực. các phát hiện này được đưa trực tiếp vào phép tính attack surface.
- phase 3: inquiry (khai thác & xác minh leo thang)
    bao gồm privilege escalation và verification escalation. bạn kiểm thử xem exposure đã đo lường có thể chuyển hóa thành truy cập trái phép hay không. tại finvault, bạn phát hiện một lỗ hổng trong customer portal cho phép một user đã xác thực có thể đọc sao kê tài khoản của customer khác. verification escalation xác nhận phạm vi ảnh hưởng: quyền đọc 12.000 tài khoản, không có quyền ghi.
- phase 4: intervention (can thiệp & kiểm soát)
    bao gồm quarantine, audit và enticement. bạn xử lý các phát hiện và đánh giá tổng thể môi trường kiểm soát. tại finvault, endpoint bị lỗi được hạn chế truy cập, đồng thời bản vá được phát triển (quarantine), mô hình access control rộng hơn được kiểm tra để tìm các lỗi tương tự (audit), và một canary token được triển khai để kiểm thử khả năng phát hiện nội bộ (enticement).

**tổng kết**
*osstmm* <u>quy định định dạng báo cáo security test audit report (star) cho các sản phẩm bàn giao, giúp đảm bảo tính nhất quán và khả năng so sánh giữa các team.</u>
<u>sự chặt chẽ về mặt khoa </u>học vừa là điểm mạnh lớn nhất vừa là rào cản chính của osstmm. osstmm giúp kết quả có thể audit và so sánh được, điều khá hiếm trong penetration testing. tuy nhiên, learning curve của nó rất dốc, việc triển khai đầy đủ mất nhiều thời gian, và các chuyên gia osstmm giàu kinh nghiệm khó tìm hơn so với những người được đào tạo theo các phương pháp luận khác. osstmm phù hợp nhất cho các tổ chức cần đo lường an ninh có tính lặp lại, có thể audit và sẵn sàng đầu tư để làm chủ phương pháp luận này.
