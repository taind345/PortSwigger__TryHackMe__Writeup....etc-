Muốn quét sạch mọi ngóc ngách để lôi ra các endpoint ẩn, mày kết hợp 5 tầng quét từ đục đường dẫn, cào file JS, soi tài liệu API cho đến dò tham số:

1. Đục thư mục và file ẩn (Directory / File Fuzzing)
Dùng ffuf kết hợp wordlist xịn trong SecLists để quét cả thư mục lẫn đuôi file mở rộng:
`ffuf -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-directories.txt -u http://TARGET/FUZZ -e .php,.json,.bak,.api,.config -mc 200,301,302,403`
Nếu gặp mã 403 Forbidden, đừng bỏ qua vì đó chính là các endpoint nội bộ có tồn tại nhưng đang chặn truy cập ngoài.
2. Cào web tự động và bới lịch sử (Crawling & Passive Recon)
Dùng crawler để nó tự bấm vào mọi link, form trên web và dùng kho lưu trữ Internet bới lại các URL cũ:

* Dùng Katana để spider toàn bộ web (cào cả code JS): `katana -u http://TARGET -jc -d 3`
* Dùng Gau hoặc Waybackurls bới lại các endpoint từng xuất hiện trong quá khứ: `gau TARGET | sort -u > endpoints.txt`

3. Mổ xẻ file JavaScript (JS Scraping)
Mấy web viết bằng React, Vue hay Angular thường nhét cứng (hardcode) cả đống endpoint API nội bộ bên trong các file .js.

* Tìm hết file .js đang tải trên trang.
* Dùng tool LinkFinder hoặc katana trích xuất tự động các đường dẫn dạng `/api/v1/...` giấu trong file script.

4. Săn tài liệu API bị bỏ quên (Swagger / GraphQL)
Dev rất hay để quên tài liệu API ở chế độ công khai:

* Fuzz nhanh các đường dẫn quen thuộc: `/swagger.json`, `/api-docs`, `/swagger-ui.html`, `/openapi.json`, `/v1/api-docs`.
* Nếu thấy cổng `/graphql`, bắn query Introspection để server tự nôn ra toàn bộ danh sách truy vấn và schema ẩn.

5. Dò tham số ẩn trên endpoint (Parameter Fuzzing)
Khi tìm ra một endpoint (ví dụ `/api/user`) nhưng không biết nó nhận tham số gì, dùng tool Arjun để tự động đoán:
`arjun -u http://TARGET/api/user -m GET,POST`
Tool sẽ tự tìm xem endpoint đó có nhận các biến ngầm như `?debug=true`, `?admin=1`, `?role=...` hay không.