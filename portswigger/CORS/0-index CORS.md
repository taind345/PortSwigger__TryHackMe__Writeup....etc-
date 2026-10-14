-đọc lý thuyết tổng hợp ở đây -> [[lys thuyet CORS]] (SOP, cách CORS nới lỏng, các lỗi hay gặp)
-chi tiết vẽ tay: [[0-map CORS]], [[vai trò của CORS và tác động tới SOP]], [[vấn đề của CORS với bảo mật]], [[whitelist CORS]], [[bai1- cors]]

# LAB
-mở bát bằng bài dễ nhất -> [[Lab-CORS basic origin reflection]] => server echo lại đúng Origin mình gửi, mình dùng XHR + withCredentials để đọc API key của admin.

-tiếp là bài trust null -> [[Lab-CORS null origin]] => whitelist có `null`, mình dùng iframe `sandbox` để tạo origin null rồi đọc ké.

-tiếp là bài http lẫn https -> [[Lab-CORS insecure protocols]] => site cho `http://sub.site` trong khi site chính https, mình chiếm subdomain http rồi đọc dữ liệu.

-cuối là bài pivot mạng nội bộ -> [[Lab-CORS internal pivot]] => dùng XSS ở subdomain public để bắt browser victim đánh vào server nội bộ rồi đọc response qua CORS.
