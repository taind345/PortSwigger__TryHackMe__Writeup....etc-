-đọc phần lý thuyết về authentication ở đây
--> [[lys thuyet authentication]] --> phần này mình học được rất nhiều thứ, từ jwt, các kỹ thuật bypass jwt và authentication. Mình đọc kỹ phần này trước để hiểu tại sao server lại lộ username qua response.

-tiếp đó mình mới đi làm lab, thứ tự mình đi như này cho dễ hiểu từ dễ tới khó:

# LAB - Password based
-đầu tiên mở bát bằng bài dễ nhất -> [[Lab-Username enumeration via different responses]] => bài này mình chỉ cần nhìn câu báo lỗi là biết username nào đúng, sau đó mình brute password thôi.

-tiếp theo là bài khó hơn một chút, response nó chỉ khác nhau 1 ký tự tàng hình -> [[Lab-Username enumeration via subtly different responses]] => trước mình tưởng lab này bị lỗi, không có tổ hợp nào pass, kể cả pass mẫu cũng không được. Sau khi quan sát kỹ gói tin trả về, mình mới thấy là nó khác nhau ở dấu `.` cuối câu, nên mình phải soi raw response chứ không nhìn mắt thường được.

-tiếp theo là đi tới 1 cái lab tận dụng lỗi "user enumeration" qua time response => [[Lab-Username enumeration via response timing]] => phần này mình học thêm được cách mà bypass ip block, hiểu được tại sao khi login đúng thì response nó bị delay, và hiểu tại sao lại cần payload với password dài.

-ok, next sang lab tiếp theo
[[Lab-Broken brute-force protection, IP block]] => bài này mình thấy server nó block theo IP qua `X-Forwarded-For`, nên mình chỉ cần đổi IP liên tục là bypass được. Mình học được cách dùng ffuf với 2 wordlist pitchfork: 1 cái password, 1 cái IP.

-tiếp là bài lock account -> [[Lab-Username enumeration via account lock]] => sau khi quan sát gói tin này, mình thấy nếu user đúng mà nhập sai 5 lần thì nó báo `You have made too many incorrect login attempts`, còn user sai thì nó vẫn báo `Invalid username or password`. Nên mình dùng chính cái lock này để enumerate username.

-cuối phần password-based là bài nhiều credential trong 1 request -> [[Lab-Broken brute-force protection, multiple credentials per request]] => mình thấy body nó là JSON mà trường `password` lại là array, nên mình nhét cả list password vào 1 request là bypass được rate-limit.

# LAB - 2FA
-mở bát 2FA bằng bài bypass đơn giản nhất -> [[Lab-2FA simple bypass]] => sau khi login đúng user/pass, mình thấy nó redirect sang `/login2`, nhưng mình thử vào thẳng `/my-account` thì nó cho vào luôn, không thèm check đã nhập 2FA chưa. Nên mình bypass được.

-tiếp là bài brute 2FA 4 số -> [[Lab-2FA bypass using a brute-force attack]] => mình thấy mã 2FA chỉ có 4 số, mà server lại không block hẳn, nó chỉ logout sau vài lần sai. Nên mình dùng macro để auto login lại rồi brute tiếp.

# LAB - Other mechanisms
-bài cookie remember-me -> [[Lab-Brute-forcing a stay-logged-in cookie]] => sau khi quan sát cookie này, mình thấy nó là `base64(username:md5(password))`, nên mình brute md5 của carlos là ra password.

-bài reset password lỗi logic -> [[Lab-Password reset broken logic]] => mình thấy khi submit form reset, server không thèm check token có khớp với user không, nên mình đổi `username=carlos` là reset được password của carlos.

-bài brute qua chức năng đổi password -> [[Lab-Password brute-force via password change]] => chỗ `/my-account/change-password` nó không có rate-limit, lại còn báo khác nhau khi user đúng/sai, nên mình brute ở đó luôn cho khỏi bị block ở `/login`.

-bài lộ thông tin để bypass -> [[Lab-Authentication bypass via information disclosure]] => sau khi quan sát gói tin này, mình thấy dùng method `TRACE` thì server nó lộ header `X-Custom-IP-Authorization`. Mình thêm header đó với giá trị `127.0.0.1` là vào được admin.

-bài encryption oracle -> [[Lab-Authentication bypass via encryption oracle]] => bài này khoai nhất, mình thấy chức năng comment nó mã hóa email bằng cùng key với cookie `stay-logged-in`, nên mình dùng nó như 1 cái máy mã hóa để tự tạo cookie admin.

-bài đầu độc host để cướp token reset -> [[Lab-Password reset poisoning via X-Forwarded-Host]] => sau khi quan sát gói tin này, mình thấy server lấy host từ `X-Forwarded-Host` để tạo link reset, nên mình đổi host thành exploit-server là cướp được token của carlos.

> [!NOTE]
> -wordlist username/password mẫu của portswigger mình lấy ở đây:
> -https://portswigger.net/web-security/authentication/auth-lab-usernames
> -https://portswigger.net/web-security/authentication/auth-lab-passwords
> -tool mình dùng là ffuf với file `login.txt`, vì mình quen ffuf hơn Burp Intruder.
