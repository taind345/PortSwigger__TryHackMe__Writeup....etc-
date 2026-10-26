![[Pasted image 20261023124602.png]]

-cái này nó cũng dùng các phương pháp encoding để escape html

![[Pasted image 20261023125810.png]]
=>tes trước với `${7*7}` => ta được kết quả 49 
=> khả năng cáo input -> server -> được xử lý như template 
payload :
```
${alert(1)}
```
![[Pasted image 20261023125236.png]]
![[Pasted image 20261023124838.png]]
![[Pasted image 20261023124704.png]]
=> các biểu thức nằm trong ${} sẽ được js xử lý và trả về kết quả
=>