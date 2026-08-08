![[blind ssrf]]
[[blind ssrf]]

#ssrf 
# 🕶️<mark style="background:#40a9ff"> Blind SSRF Vulnerabilities </mark>

## Blind SSRF là gì?
- Ứng dụng **gửi back-end HTTP request đến URL do attacker cung cấp**, nhưng **response từ back-end không được trả về** trong front-end response.
- Attacker không trực tiếp thấy dữ liệu từ server nội bộ, do đó gọi là "mù" (blind).

## Tác động
- ==Thường **thấp hơn** SSRF thông thường==<mark style="background:#d3f8b6"> vì tính một chiều</mark>, không dễ đọc dữ liệu.
- Tuy nhiên trong một số trường hợp, vẫn có thể đạt **RCE (Remote Code Execution)** <mark style="background:#d3f8b6">nếu khai thác được lỗ hổng phía server hoặc client HTTP.</mark>

## Cách phát hiện (sử dụng Out-of-band – OAST)
- **Nguyên lý**: Kích hoạt request đến một hệ thống bên ngoài mà bạn kiểm soát, giám sát tương tác mạng.
- **Công cụ phổ biến**: **Burp Collaborator**.
  - Tạo domain duy nhất, chèn vào payload.
  - Nếu ứng dụng gửi request đến domain đó → xác nhận Blind SSRF.
- **Lưu ý quan trọng**:
  - **Chỉ thấy DNS lookup** (không có HTTP request) → ứng dụng đã cố gắng kết nối nhưng bị chặn ở tầng mạng (outbound DNS được phép, HTTP bị chặn). Vẫn coi là dấu hiệu Blind SSRF.
  - **Cả DNS và HTTP request** → khả năng khai thác cao hơn.

## Cách khai thác
- **Blind SSRF không cho phép xem response** → không dùng để đọc dữ liệu trực tiếp.
- Nhưng vẫn có thể:
  1. **Quét dải IP nội bộ (blind sweeping)**:
     - Gửi payload tới các IP/port nội bộ (ví dụ `http://192.168.0.1:22/`).
     - Kết hợp kỹ thuật out-of-band để phát hiện service (dựa vào thời gian phản hồi, DNS callback, lỗi kết nối).
  2. **Tấn công các lỗ hổng nội bộ đã biết**:
     - Gửi payload khai thác (RCE, SQLi,…) qua request SSRF.
     - Dùng out-of-band để nhận callback khi khai thác thành công (ví dụ: dùng `nslookup your-domain` qua RCE).
  3. **Ép server kết nối tới hệ thống của attacker và trả về malicious response**:
     - Nếu server dùng thư viện HTTP client có lỗ hổng (ví dụ buffer overflow, deserialization), attacker có thể khai thác client-side RCE.
     - Attacker kiểm soát hoàn toàn HTTP response gửi về server → chèn shellcode, payload.

## Ghi nhớ nhanh
| Phương diện | Điểm chính |
|-------------|------------|
| **Bản chất** | Không thấy response, nhưng vẫn có tương tác 2 chiều (qua DNS, thời gian). |
| **Phát hiện** | Burp Collaborator – theo dõi DNS/HTTP callback. |
| **Khai thác** | Quét mù internal IP, tấn công lỗ hổng nội bộ qua blind RCE, client-side exploit. |
| **Rủi ro** | Có thể leo thang lên full RCE nếu tìm được điểm yếu trong nội bộ. |

✍️ **Tóm lược**: Blind SSRF giống như bắn tên trong bóng tối – bạn không thấy đích, nhưng nếu biết hướng và có phản hồi ngoài luồng (OAST), bạn vẫn có thể bắn trúng mục tiêu và gây sát thương lớn.




# 🎯 <mark style="background:#40a9ff">Kịch bản tấn công khi server request đến domain do attacker kiểm soát</mark>

Khi ứng dụng (server) kết nối ra ngoài tới một tên miền bạn dựng lên, bạn **kiểm soát hoàn toàn dữ liệu trả về** và có thể chủ động mở rộng tấn công theo các hướng sau.

## 1. Thu thập thông tin nhạy cảm
- **Header & Body**: Ghi nhận tất cả thông tin server gửi tới (IP nội bộ, User-Agent, token, cookie, Basic Auth, API key…).
- **Client Certificate**: Nếu server dùng chứng chỉ client, bạn có thể capture được thông tin chứng chỉ hoặc chính private key (nếu sơ hở).
- **Phát hiện công nghệ**: ==Xác định HTTP client library, phiên bản== (qua header, quirks) ==để tìm lỗ hổng phù hợp.==

## 2. Phản hồi độc hại để khai thác chính server (Client-Side Exploit)
- ==**Lỗi trong thư viện HTTP client**==: Gây tràn bộ đệm, deserialization không an toàn (ví dụ Java deserialization, Python pickle), XXE (nếu xử lý XML/Soap).
- **Xử lý nội dung đặc biệt**: Trả về ảnh, PDF, font… chứa payload để khai thác lỗi parser (ImageMagick, Ghostscript, PDFium…).
- **HTTP Response Splitting / CRLF Injection**: Chèn header độc hại để đầu độc cache, đánh lừa proxy trung gian.

## 3. Chuyển hướng (Redirect) đến mục tiêu nội bộ
- **Bypass whitelist**: Server follow redirect (301/302/307) tới internal IP (`192.168.x.x`, `127.0.0.1`) mà filter không kiểm tra đích đến của redirect.
- **Quét cổng mù (blind port scan)**: <mark style="background:#d3f8b6">Redirect đến các IP:port nội bộ,</mark> dựa vào thời gian phản hồi để suy đoán cổng mở/đóng.
- **Tấn công chéo giao thức**: Redirect sang scheme như `gopher://`, `dict://`, `ftp://`, `file://` để gửi lệnh đến Redis, Memcached, SMTP, Zabbix… mà không cần tương tác trực tiếp từ attacker.

## 4. DNS Rebinding
- **Cơ chế**: Domain của attacker có TTL thấp. Lần phân giải đầu trỏ ra IP public (vượt filter), lần phân giải sau trỏ về IP nội bộ (127.0.0.1, 192.168.0.1…).
- **Khai thác**: Khi server tự động retry hoặc follow redirect, nó sẽ kết nối đến IP nội bộ, truy cập dịch vụ cấm.

## 5. Tấn công vào các dịch vụ nội bộ (Internal Pivoting)
- **Đọc dữ liệu nội bộ**: Nếu server có thể bị ép request đến các API nội bộ (REST, SOAP) và bạn có thể lấy kết quả (qua OAST hoặc timing), chiếm dữ liệu nhạy cảm.
- ==**RCE gián tiếp**==: Dùng<mark style="background:#d3f8b6"> redirect hoặc response để gửi payload khai thác CVE </mark>trên các service nội bộ không cần xác thực (ví dụ Jenkins, Kibana, Elasticsearch, Tomcat…).
- **Cloud metadata**: Trên cloud (AWS, GCP, Azure), nếu SSRF tới được metadata endpoint (`169.254.169.254`), bạn có thể đánh cắp token, key, sau đó dùng domain của mình làm nơi nhận dữ liệu (qua DNS/HTTP request).

## 6. Gây rối loạn dịch vụ (DoS)
- **Slow Response**: <mark style="background:#d3f8b6">Giữ kết nối mở,</mark> gửi dữ liệu rất chậm (Slowloris style) làm cạn kiệt connection pool của server.
- **Phản hồi cực lớn**: Trả về body khổng lồ khiến server hết bộ nhớ.
- **Lặp vô hạn**: Tạo vòng lặp redirect giữa các domain của bạn → tiêu tốn tài nguyên.

## 7. Lợi dụng server làm "==proxy==" để tấn công bên thứ ba
- **Amplification**: Ép server gửi request lớn đến mục tiêu khác (tấn công DDoS phản xạ).
- **Webhook tùy chỉnh**: Đặt server trung gian để thực hiện request đến bất kỳ đâu, che giấu nguồn gốc tấn công.

## 8. Kết hợp với lỗ hổng khác trên ứng dụng
- **Blind XSS/HTML injection**: Nếu phản hồi từ attacker được hiển thị ở đâu đó (dù không thấy trực tiếp), có thể chèn script.
- **Cache poisoning**: Trả về header cache đặc biệt (Age, Cache-Control) để đầu độc cache của proxy hoặc CDN.

---

💡 **Mấu chốt**: Bạn không chỉ quan sát request, mà còn **chủ động trả về bất kỳ dữ liệu nào**, mở ra vô số hướng leo thang từ đọc thông tin đến chiếm toàn bộ hệ thống nội bộ.
