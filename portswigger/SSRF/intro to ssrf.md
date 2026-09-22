**note ở phần này ở đây ==>** [[0-note-ssrf#19/9]]
# I)Intro 
## What is SSRF?

**Server-Side Request Forgery (SSRF)** is a vulnerability that allows an attacker to cause the server-side application to make HTTP requests to a destination of the attacker's choosing. In a typical SSRF attack, the attacker manipulates a parameter that the application uses to construct a server-side request, redirecting it to an internal service, a cloud metadata endpoint, or an external server under their control.

SSRF exploits the trust that internal systems place in the application server. Backend services, databases, and cloud infrastructure often accept requests from the server without additional authentication, because they assume any request arriving from a trusted internal IP address is legitimate. An attacker who can control where the server sends its requests effectively inherits that trust.

## Types of SSRF

There are two categories of SSRF vulnerability, and the distinction affects how exploitation is approached.

|Type|Response Visible?|Description|
|---|---|---|
|Regular SSRF|Yes|The response from the back-end request is returned in the application's front-end response. The attacker can directly read the output.|
|Blind SSRF|No|The application makes the back-end request but does not return the response. The attacker must use indirect methods to confirm exploitation.|

With a regular SSRF, if an attacker forces the server to fetch an internal admin page, the contents of that page appear directly in the HTTP response. This provides immediate, readable output.

With a Blind SSRF, the application may display a fixed success message regardless of the back-end outcome. However, blind SSRF can still be exploited. An attacker can confirm the vulnerability by directing the request to a server they control (using a tool such as Burp Collaborator) and observing whether a callback arrives. Differences in response time or error messages between reachable and unreachable hosts can also reveal information about internal infrastructure.

## Impact

The impact of SSRF depends on what internal services are reachable from the application server.

| Impact                          | Description                                                                                                                                                                                                             |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Access to internal endpoints    | Admin panels, configuration interfaces, and monitoring dashboards that are not exposed to the internet become reachable. IP-based access controls are bypassed because the request originates from the server itself.   |
| Sensitive data exposure         | Backend databases, private APIs, and internal tooling that trust the server's network position may return customer data, organisational records, or application secrets.                                                |
| Internal network reconnaissance | By sending requests to different IP addresses and ports, an attacker can map internal hosts and services using variations in response time, status codes, and error messages.                                           |
| Cloud metadata theft            | Cloud providers such as AWS, GCP, and Azure expose instance metadata at `169.254.169.254`. An attacker who reaches this endpoint can retrieve temporary credentials, IAM role details, and instance configuration data. |
| Credential and token leakage    | Authentication tokens and secrets passed between internal services can be intercepted, particularly where back-end communication runs over unencrypted HTTP.                                                            |

In the following tasks, we will examine how SSRF manifests in different application features, how to identify it, and how to bypass common defences.

> [!NOTE]
> phần này nói về những thứ đã biết

# 2-các dạng thao túng url parameter trong ssrf

![[Pasted image 20260919145554.png]]![[Pasted image 20260919145543.png]]SSRF không phải lúc nào cũng phơi mặt ra thành một cái URL hoàn chỉnh nằm tòng teng trong query parameter đâu. Cách một con app nhét input của người dùng vào request ở phía server muôn hình vạn trạng, và việc bắt bài được mấy cái pattern này là kỹ năng sinh tồn để tìm ra lỗ hổng. Trong phần này, mình sẽ soi 4 vector SSRF phổ biến nhất.

**1. URL hoàn chỉnh trong Parameter (Full URL in a Parameter)**
Đây là form SSRF trực diện và "thô thiển" nhất. Con app ngoan ngoãn nhận nguyên một cái URL hoàn chỉnh từ input và dùng nó để bắn request ở phía server. Mấy cái tính năng như preview link, cấu hình webhook, hay xuất file PDF rất hay xài pattern này.

Thử tưởng tượng một con app có endpoint check hàng tồn kho như vầy:
[https://website.thm/item/2?server=api]

App sẽ bế cái giá trị của tham số `server` rồi đắp vào để tạo thành một request gửi đến [https://server.website.thm/api/item?id=2] Dĩ nhiên, anh em mình (attacker) có thể tráo cái giá trị này để bẻ lái request đi chỗ khác:

| Input | Request thực tế phía Server |
| --- | --- |
| `server=api` | `[https://server.website.thm/api/item?id=2](https://server.website.thm/api/item?id=2)` |
| `server=server.website.thm/flag?id=9&x=` | `[https://server.website.thm/flag?id=9&x=/api/item?id=2](https://server.website.thm/flag?id=9&x=/api/item?id=2)` |

> [!NOTE] sử dụng &x=
> Chú ý cái trường hợp thứ hai, việc nhét thêm `&x=` ở đuôi là một trick cực hay. Nó biến mọi râu ria mà con app tự động nối thêm vào URL trở thành một parameter `x` vô dụng, vô hiệu hóa hoàn toàn cái đuôi phiền phức đó.
![[Pasted image 20260919155013.png]]

**2. URL một nửa - Chỉ có Hostname hoặc Path (Partial URL)**
Nhiều app cẩn thận hơn, chỉ nhận đúng cái hostname hoặc một khúc path, rồi để server tự ghép phần còn lại. Mấy ông dev thường nghĩ làm vậy là thu hẹp được bề mặt tấn công rồi. Cơ mà nằm mơ đi, attacker vẫn có thể tuồn cái hostname do chúng kiểm soát vào.

Ví dụ, với cái request:
[https://website.thm/stock?server=api.internal]
Con app sẽ tự ghép thành [https://api.internal/stock/item]. Nếu cái tham số `server` không được check bằng một allow list (danh sách trắng) đàng hoàng, mình cứ việc tráo nó thành:
[https://website.thm/stock?server=attacker.com]

Thế là con server ngoan ngoãn gọi thẳng về domain của mình. Nếu data trả về được in lên web (reflected), mình có thể bú trọn data nội bộ. Còn nếu là dạng mù (Blind SSRF), ít nhất mình cũng chốt được là app có lỗi vì server của mình có nhận được ping kết nối.


**3. Chèn Path Traversal vào URL**
Khi mày chỉ kiểm soát được mỗi một khúc path, hãy xài tuyệt chiêu "dịch chuyển thư mục" (directory traversal sequences) để nhảy vọt ra khỏi cái endpoint bị nhốt.

Ví dụ, nếu app ghép request kiểu vầy:
[https://website.thm/stock?url=/item/123/details]

Mày phang ngay cục `/../admin` vào, ép con server phải bẻ lái gọi đến:
[https://website.thm/admin]

> [!NOTE]
> Nó y chang cái trick Traversal trong mấy lỗi LFI (File Inclusion) ấy, khác cái là áp dụng cho URL path thay vì đường dẫn thư mục trên ổ cứng thôi.


**4. Lẩn khuất trong Hidden Form Fields**
Không phải lỗ hổng SSRF nào cũng phơi tơ hơ trên thanh URL đâu. Nhiều cái lẩn như chạch trong source HTML của trang và mày phải soi bằng tay (manual inspection) hoặc chặn bắt request qua proxy mới thấy.

Điển hình nhất là tính năng đổi avatar, đường dẫn ảnh bị giấu trong một trường input ẩn:
`<input type="hidden" name="avatar" value="/images/avatars/default.png">`

Nếu server dại dột bốc tài nguyên từ bất cứ đường dẫn nào cái trường này báo về, mày có thể sửa lại giá trị (dùng Developer Tools của trình duyệt hoặc xài tool proxy như Burp Suite) để trỏ thẳng vào mạng nội bộ. Đó là lý do tại sao pentester có tâm phải soi kỹ từng cái form ẩn, request API, và bất kỳ parameter nào chui vào server.


# 3-dấu hiệu nhận biết ssrf
Identifying SSRF during an engagement requires **knowing where applications accept input that influences server-side requests.** In this task, we will cover the most common indicators and how to confirm the vulnerability when the response is not directly visible.

## Common Indicators

The following patterns are strong signals that an application may be vulnerable to SSRF.

**Full URL in a parameter.** When a complete URL appears as a query parameter in the address bar, the application is almost certainly using it to make a server-side request:

![Full URL in a parameter](https://cdn-images.tryhackme.com/user-uploads/5efe36fb68daf465530ca761/room-content/956e1914b116cbc9e564e3bb3d9ab50a.png)

**Hidden form fields.** These are not visible on the rendered page. Inspecting the page source or intercepting requests with a proxy reveals fields whose values control server-side resource fetching:

![Hidden field in a form](https://cdn-images.tryhackme.com/user-uploads/5efe36fb68daf465530ca761/room-content/237696fc8e405d25d4fc7bbcc67919f0.png)

**Partial URL (hostname only).** The application accepts a hostname and constructs the full URL on the server side:

![Partial URL with hostname](https://cdn-images.tryhackme.com/user-uploads/5efe36fb68daf465530ca761/room-content/f3c387849e91a4f15a7b59ff7324be75.png)

**Path only.** Only the path portion of the URL is user-controlled. The application prepends the scheme and hostname:

![Path only in a parameter](https://cdn-images.tryhackme.com/user-uploads/5efe36fb68daf465530ca761/room-content/3fd583950617f7a3713a107fcb4cfa49.png)

Beyond these four patterns, the following application features frequently contain SSRF vectors:

| Feature               | Why It's Relevant                                                                |
| --------------------- | -------------------------------------------------------------------------------- |
| Webhook configuration | The application makes a request to a user-supplied URL to verify the endpoint.   |
| PDF/report generation | The server fetches content from a supplied URL to render it into a document.     |
| URL preview/unfurling | The application retrieves metadata (title, thumbnail) from a user-provided link. |
| File import by URL    | The server downloads a file from a remote location specified by the user.        |
| Integration settings  | Third-party service URLs are stored and queried by the server.                   |

Some of these cases are easier to exploit than others. A full URL in a query parameter is straightforward to test, while a partial path segment may require considerable trial and error to produce a working payload. The important step is recognising the pattern first, then experimenting with different inputs.

## Confirming Blind SSRF

When the server makes the request but does not reflect the response, you need an alternative method to confirm the vulnerability.

|Method|How It Works|
|---|---|
|External HTTP logger (e.g. requestbin.com)|Supply the logger's URL as the SSRF payload. Check the dashboard for incoming requests from the target server.|
|Burp Collaborator|Generates a unique domain that logs HTTP and DNS callbacks. Useful when HTTP is blocked but DNS resolution still occurs.|
|Self-hosted listener (`python3 -m http.server`)|Run a simple HTTP server on your own machine and monitor for incoming connections from the target.|
|Timing analysis|Compare response times for requests to internal hosts that exist versus hosts that do not. Consistent differences indicate the server is resolving and connecting to the supplied address.|
|Error-based inference|Different error messages for reachable versus unreachable hosts reveal information about the internal network, even when the actual response body is hidden.|

These techniques are covered in more depth in later rooms. For now, confirming that the server is making outbound requests based on your input is sufficient to establish the vulnerability.

> [!NOTE]
> ->đây là dạng blind, mình cần có 1 domain , hoặc burpcolapborator để xem liệu server có request về cái url mình đưa vào input hay ko
> ->



# 3-redirect

Các developer biết về rủi ro SSRF thường sẽ cài cắm thêm bước kiểm tra đầu vào (input validation) để giới hạn những nơi mà server có thể gửi request tới. Các chốt chặn này thường rơi vào một trong ba loại: **deny lists** (danh sách đen), **allow lists** (danh sách trắng) và lợi dụng **open redirect** (chuyển hướng mở). Trong phần này, mình sẽ soi từng lớp phòng thủ một và các trick để lách (bypass) qua chúng.

> [!NOTE] Title
> Có 3 cái trên

**Deny Lists (Danh sách đen)**

Danh sách đen sẽ chặn đứng các request gửi tới những địa chỉ hoặc dải IP cụ thể, còn lại thì thả cửa. Mục đích là để cấm cửa các điểm đến nhạy cảm đã biết tỏng như `localhost`, `127.0.0.1` hay các endpoint chứa cloud metadata.

Thế nhưng, deny list bản chất mỏng manh vãi chưởng. Cái địa chỉ loopback IPv4 `127.0.0.1` có hàng tá vỏ bọc (representation) khác nhau. Nếu cái deny list chỉ chặn những dạng phổ biến nhất thì kiểu gì cũng bị lọt lưới.

| Cách biểu diễn (Representation) | Giá trị (Value) |
| --- | --- |
| Tiêu chuẩn (Standard) | `127.0.0.1` |
| Thập phân (Decimal) | `2130706433` |
| Bát phân (Octal) | `017700000001` |
| Dạng rút gọn (Shorthand) | `127.1` hoặc `0` hoặc `0.0.0.0` |
| Ký tự đại diện (Wildcard) | `127.*.*.*` |
| IPv6 | `[::1]` |
| Dựa trên DNS (DNS-based) | `127.0.0.1.nip.io` |

Cái trò dùng DNS là hiệu quả và thâm độc nhất. Mấy dịch vụ như `nip.io` cho phép kẻ tấn công tạo ra các subdomain trỏ tới bất kỳ IP nào. Ví dụ, một cái hostname như `127.0.0.1.nip.io` sẽ luôn phân giải ra đúng `127.0.0.1`. Tuy nhiên, mấy cái deny list quét chuỗi (string-based) ngây thơ nhìn vào thì chỉ thấy đây là một tên miền bình thường và vui vẻ cho qua.

Trong môi trường Cloud, deny list xịn cũng phải chặn luôn cái IP `169.254.169.254` (cái metadata endpoint chuyên dùng của AWS, GCP và Azure). Mặc dù vậy, kẻ tấn công hoàn toàn có thể tự bỏ tiền túi mua một cái tên miền, cài bản ghi DNS trỏ thẳng tới `169.254.169.254`. Deny list check cái chuỗi tên miền, thấy không khớp với danh sách đen nên vẫy tay cho qua. Tới lúc server tự tay phân giải cái tên miền đó và ném request đi thì bùm, nó đâm thẳng vào dịch vụ metadata.



**Allow Lists (Danh sách trắng)**

Danh sách trắng thì gắt hơn: mặc định là khóa mõm chặn mọi request, trừ phi đích đến khớp đúng với một entry hoặc pattern đã được phê duyệt. Ví dụ, app có thể bắt buộc URL phải luôn bắt đầu bằng `[https://website.thm](https://website.thm)`. Lớp bảo vệ này chắc cú hơn deny list nhiều. Dù vậy, code lỗi thì vẫn đầy khe hở để lách.

| Kỹ thuật lách (Bypass Technique) | Ví dụ (Example) | Tại sao lại trót lọt? (Why It Works) |
| --- | --- | --- |
| Trùng khớp Subdomain (Subdomain matching) | `[https://website.thm.attackers-domain.thm](https://website.thm.attackers-domain.thm)` | Chuỗi URL vẫn bắt đầu đúng với tiền tố (prefix) bị ép buộc, nhưng thực chất phần hostname lại đang trỏ thẳng về domain do attacker làm chủ. |
| Trượt thông tin đăng nhập (URL credential abuse) | `[https://website.thm@attacker.com/](https://website.thm@attacker.com/)` | Vài thư viện HTTP tách chuỗi rất ngáo: nó coi phần nằm trước chữ `@` là username/password (credentials), còn phần sau `@` mới là hostname thực sự. Thế là allow list check thấy chữ `website.thm` thì duyệt qua, nhưng request lại chạy vèo sang `attacker.com`. |

Ở cả hai trường hợp, nguyên nhân cốt lõi là một: con app thẩm định cái chuỗi URL bằng mấy logic so khớp pattern đơn giản thay vì bóc tách (parse) URL ra thành từng thành phần chuẩn chỉ.


**Open Redirects (Lợi dụng chuyển hướng mở)**

Nếu gặp phải tường lửa sắt đá, lách deny/allow list đều tịt ngòi, thì kẻ tấn công có thể mò xem trên domain mục tiêu có lỗi Open Redirect (chuyển hướng mở) nào không để "mượn đao giết người". Open redirect là một cái endpoint tự động đá khách truy cập sang một URL được chỉ định sẵn trong tham số. Tính năng này hay được dùng để tracking các lượt click link ra ngoài.

Ví dụ, ngó thử endpoint này xem:
[https://website.thm/link?url=https://tryhackme.com]

Thằng endpoint này có nhiệm vụ ghi log lượt click xong sẽ chuyển hướng (redirect) khách sang [https://tryhackme.com](https://tryhackme.com). Nếu cơ chế chống SSRF của app này cực gắt, chỉ cho phép URL bắt đầu bằng [https://website.thm/](https://website.thm/), attacker có thể nối (chain) cái open redirect này chung với SSRF:

[https://website.thm/link?url=http://169.254.169.254/latest/meta-data/](https://website.thm/link?url=http://169.254.169.254/latest/meta-data/)

Cái allow list sẽ gật gù cho qua vì rõ ràng URL bắt đầu bằng domain uy tín. Thế nhưng, khi server thực thi request đó, nó va phải cái endpoint open redirect, và thế là bị "sút" thẳng tắp sang dịch vụ cloud metadata. Nôm na là dùng chính tính năng của app để phá vỡ lớp bảo vệ của app.

> [!NOTE] hiểu đơn giản
> là cả cái link đi qua allowlist thì nó ok do có .thm ở url, nhưng mà do chức năng cái enpoint này là redirect==> giúp bypass allow list trong url

Cách bypass này đỉnh ở chỗ nó kết nối hai tính năng (mà nếu đứng tách riêng thì trông có vẻ cực kỳ vô hại) lại với nhau. Nó là bài học nhãn tiền cho việc: xây dựng chốt chặn bảo mật là phải đánh giá cả sự tương tác chéo giữa các tính năng, chứ không chỉ test hời hợt hành vi của từng cái một.

# 5- LAB

> [!NOTE] đơn giản như này
> 1-enpoint-->**/customers/new-account-page** -> thằng này cho phép sửa avatar, bằng cách refenrence 1 cái url chứa ảnh
> 2-> đơn giản là bắt burpsuite xem cái cách url ntn?
> 3-> sau đó là truyền input sao cho sever nhận input và request tới enpoint **private**
> 

## Scenario

During a content discovery exercise against the Acme IT Support website, two endpoints have been identified:

| Endpoint                      | Behaviour                                                                                                                                |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `/private`                    | Returns an error stating the contents cannot be viewed from your IP address. Access is restricted based on the source IP of the request. |
| `/customers/new-account-page` | A newer version of the customer account page. Includes a feature for selecting a profile avatar.                                         |

The `/private` endpoint is the target. It contains restricted content that is only accessible to requests originating from the server itself. The avatar feature on `/customers/new-account-page` is the attack vector.

Click the **Start Lab Machine** button to launch the Acme IT Support website. Once running, visit it at `https://LAB_WEB_URL.p.thmlabs.com` and follow the steps below.

### Step 1: Locate the Avatar Feature

1. Create a customer account on the site and sign in.
2. Navigate to `https://LAB_WEB_URL.p.thmlabs.com/customers/new-account-page`.
3. Right-click the page and select **View Page Source** (or press `Ctrl+U`).

In the source, each avatar option is a radio button whose `value` attribute contains the path to an image file. The `background-image` CSS property on the surrounding `<div>` element confirms this:

![Avatar form field value containing the image path](https://cdn-images.tryhackme.com/user-uploads/5efe36fb68daf465530ca761/room-content/bd9ee9ac0b7592b5343cbc8dd9b57189.png)

The server uses the value from this form field to fetch the resource. If the value is not validated, it can be redirected to any path on the server.

### Step 2: Observe How the Server Handles the Request

Select one of the avatars and click **Update Avatar**. The page updates to display the selected avatar:

![Currently selected avatar displayed on the page](https://cdn-images.tryhackme.com/user-uploads/5c549500924ec576f953d9fc/room-content/8685bf7a4b24616031425a7f5e8db1ae.png)

Inspect the page source again. The avatar is now rendered using the data URI scheme, with the image content base64-encoded in the `src` attribute:

![Base64-encoded avatar in page source](https://cdn-images.tryhackme.com/user-uploads/5efe36fb68daf465530ca761/room-content/fff0ea113602635dcf5d1e8d0b1d8bca.png)

This behaviour is significant. The server fetches the resource at the path specified in the form field, reads the response, and encodes it into the page. If the server can be directed to fetch `/private` instead of an image, its contents will appear as base64-encoded data in the page source.

> [!NOTE] Title
> **=>** thằng này nó mã hóa url bằng base64 -> sau đó đưa về server , giải mã , rồi reference cái url đấy thôi

### Step 3: Attempt Direct Access to /private

1. Right-click one of the avatar radio buttons and select **Inspect**:

![Right-click and select Inspect on a radio button](https://cdn-images.tryhackme.com/user-uploads/5efe36fb68daf465530ca761/room-content/2ef87608418e47625bedad9d0361ed08.png)

2. Change the `value` attribute from the image path to `private`:

![Editing the radio button value to private](https://cdn-images.tryhackme.com/user-uploads/5efe36fb68daf465530ca761/room-content/a1712298679cc642d792d935b14effe5.png)

3. Select the modified radio button and click **Update Avatar**.

The application returns an error indicating the path cannot start with `/private`:

![Deny list error message blocking /private](https://cdn-images.tryhackme.com/user-uploads/5efe36fb68daf465530ca761/room-content/a59460cc19eaf5776ee8a882e25b2d64.png)

The application has a deny list that blocks requests where the path begins with `/private`.

### Step 4: Bypass the Deny List

The deny list performs a string match against the start of the path. However, as covered in Task 4, deny lists that only check the raw input can be bypassed using directory traversal.

Change the radio button's `value` attribute to:

```bash
x/../private
```

![Setting the avatar value to x/../private](https://cdn-images.tryhackme.com/user-uploads/5efe36fb68daf465530ca761/room-content/84b88d9c6fa6a29450520625bb42870d.png)

The following table shows why this works:

|Stage|Path|Explanation|
|---|---|---|
|Input validation|`x/../private`|The deny list checks the raw string. It does not begin with `/private`, so the check passes.|
|Path normalisation|`/private`|The web server resolves `x/../private` by entering directory `x`, then moving up one level with `../`, arriving at `/private`.|

The deny list and the web server interpret the path at different stages. In this case, the validation checks the string before normalisation occurs, allowing the traversal to bypass the restriction.

Select the modified radio button and click **Update Avatar**. The request succeeds.

### Step 5: Decode the Flag

View the page source. The avatar `<img>` tag now contains base64-encoded data representing the contents of `/private` rather than an image file.

Copy the base64 string and decode it:

```bash
echo "PASTE_BASE64_STRING_HERE" | base64 -d
```

The decoded output contains the flag.
