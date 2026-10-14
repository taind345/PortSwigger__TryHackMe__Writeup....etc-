-đọc lý thuyết tổng hợp ở đây -> [[lys thuyet JWT]] (cấu trúc 3 phần, luồng verify, các đòn, tool JWT Editor/hashcat)
-tool bắt buộc: Burp extension **JWT Editor Keys**, wordlist rockyou cho hashcat.

# LAB (từ dễ tới khó)
-mở bát bằng bài secret yếu -> [[Lab-JWT weak signing key]] => `hashcat -m 16500` brute secret rồi tự ký token `administrator`.

-tiếp là bài bỏ signature -> [[Lab-JWT flawed signature verification]] => sửa `alg` thành `none`, xóa signature, server ẩu vẫn cho vào admin.

-tiếp là bài nhầm lẫn thuật toán -> [[Lab-JWT algorithm confusion]] => server RS256 nhưng mình đổi `alg` thành HS256 rồi ký bằng public key lấy ở `/jwks.json`.

-tiếp là bài nhúng key -> [[Lab-JWT jwk injection]] => tự tạo cặp RSA, nhét public key mình vào header `jwk`, ký bằng private key mình.

-tiếp là bài kid path traversal -> [[Lab-JWT kid path traversal]] => `kid` là đường dẫn file nên mình trỏ về `/dev/null` (secret rỗng) rồi ký rỗng.

-cuối là bài kid SQLi -> [[Lab-JWT kid SQL injection]] => `kid` ghép vào SQL nên mình UNION SELECT ra secret do mình chọn rồi ký bằng nó. Bài này khoai nhất topic.
