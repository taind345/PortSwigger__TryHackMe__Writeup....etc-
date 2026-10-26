![[Pasted image 20261021180806.png]]

### lý thuyết 
![[Pasted image 20261021181050.png]]


![[Pasted image 20261021181338.png]]


root cause
![[Pasted image 20261023104523.png]]
![[Pasted image 20261023104634.png]]

=>input lưu vào biến searchterm và được sigle quote 
payload
```
\'-alert(1)//
```
![[Pasted image 20261023105046.png]]
=> nhìn vào ảnh thì alert(1) đã thoát khỏi dấu nháy ' và nằm trong ther script html của trinhf duyệt. Do đó nó popup thông báo 