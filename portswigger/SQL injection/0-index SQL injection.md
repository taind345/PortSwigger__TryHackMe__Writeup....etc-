sau khi học phần này, mình hiểu là...
SQLi phải đi từ dễ tới khó.
-> dễ là thấy data luôn.
-> khó là blind.
=> mình xếp thứ tự như dưới.

lý thuyết nền đọc trước:
- [[lys thuyet sql injection]]
- [[các kỹ thuật chèn sql]]
- [[SQL cheet sheet]]
- [[THM-0-sql injection]]
- [[nền tẩng sql injection]]

thứ tự lab mình đề xuất:
- mở bát bằng bài này -> [[Lab-SQLi retrieval of hidden data]] => mình học được `OR 1=1` lụm hàng hidden.
- tiếp theo quất login -> [[Lab-SQLi login bypass]] => mình học được `administrator'--` bypass password.
- xong tới đếm cột -> [[Lab-SQLi UNION determine columns]] => mình học được ORDER BY + UNION NULL.
- rồi đọc version + users -> [[Lab-SQLi UNION retrieve version and users]] => mình học được đọc @@version / version() rồi dump users.
- đọc version trên Oracle -> [[Lab-SQLi querying database version Oracle]] => mình học được `v$version.banner` + nhớ mọi SELECT phải có `FROM`.
- qua bản Oracle list bảng/cột -> [[Lab-SQLi listing database contents Oracle]] => mình học được `all_tables` / `all_tab_columns` + `FROM dual` để tự tìm bảng/cột rồi dump users.
- qua blind boolean -> [[Lab-Blind boolean conditional responses]] => mình học được TrackingId + SUBSTRING brute.
- qua blind time -> [[Lab-Blind time delays]] => mình học được pg_sleep conditional exfil.
- qua OAST cho gọn -> [[Lab-Blind OAST]] => mình học được DNS lookup ra Collaborator.
- cuối cùng là bypass filter -> [[Lab-SQLi filter bypass]] => mình học được encode `SELECT` / `UNION` để qua WAF.

file bổ trợ khi kẹt:
- [[union sql  injection]]
- [[xác định số cột&kiểu dl]]
- [[các bước thu thập thông tin về csdl]]
- [[tấn công mật khẩu]]
- [[Blind sql injection]]
- [[bai1- based errorr sql injection]]
- [[bai2-error based injection]]
- [[based error sql injection]]
- [[payload giúp brute force mật khẩu cho blind && error-sql]]
- [[Minh_hoa_prepare_sql]]

> [!NOTE] bài học
> - đừng nhảy vô blind khi chưa chắc UNION.
> - mỗi lab 1 kỹ thuật, xong lab nào ghi lab đó.

# LAB nâng cao
- [[Lab-Blind OAST exfiltration]] -> boolean/time đều chết thì dùng DNS exfil về collaborator, mỗi ký tự password 1 hit.
