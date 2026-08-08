This lab contains a DOM-based cross-site scripting vulnerability in the submit feedback page. It uses the jQuery library's `$` selector function to find an anchor element, and changes its `href` attribute using data from `location.search`.

To solve this lab, make the "back" link alert `document.cookie`.

-form này có input
![[Pasted image 20260725110128.png]]

-nhìn vào mã js của trang feedback này, ta có thể thấy location.search được đưa vào cái href, cái này cho phép thực thi mã js , đọc thêm tại [[DOM based Xss#3- Sources and sinks in third-party dependencies]]

![[Pasted image 20260725110734.png]]
"backlink" ở đây là cái nút back ý, khi mình click thid đoạn cript nó được thực thi 

-bây giơ ta cânf chèn input là : javascript:alert() là xong
![[Pasted image 20260725105925.png]]

-khi ta nhấn vào nút back thì mã script được thực thi 
![[Pasted image 20260725111222.png|494]]
