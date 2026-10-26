![[Pasted image 20261023110215.png]]

-phân tích những gì bài lab này có:
![[Pasted image 20261023110436.png]]
-> các cái encode trên ngăn chặn việc mình escape html
![[Pasted image 20261023110916.png]]
-> test thử có vẻ source nằm ở phần website

-> payload:
```

```

![[Pasted image 20261023111148.png]]

-> payload đi vào trong onclick -> có thể tra thêm trong [[0-xss cheetsheet]] .
![[Pasted image 20261023112306.png]]
![[Pasted image 20261023111631.png]]
=> cái payload trên nó sẽ dùng `&apos` để có thể vượt qua lớp encoding phía server, nhưng khi nó đi về phía client, trình duyệt sẽ giải mã nó lại thành dấu nháy '. => bypass được encode phía server và escape được html

![[Pasted image 20261023112023.png]]