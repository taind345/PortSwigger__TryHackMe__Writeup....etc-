Tiếp theo, chúng ta sẽ đi sâu vào các loại XSS, mở bát bằng **Reflected XSS (XSS Phản xạ)**.

Reflected XSS xảy ra khi một ứng dụng web nhận dữ liệu đầu vào của người dùng (như chuỗi tham số trên URL, trường nhập liệu, hoặc header) và in ngay lập tức ra màn hình mà không thèm dọn dẹp (sanitising) nó. Kẻ tấn công sẽ tạo một đường link hoặc biểu mẫu độc hại chứa mã JavaScript rồi lừa nạn nhân click vào; vì trang web "phản xạ" (echo) lại đầu vào của kẻ tấn công, trình duyệt sẽ ngoan ngoãn chạy đoạn script bị chèn ngay trên trang đó. Các vị trí dễ dính đòn nhất là ô tìm kiếm, thông báo lỗi, hoặc bất kỳ trang nào bốc tham số từ URL hoặc POST body ra để in thẳng lên giao diện.

Khi script độc hại chạy trên trình duyệt của nạn nhân, nó có thể đọc hoặc sửa đổi trang, thó luôn cookie hay session token (trừ khi cookie đó được bảo vệ bằng cờ `HttpOnly`), mạo danh người dùng làm trò mờ ám, hoặc tải thêm rác rưởi độc hại khác. Nguyên nhân gốc rễ là do developer xuất dữ liệu không đáng tin cậy dưới dạng mã lệnh (code) thay vì mã hóa (escaping) nó cho an toàn.

### Thực hành

Để dễ hình dung phần thực hành, mày xem thử đoạn code `app.py` (Atlas News) này:
**==> đây là backend của bài lab**
```python
from flask import Flask, request, render_template, redirect
from markupsafe import escape
from datetime import datetime

app = Flask(__name__)

# helper for template footer (year)
@app.context_processor
def inject_now():
    return {"now": lambda: datetime.utcnow().year}

NEWS = [
    {"title": "Product launch: SecureMail", "summary": "A privacy-focused email client arrives."},
    {"title": "Weekly Roundup", "summary": "Top vulnerabilities and patches this week."},
    {"title": "Research: XSS Trends 2025", "summary": "A short summary of XSS cases observed in the wild."},
]

COMMENTS = []

@app.route("/")
def home():
    q = request.args.get("q", "")
    query_escaped = escape(q) 
    return render_template("news.html", news=NEWS, query=q, query_escaped=query_escaped)

@app.route("/guestbook", methods=["GET", "POST"])
def guestbook():
    if request.method == "POST":
        name = request.form.get("name", "Anonymous")
        comment = request.form.get("comment", "")
        
        COMMENTS.append({"name": escape(name), "comment": comment})
        return redirect("/guestbook")

    return render_template("guestbook.html", comments=COMMENTS)

@app.route("/dom")
def dom_preview():
    return render_template("dom_preview.html")

if __name__ == "__main__":
    # bind to all interfaces for lab convenience (change to 127.0.0.1 for local-only)
    app.run(debug=True, host="0.0.0.0", port=5000)

```

Mở Mozilla Firefox trên AttackBox và truy cập vào `http://MACHINE_IP:5000` để vào trang web (Nếu mày đang cắm VPN thì mở trình duyệt máy mày lên và vào link tương tự). Mày sẽ thấy trang web AtlasNews với một ô tìm kiếm nằm ngay trên cùng.
![[Pasted image 20260919110008.png]]
Trong ô tìm kiếm, gõ thử một từ bình thường như "product" rồi bấm Search để xem kết quả hiển thị ra sao.

Bây giờ, với tư duy của một pentester, mình sẽ đào sâu hơn bằng cách dán một đoạn code dị dị như `<script>alert('Hack')</script>`. Nếu trang web này dính lỗ hổng, mày sẽ thấy một hộp thoại pop-up của trình duyệt nhảy xổ ra, và nội dung bị chèn hiển thị chễm chệ ngay bên trong khu vực kết quả tìm kiếm.

Đó chính là Reflected XSS; con app đọc tham số truy vấn `q` từ URL và lập tức kết xuất (render) ngược lại trang. Hậu quả là cái giá trị URL do kẻ tấn công nhào nặn được thực thi thẳng trên trình duyệt của nạn nhân.

### Nguyên nhân gốc rễ (Root Cause)

Nhìn vào file `app.py` của ứng dụng web, hàm xử lý đã bê nguyên cục tham số thô ném thẳng vào template:

```python
return render_template("news.html", news=NEWS, query=q, query_escaped=query_escaped)

```

Trong đó, cái biến `q` được lấy trực tiếp từ lệnh `q = request.args.get("q", "")`, không hề đi qua bộ lọc nào. Nó chính là khởi nguồn của dữ liệu độc hại (untrusted data).


