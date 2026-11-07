-đọc lý thuyết tổng hợp ở đây -> [[0-Lý thuyet JWT]] (cấu trúc 3 phần, luồng verify, các đòn, tool JWT Editor/hashcat)
-tool bắt buộc: Burp extension **JWT Editor Keys**, wordlist rockyou cho hashcat.

 -ok trước tiên cần phải học cách sử dụng jwt editgor trong burp site https://portswigger.net/burp/documentation/desktop/testing-workflow/vulnerabilities/session-management/jwts
-hoặc đơn giản hơn lên trang này https://www.jwt.io/
# LAB (từ dễ tới khó)
đầu tiên dễ nhất [[Lab-JWT authentication bypass via flawed signature verification]]

-tiếp
[[Lab-JWT authentication bypass via unverified signature]] -> bypass thông qua việc sever nó ko check cái signature 

-mở bát bằng bài secret yếu -> [[Lab-JWT authentication bypass via weak signing key]] => `hashcat -m 16500` brute secret rồi tự ký token `administrator`.


-tiếp là bài nhầm lẫn thuật toán -> [[Lab-JWT algorithm confusion]] => server RS256 nhưng mình đổi `alg` thành HS256 rồi ký bằng public key lấy ở `/jwks.json`.--> bài này nó cứ bị đao đao cái phần copy key trên burp nên làm mãi ko ra

-tiếp là bài nhúng key -> [[Lab-JWT authentication bypass via jwk header injection]] => tự tạo cặp RSA, nhét public key mình vào header `jwk`, ký bằng private key mình.

-[[Lab-JWT authentication bypass via jku header injection]]

-tiếp là bài kid path traversal -> [[Lab-JWT kid path traversal]] => `kid` là đường dẫn file nên mình trỏ về `/dev/null` (secret rỗng) rồi ký rỗng.

-cuối là bài kid SQLi -> [[Lab-JWT kid SQL injection]] => `kid` ghép vào SQL nên mình UNION SELECT ra secret do mình chọn rồi ký bằng nó. Bài này khoai nhất topic.
