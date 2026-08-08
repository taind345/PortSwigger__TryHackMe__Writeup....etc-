 This lab contains a DOM-based cross-site scripting vulnerability on the home page. It uses jQuery's $() selector function to auto-scroll to a given post, whose title is passed via the location.hash property.

To solve the lab, deliver an exploit to the victim that calls the print() function in their browser. 

> -ok bài này mình xem giải, qua những gì mình xem *mình học được là:*
>    -  dùng hàm *hashchange*, hashchange được kích hoạt như nào
>    - dùng f12 -> vào *source* để xem script ( ctr shift F để tìm `<script>')
>    - hiểu hơn về cái hash # được truyền qua url
>    - *onload* và *onerror* trong thẻ image ?

## write up
### các bước tìm ra lỗ hổng
-ban đầu tìm đoạn script trên trang web
![[Pasted image 20260728135533.png]]
-
-nhìn vào đoạn script này, ta thấy web sẽ lấy hash từ url; sau đó với *hashchange*,thì khi nào có thay đổi hash trong url thì sự kiện sẽ được kích hoạt
- sự kiện là: lấy hash từ url--> dùng hash đó scoll đến bài viết.Hash ở đây là tên bài đăng 
![[Pasted image 20260728135617.png]]

- ta sẽ test thử .Khi ta thêm hash của title bài đăng vào, ta sẽ thấy nó scoll đến bài đăng(sự kiện này chỉ xảy ra khi có *hashchange* tức là có sự thay đổi hash ở url)
![[Pasted image 20260728140900.png]]
*-->* vấn đề là cái hàm hash change này trong jquery nó có lỗ hổng bảo mật , cho phép chèn html vao element .ở đây thẻ html là $(winodow) sẽ bị khai thác bởi DOM là scrollIntoView() - tức là cái cửa sổ hiển thị sẽ scoll tới cái html truyền vào
*-->* nếu ta chèn vào 1 hash là `<img src=x onerrorr= sự kiện>` thì cái *window* sẽ scroll tới cái img trên--> ko tìm thấy--> kích hoạt sự kiện.Nhìn hình bên dưới để hiểu thêm
![[Pasted image 20260728142417.png]]


### khai thác trên máy nạn nhân
-để khai thác trên máy nạn nhân , ta sẽ dùng 1 *exploit server* ,đại khái nó là 1 đường link độc hại, sẽ gửi cái gói tin http tới máy nạn nhân
*-->* ta sẽ gửi tới máy nạn nhân 1 bản tin html .Hiểu sơ qua là iframe tới trang web của chúng ta, tại đó *onload* sẽ kích hoạt sự kiện : thêm thẻ img có chưá hàm print vào sau hash của url
``` html
<iframe src="https://0a7a00a30425199280770d8e008f00d3.web-security-academy.net/#" onload='this.src+="<img src=x onerror=print()>"'></iframe>
```


![[Pasted image 20260728143540.png]]
*-->* cái web của chúng ta thực thi hàm print.Và cái web nạn nhân là _exploit/...._ nó sẽ embedded cái web của chúng ta qua `<iframe>`

>*tại sao lại cần onload, mà không cho luôn payload vào src ??*
>Để kích hoạt sự kiện **`hashchange`**:
>1. **Cho sẵn vào `src`:** Trang tải xong ngay từ đầu $\rightarrow$ hash không thay đổi $\rightarrow$ `onhashchange` **không chạy**.
>2. **Dùng `onload` để cộng thêm payload:** Trang tải xong trước $\rightarrow$ gắn event listener xong $\rightarrow$ `onload` đổi hash $\rightarrow$ kích hoạt `hashchange` để thực thi DOM XSS.