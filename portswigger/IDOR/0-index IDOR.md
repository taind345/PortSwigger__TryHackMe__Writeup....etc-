-đọc lý thuyết tổng hợp ở đây -> [[lys thuyet IDOR]] (các dạng IDOR, cách test 2 account)
-chi tiết dài: [[IDOR]]

# LAB
-mở bát bằng bài admin lộ -> [[Lab-Unprotected admin functionality]] => đọc `/robots.txt` thấy `/administrator-panel`, vào xóa carlos.

-tiếp là bài role trong cookie -> [[Lab-User role controlled by request parameter]] => cookie `Admin=false`, sửa `true` là lên admin.

-tiếp là bài id trong URL -> [[Lab-User ID controlled by request parameter]] => `/my-account?id=wiener` đổi thành `?id=carlos` là xem + leak API key của carlos.

-tiếp là bài check method ẩu -> [[Lab-Method-based access control]] => GET bị chặn nhưng POST thì không, mình đổi method để upgrade role.

-cuối là bài IDOR đọc chat -> [[Lab-Insecure direct object references]] => `/download-transcript` đổi `key` là đọc được chat của carlos lấy password.

-tiếp là bài check Referer -> [[Lab-Referer-based access control]] => thêm `Referer: .../admin` là vào được admin, vì server tin header client gửi.

-cuối là bài multi-step -> [[Lab-Multi-step no access control]] => bước 2 upgrade role không check quyền, mình gọi thẳng bước 2 là thành admin.
