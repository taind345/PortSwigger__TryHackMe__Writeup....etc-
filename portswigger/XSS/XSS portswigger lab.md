### reflected XSS lab
[[Lab-Reflected XSS into HTML context with nothing encoded]]

### stored xss lab
[[lab-Stored XSS into HTML context with nothing encoded]]


### DOM based XSS
-đầu tiên ta sẽ làm các bài lab liên quan đến việc tận dụng *document.write()* ; hàm này sẽ cho phép nhận đầu vào là 1 element có dạng là script
[[Lab- DOM XSS in document.write sink using source location.search]]

-tiếp theo ta sẽ làm tiếp về một vbaifi liên quan tới source là *document.write* và sink là từ *location.search*; khá giống với bài trên
[[Lab- DOM XSS in document.write sink using source location.searc inside a select element]]

-tiếp theo là 1 lab thực hành việc thao túng sink *innerHTML*, với source đến từ *location.search*  
[[lab-DOM XSS in innerHTML sink using source location.search]]

-lab tiếp theo sẽ luyện tập về DOM xss bằng cách khai thác các thư viện bên ngoài như *jquery* hay *angularjs* 
[[DOM XSS in jQuery anchor href attribute sink using location.search  source]]
- tiếp tục với jquery ta sẽ tiếp tục luyện tập 
[[ DOM XSS in jQuery selector sink using a hashchange event]]
- tiep tuc khai thác XSS qua lỗ hổng của angular js [[ DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded]]

-khai thac lo hong DOM+ reflected xss
[[LAB-Reflected DOM XSS]]
