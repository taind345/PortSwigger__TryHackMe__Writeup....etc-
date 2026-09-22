# 1-Giới thiệu về IDOR
Các ứng dụng web thường dựa vào các định danh (identifiers) để phân biệt các đối tượng với nhau. Chẳng hạn như hồ sơ người dùng, hóa đơn, ticket hỗ trợ hay một tài liệu mật, mỗi thứ đều có một mã tham chiếu riêng (thường là một con số hoặc một chuỗi ký tự) để hệ thống nội bộ có thể định vị được chúng. Khi ứng dụng cho phép người dùng tự nhập mã tham chiếu này, rồi hồn nhiên truy xuất đối tượng đó ra trả về mà không thèm kiểm tra xem người dùng có quyền truy cập hay không, thì kết quả là chúng ta có một lỗ hổng **Insecure Direct Object Reference (IDOR)** (Tham chiếu đối tượng trực tiếp không an toàn).

IDOR được xếp vào nhóm lỗ hổng kiểm soát truy cập (access control). Nó chễm chệ ở vị trí số 1 trong danh sách OWASP Top 10 thuộc hạng mục *Broken Access Control* (Kiểm soát truy cập bị hỏng). Cái lỗi cốt lõi này cũng góp mặt trong bảng phong thần OWASP API Security Top 10 nhưng dưới một cái tên khác là **Broken Object Level Authorisation (BOLA)**. Tên gọi thì thiên biến vạn hóa tùy ngữ cảnh, nhưng nguyên nhân gốc rễ chỉ có một: server không chịu xác minh xem người dùng đang đăng nhập có thực sự được phép tương tác với đối tượng cụ thể mà họ vừa yêu cầu hay không.

Điều khiến IDOR trở nên đáng chú ý chính là sự chênh lệch giữa mức độ đơn giản khi khai thác và sức tàn phá của nó. Để khai thác lỗ hổng này, nhiều khi bạn chỉ việc đổi đúng một con số trên URL hoặc trong phần thân (body) của request là xong. Không cần kỹ thuật chèn mã (injection) phức tạp, chẳng cần cướp session, cũng không cần xài đến các công cụ chuyên dụng nào sất. Thế nhưng, hậu quả để lại có thể đi từ việc lộ lọt dữ liệu hàng loạt cho đến bị chiếm đoạt tài khoản hoàn toàn, tùy thuộc vào việc cái endpoint dính lỗi đó đang nắm giữ thông tin gì.

# 2- Ví dụ về IDOR
Dạng IDOR cơ bản và "ngây thơ" nhất xảy ra khi ứng dụng nhét thẳng một định danh đối tượng (object identifier) vào tham số URL, rồi vô tư dùng nó để chọc vào cơ sở dữ liệu phía sau (back-end database) mà chẳng thèm kiểm tra phân quyền (authorisation check) gì sất.

Giả sử khi đăng ký một dịch vụ trực tuyến và truy cập vào trang cá nhân. URL hiển thị như sau:

[http://online-service.thm/profile?user_id=1305]
Tham số `user_id` chỉ cho server biết cần phải móc bản ghi nào từ database ra. Trong trường hợp này, bản ghi 1305 tương ứng với tài khoản của mày, và trang web sẽ hiển thị tên, địa chỉ email, cùng các thông tin cá nhân khác đúng như kỳ vọng.

Giờ thử tưởng tượng mày táy máy đổi cái giá trị đó thành 1000 xem:

[http://online-service.thm/profile?user_id=1000]
Nếu trang web trả về hồ sơ của một người dùng hoàn toàn khác, bê nguyên cả thông tin cá nhân của họ ra, thì xin chúc mừng, con app đó đã dính lỗ hổng IDOR. Server chỉ đơn giản nhận cái tham số bị sửa, đi tìm bản ghi số 1000 rồi ném toẹt kết quả trả về. Nó chả thèm đoái hoài gì đến việc xác minh xem phiên đăng nhập (session) của mày có thực sự được phép xem bản ghi đó hay không.

Tại sao chuyện này lại xảy ra? Lớp xác thực (authentication) của ứng dụng vẫn đang hoạt động đúng. Nó biết mày là ai vì mày đã đăng nhập bằng một session hợp lệ. Thế nhưng, lớp phân quyền (authorisation) thì lại bốc hơi hoàn toàn. Chẳng có dòng code logic nào ở phía server đứng ra hỏi: "Cái session này có phải của user 1000 không?" hay "Thằng user này có quyền xem profile này không?". Con server cứ ngây thơ mặc định rằng: hễ ai đã đăng nhập (authenticated) là có đặc quyền truy cập vào bất cứ đối tượng nào họ yêu cầu.

Lỗ hổng này không chỉ giới hạn ở các thao tác đọc (read operations). Tùy thuộc vào cách endpoint được xây dựng, việc sửa đổi mã định danh có thể giúp kẻ tấn công cập nhật email của người dùng khác, reset mật khẩu, xóa sạch dữ liệu của họ, hoặc thực hiện các hành động cần đặc quyền khác. Chỉ một cú bỏ quên kiểm tra phân quyền trên một endpoint có chức năng ghi (write endpoint), sự cố rò rỉ dữ liệu (data disclosure) có thể lập tức biến thành thảm họa chiếm đoạt tài khoản toàn diện (full account takeover).

# 3-
Không phải con app nào cũng phơi bày các mã tham chiếu đối tượng (object references) dưới dạng văn bản rõ (plaintext) đâu. Các developer thường xuyên mã hóa (encode) mấy cái định danh này trước khi nhét chúng vào query string, POST data, hay cookie. Mã hóa (Encoding) biến dữ liệu thô thành một chuỗi an toàn theo chuẩn ASCII, chỉ dùng các ký tự a-z, A-Z, 0-9 và dấu `=` để chèn thêm (padding). Mục đích của việc này thực chất chỉ là để đảm bảo con server đích nhận và xử lý giá trị mượt mà không bị hiểu nhầm bởi mấy ký tự đặc biệt.

Kiểu mã hóa phổ biến nhất trên web là Base64. Các chuỗi Base64 thường dài hơn đáng kể so với giá trị thật của chúng và rất hay kết thúc bằng một hoặc hai dấu `=` ở đuôi. Ví dụ, số nguyên `123` khi encode Base64 sẽ biến thành `MTIz`. Hoặc một cục JSON nhỏ nhắn như `{"user_id": 5}` sẽ có hình hài kiểu `eyJ1c2VyX2lkIjogNX0=`.

Để khai thác một lỗ hổng IDOR bị ẩn sau lớp mã hóa, mày cứ ốp đúng 4 bước sau:

1. **Giải mã (Decode):** Bốc cái giá trị từ request ra và decode nó bằng mấy công cụ trực tuyến như `base64decode.org` hoặc xài lệnh terminal `echo 'value' | base64 -d`.
2. **Sửa đổi (Modify):** Xào xáo lại cái kết quả vừa giải mã để nó trỏ tới một đối tượng khác (ví dụ: đổi ID người dùng từ `5` thành `1`).
3. **Mã hóa lại (Re-encode):** Bọc cái giá trị vừa sửa bằng Base64 lần nữa qua trang `base64encode.org` hoặc dùng lệnh `echo 'value' | base64`.
4. **Tráo hàng (Substitute):** Nhét cái chuỗi mới encode đó ngược lại vào vị trí cũ trong request và gửi (submit) lên server.

Nếu server ngoan ngoãn trả về dữ liệu thuộc về một người dùng khác, thì bingo, cái endpoint đó dính lỗ hổng IDOR.

![[Pasted image 20260921112821.png]]

Một lầm tưởng cực kỳ phổ biến của giới dev là cho rằng "encoding" mang lại sự bảo mật. Không hề nha! Base64 chỉ là một phép biến đổi có thể dịch ngược (reversible transformation), chứ không phải là mã hóa bảo mật (encryption). Bất cứ ai nắm được chuỗi encode này đều có thể tự giải mã nó dễ dàng. Một ứng dụng dựa dẫm vào các định danh được encode mà lại "quên" kiểm tra phân quyền (authorisation checks) ở phía server thì cũng mỏng manh dễ vỡ y hệt như xài ID dạng plaintext vậy.
# 4-
lỗi thực ra ko nằm ở cái mã hóa mà nằm ở phân quyền , phân quyền mới là vấn đề cốt lõi, cho dù đặt id=1,2,3 , nhưng mà phân quyền chặt thì cũng chẳng thể nào vào được
