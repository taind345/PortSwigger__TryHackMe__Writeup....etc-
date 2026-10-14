-đọc lý thuyết tổng hợp ở đây -> [[lys thuyet OAuth]] (luồng authorize->code->token, các chỗ chết)
-chuẩn bị: 1 account social/lab + exploit-server để hứng code/token.

# LAB (từ dễ tới khó)
-mở bát bằng bài tin email -> [[Lab-OAuth implicit flow]] => POST `/authenticate` sửa `email` thành `carlos@carlos-montoya.net` là vào luôn account carlos.

-tiếp là bài thiếu state -> [[Lab-Forced OAuth linking]] => lấy code của mình, dụ admin click link callback để account admin bị link vào social mình.

-tiếp là bài redirect_uri -> [[Lab-OAuth redirect_uri hijack]] => sửa `redirect_uri` thành domain mình để cướp code của admin.

-tiếp là bài open redirect -> [[Lab-OAuth open redirect steal]] => kết hợp redirect_uri + open redirect ở client để access token bay về server mình.

-cuối là bài đăng ký client SSRF -> [[Lab-OpenID dynamic registration SSRF]] => `logo_uri` nhận URL bừa, server tự fetch vào mạng nội bộ. Bài này khoai nhất topic.
