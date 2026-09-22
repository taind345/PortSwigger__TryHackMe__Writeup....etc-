[[note-22-9]]
# 1-intro
Con người luôn tìm kiếm những cách thức đơn giản và hiệu quả hơn để làm việc. Ngay từ khi bắt đầu lập trình và phát triển phần mềm, chúng ta đã tìm cách tự động hóa một số tác vụ. Ngày nay, tự động hóa gắn liền mật thiết với Vòng đời phát triển phần mềm (SDLC) và các quy trình DevOps. Mặc dù điều này mang lại lợi ích to lớn cho hệ thống, giúp phát triển và triển khai nhanh hơn, nhưng nó cũng đi kèm với những rủi ro bảo mật mới.

Khi các quy trình này được làm thủ công, kẻ tấn công sẽ phải xâm phạm thông tin xác thực hoặc máy trạm của cá nhân thực hiện quy trình đó. Tuy nhiên, với tự động hóa, kẻ tấn công giờ đây có thể nhắm thẳng vào chính pipeline (chuỗi quy trình).

**Mục tiêu học tập**
Room này sẽ giúp bạn tìm hiểu về các khái niệm sau:

* Giới thiệu về DevOps pipeline
* Giới thiệu về tự động hóa và các công cụ DevOps
* Giới thiệu về các nguyên tắc bảo mật cho DevOps pipeline

Vì đây là room giới thiệu, hầu hết các khái niệm trên sẽ chỉ được đề cập sơ lược và sẽ được đi sâu vào chi tiết trong các room còn lại của module này.
# 2- devops pipe line explain
Before learning about automation security, we should start by defining the pipeline and showing where automation can take place. The diagram below shows what a typical pipeline can look like, as well as the software that could be used for this purpose:
![Pipeline diagram](https://cdn-images.tryhackme.com/user-uploads/6093e17fa004d20049b6933e/room-content/5bf9574f4b8f6bc202123c9476650e58.png)  

For each of these items, we will look at what they are, the common tools used for them, an introduction to their security, and a case study of what can happen when security fails. Each of these components will be reviewed in-depth in the coming rooms of this module.

> [!NOTE]
> cái này nói về pipeline trong sofware deverlopment 
> -> hiểu đơn giàn nó là các quy trình tự động hóa trong việc phát triển phần mềm
> --> liệu trong security, thì các pepline tự động hóa sẽ là gì?

# 3-soucre code and version control

> [!NOTE]
> mình sẽ dần tìm hiểu các thứ trong ảnh trên có nghĩa là gì

![[Pasted image 20260922143625.png|492]]Hãy cùng tìm hiểu về mã nguồn (source code) và kiểm soát phiên bản (version control). Đây là điểm khởi đầu cho pipeline của chúng ta. Chúng ta cần một vị trí để lưu trữ mã nguồn. Hơn thế nữa, chúng ta thường muốn lưu giữ nhiều phiên bản khác nhau của mã nguồn vì chúng ta liên tục cải tiến và bổ sung các tính năng.

**Lưu trữ mã nguồn**
Chúng ta cần xem xét một số yếu tố khi quyết định nơi lưu trữ mã nguồn:

* Làm cách nào để thực hiện kiểm soát truy cập cho mã nguồn?
* Làm cách nào để đảm bảo rằng các thay đổi được theo dõi?
* Liệu chúng ta có thể tích hợp hệ thống lưu trữ mã nguồn với các công cụ phát triển của mình không?
* Chúng ta có thể lưu trữ và chủ động sử dụng nhiều phiên bản mã nguồn khác nhau không?
* Chúng ta nên lưu trữ mã nguồn trên máy chủ nội bộ hay sử dụng một bên thứ ba để lưu trữ?

Câu trả lời cho những câu hỏi này sẽ giúp chúng ta chọn ra giải pháp lưu trữ mã nguồn phù hợp cho dự án của mình.

> [!NOTE] Title
> **=>** lưu source code ở đâu ?--> github??

**vesion control**
Chúng ta cần kiểm soát phiên bản vì hai lý do chính:

1. Chúng ta thường xuyên tích hợp các tính năng mới vào phần mềm. Các phương pháp phát triển hiện đại, chẳng hạn như Agile, đồng nghĩa với việc chúng ta liên tục cập nhật code. Để kiểm soát tất cả các bản cập nhật này, chúng ta cần kiểm soát phiên bản.
2. Toàn bộ đội ngũ phát triển đang cùng làm việc trên một mã nguồn, chứ không chỉ một nhà phát triển. Để đảm bảo rằng chúng ta có thể tích hợp các thay đổi từ nhiều nhà phát triển, việc kiểm soát phiên bản là bắt buộc.

Kiểm soát phiên bản cho phép chúng ta giữ nhiều phiên bản của code. Đó có thể là phiên bản cụ thể mà mỗi lập trình viên đang làm việc, nhưng nó cũng có thể là các phiên bản hoàn toàn khác nhau của ứng dụng, bao gồm cả các phiên bản phụ (minor) và phiên bản chính (major).

**Các công cụ phổ biến**
Hai hệ thống lưu trữ mã nguồn và kiểm soát phiên bản phổ biến nhất là Git và SubVersion (SVN). Git là một công cụ kiểm soát mã nguồn phân tán, nghĩa là mỗi người đóng góp sẽ có một bản sao mã nguồn riêng của họ. Ngược lại, SVN là một công cụ kiểm soát mã nguồn tập trung, nghĩa là việc kiểm soát kho lưu trữ (repo) được quản lý ở vị trí trung tâm.

GitHub hiện là nhà cung cấp dịch vụ lưu trữ Internet lớn nhất cho việc phát triển phần mềm và kiểm soát phiên bản bằng Git. Bạn có thể tạo tài khoản GitHub và sử dụng nó để quản lý các repo mã nguồn của mình. Tuy nhiên, bạn cũng có thể tự cài đặt máy chủ Git riêng bằng các phần mềm như GitLab. Đối với SVN, hai công cụ phổ biến nhất là TortoiseSVN và Apache SVN.

Tuy nhiên, cần lưu ý rằng các giải pháp lưu trữ mã nguồn như GitLab cung cấp nhiều tính năng hơn là chỉ lưu trữ và kiểm soát phiên bản đơn thuần. Ngày nay, những công cụ này có thể được sử dụng cho gần như toàn bộ pipeline!

**Các lưu ý về bảo mật**
Mã nguồn thường là "vũ khí bí mật" của chúng ta. Vì vậy, chúng ta muốn đảm bảo rằng nó không bị lộ. Đó là lý do tại sao xác thực và kiểm soát truy cập cho mã nguồn lại quan trọng đến vậy. Chúng ta cũng muốn đảm bảo rằng các thay đổi và cập nhật được theo dõi đầy đủ, cho phép chúng ta luôn có thể khôi phục lại phiên bản trước đó nếu có sự cố xảy ra.

Tuy nhiên, chúng ta cũng cần cẩn trọng với những gì mình lưu trữ dưới dạng mã nguồn. Mã nguồn không thể bảo mật tuyệt đối vì các nhà phát triển vẫn cần quyền truy cập vào nó. Do đó, chúng ta không nên nhầm lẫn giữa việc lưu trữ mã nguồn với quản lý thông tin bí mật (secret management). Chúng ta cần đảm bảo không lưu các thông tin bí mật, chẳng hạn như chuỗi kết nối cơ sở dữ liệu và thông tin xác thực, bên trong mã nguồn. Vì chúng ta lưu giữ tất cả các phiên bản của mã nguồn, nên ngay cả khi ta xóa bỏ những bí mật đó ở phiên bản mới, chúng vẫn sẽ bị lộ ở các phiên bản trước.

**Nghiên cứu điển hình: Git không bao giờ quên**
Như đã đề cập, kiểm soát phiên bản có thể để lại hậu quả tồi tệ nếu chúng ta mắc sai lầm. Đây là một vấn đề phổ biến khi sử dụng các công cụ như Git. Có một câu nói: "Git không bao giờ quên". Code được "commit" (lưu) vào một repo Git. Khi điều này xảy ra, Git sẽ xác định các thay đổi đối với tệp và tạo một phiên bản mới dựa trên các thay đổi này. Bất kỳ người dùng nào có quyền truy cập vào repo đều có thể xem lịch sử các commit và những thay đổi đã được thực hiện.

Điều thường xảy ra là một nhà phát triển vô tình commit các thông tin bí mật (như thông tin đăng nhập hoặc chuỗi kết nối DB) lên Git repo. Khi nhận ra sai lầm, họ xóa các thông tin bí mật này đi và tạo một commit khác. Tuy nhiên, lúc này repo sẽ lưu giữ cả hai commit. Nếu kẻ tấn công có quyền truy cập vào repo, chúng có thể sử dụng các công cụ như GittyLeaks để quét qua các commit nhằm tìm kiếm thông tin nhạy cảm. Ngay cả khi thông tin này không còn tồn tại trong phiên bản hiện tại, các công cụ này vẫn có thể lục soát lại tất cả các phiên bản trước đó để thu thập những bí mật này.

> [!NOTE]
> Một khi đã commit là sẽ có dấu vết trên git--> có cách nào xóa sạch k?

Hãy cùng bàn về các **dependencies** (thành phần phụ thuộc). Mặc dù chúng ta có thể nghĩ rằng mình đang viết một lượng lớn code trong quá trình phát triển (develop), nhưng sự thật đó chỉ là phần nổi của tảng băng chìm. Trừ khi bạn đang code bằng mã nhị phân (binary), rất có thể thực tế bạn chỉ đang viết một phần nhỏ của toàn bộ lượng code thực sự. Điều này là do rất nhiều code đã được viết sẵn cho chúng ta dưới dạng các **libraries** (thư viện) và **SDKs** (Software Development Kits). Ngay cả những biến số như `String` trong một ứng dụng cũng có cả một library đứng sau nó! Việc quản lý các dependencies này là một phần thiết yếu của **pipeline**.

# 3-dependency management
## External và Internal Dependencies

**External dependencies** là các libraries và SDKs được cung cấp công khai. Chúng được lưu trữ trên các **dependency managers** bên ngoài như PyPi cho Python, NuGet cho .NET và Gems cho các libraries của Ruby.

**Internal dependencies** là các libraries và SDKs do một tổ chức tự phát triển và duy trì nội bộ. Ví dụ, một tổ chức có thể phát triển một authentication library (thư viện xác thực). Library này sau đó có thể được sử dụng cho tất cả các ứng dụng do tổ chức phát triển.

Có những mối lo ngại về bảo mật (security concerns) khác nhau đối với internal và external dependencies:

| Internal Dependencies                                                                                                                                        | External Dependencies                                                                                                                                                                                                   |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Các libraries thường có thể trở thành **legacy software** (phần mềm lỗi thời) do không còn nhận được các bản cập nhật hoặc developer ban đầu đã rời công ty. | Vì chúng ta không có **full control** (toàn quyền kiểm soát) đối với dependency, chúng ta phải thực hiện **due diligence** (thẩm định) để đảm bảo rằng library đó an toàn.                                              |
| Bảo mật của **package manager** đối với các internal libraries là trách nhiệm của chúng ta.                                                                  | Nếu một package manager hoặc **CDN** (Content Distribution Network) bị **compromised** (xâm phạm), nó có thể dẫn đến một **supply chain attack** (tấn công chuỗi cung ứng).                                             |
| Một **vulnerability** (lỗ hổng) trong internal library có thể ảnh hưởng đến một số ứng dụng của chúng ta vì nó được sử dụng trong tất cả các ứng dụng đó.    | Các external libraries có thể bị **attackers** (kẻ tấn công) nghiên cứu để tìm ra các **0day vulnerabilities**. Nếu phát hiện ra vulnerability như vậy, nó có thể dẫn đến việc xâm phạm hàng loạt tổ chức cùng một lúc. |

> [!NOTE]
> SDK==> giống với APi nhưng to hơn và bao gồm APi, như một dịch vụ có sẵn, dùng đẻ tích hợp luôn vào app ==> cổng thanh toán momo
## Common Tools

Một dependency manager, hay còn gọi là package manager, là cần thiết để quản lý các libraries và SDKs. Như đã đề cập trước đó, **các công cụ như PyPi, NuGet và Gems được sử dụng cho external dependencies**. Việc quản lý internal dependencies phức tạp hơn một chút. Đối với những trường hợp này, chúng ta có thể sử dụng các công cụ như **JFrog Artifactory hoặc Azure Artifacts để quản lý**.

## Security Considerations

Một số lưu ý về bảo mật đã được đề cập trước đó. Tuy nhiên, mối lo ngại bảo mật chính là các dependencies là những đoạn code nằm ngoài tầm kiểm soát của chúng ta (outside our control). Đặc biệt trong thời đại hiện nay, khi có quá nhiều dependencies khác nhau được sử dụng, việc **track** (theo dõi) các dependencies là vô cùng khó khăn. Nếu có bất kỳ vulnerabilities nào trong các dependencies này, nó có thể dẫn đến các vulnerabilities trong ứng dụng của chúng ta.

## Case Study: Log4Shell

Một 0day vulnerability đã được phát hiện trong dependency **Log4j** vào năm 2021 có tên là **Log4Shell**. Log4j là một logging utility (tiện ích ghi log) dựa trên Java. Nó là một phần của Apache Logging Services, một dự án của Apache Software Foundation. Vulnerability này có thể cho phép một **unauthenticated attacker** (kẻ tấn công chưa xác thực) đạt được **remote code execution** (thực thi mã từ xa) trên một hệ thống sử dụng logger đó.

Vấn đề thực sự? Dependency nhỏ bé này gần như được sử dụng ở khắp mọi nơi, như minh họa trong bức tranh biếm họa XKCD này (mở trong tab mới). Đây không phải là nói quá. Hãy xem tại đây (mở trong tab mới) để thấy có bao nhiêu sản phẩm khác nhau bị **vulnerable** (dễ bị tổn thương) vì chúng sử dụng dependency này. Danh sách này lớn đến mức họ phải chia nó theo thứ tự bảng chữ cái. Điều này cho thấy mức độ ảnh hưởng của những gì có thể xảy ra khi một vulnerability được phát hiện trong một dependency.
![[Pasted image 20260922150645.png]]
> [!NOTE]
> internal và external dependency==> quản lý thế nào?
> - pyPi,nuget, gems
> - JFrog ; azzure artifact

# 4-Automated testing

> [!NOTE]
> tiếp tục tìm hiểu về cái bước testing trong ci/cd pipeline

Hãy cùng xem xét kỹ hơn về **Automated Testing** (Kiểm thử tự động). Trước đây, kiểm thử là một quá trình thủ công và khá tẻ nhạt. Một tester sẽ phải chạy và ghi chép thủ công từng test case và hy vọng rằng độ bao phủ (coverage) đủ để đảm bảo ứng dụng hoặc dịch vụ hoạt động ổn định. Tuy nhiên, trong các pipeline hiện đại, automated testing có thể đảm nhận một phần lớn công việc này.
## Unit Testing

Khi nói về automated testing trong một pipeline, đây sẽ là loại hình kiểm thử đầu tiên mà hầu hết các developer và kỹ sư phần mềm đều quen thuộc. Một **unit test** là một test case cho một phần nhỏ của ứng dụng hoặc dịch vụ. Ý tưởng là chia nhỏ ứng dụng ra để kiểm tra nhằm đảm bảo mọi chức năng đều hoạt động đúng như thiết kế.

Trong các pipeline hiện đại, unit testing có thể được sử dụng như các **quality gates**. Các test case có thể được tích hợp vào giai đoạn **CI/CD** của pipeline, nơi quá trình build sẽ bị chặn lại nếu các test case này thất bại (fail). Tuy nhiên, unit testing thường chỉ tập trung vào chức năng (functionality) chứ không phải bảo mật (security).

## Integration Testing

Một phương pháp kiểm thử phổ biến khác là **integration testing**. Nếu unit tests tập trung vào các phần nhỏ, thì integration testing tập trung vào cách các phần nhỏ này hoạt động cùng nhau. Tương tự như unit tests, việc kiểm thử sẽ được thực hiện cho từng phần tích hợp và cũng có thể được đưa vào giai đoạn CI/CD của pipeline.

Một tập hợp con của integration testing là **regression testing** (kiểm thử hồi quy), nhằm mục đích đảm bảo các tính năng mới không gây ảnh hưởng xấu đến các chức năng hiện có. Dù vậy, giống như unit testing, integration testing (bao gồm cả regression testing) thường không được thực hiện vì mục đích bảo mật.

## Security Testing

Vậy nếu hai loại automated testing đầu tiên không dành cho bảo mật, thì loại nào mới phải? Có hai loại automated security testing chính:
### SAST

**Static Application Security Testing (SAST)** hoạt động bằng cách review mã nguồn của ứng dụng hoặc dịch vụ để xác định các nguồn gốc gây ra lỗ hổng (vulnerabilities). Các công cụ SAST có thể được dùng để scan mã nguồn nhằm tìm kiếm vulnerabilities. Quá trình này có thể được tích hợp thẳng vào quy trình phát triển để cảnh báo sớm các vấn đề tiềm ẩn cho developer ngay khi họ đang viết code. Chúng ta cũng có thể đưa nó vào quy trình CI/CD. Không phải dưới dạng quality gates, mà là **security gates**, ngăn pipeline chạy tiếp nếu công cụ SAST vẫn phát hiện ra vulnerabilities mà chưa được đánh dấu là **false positives** (cảnh báo giả).

> [!NOTE]
> - [ ] qality gate
> - [ ] security gate
> -> 1 thằng kiểm thử ở khâu cuối, 1 thằng ngăn cho pipeline chạy khi có cảnh báo bảo mật khi static security testing

### DAST

**Dynamic Application Security Testing (DAST)** tương tự như SAST nhưng thực hiện kiểm thử động (dynamic testing) bằng cách thực thi code. Điều này cho phép các công cụ DAST phát hiện thêm các vulnerabilities mà nếu chỉ review source code thì không thể thấy được.

Một phương pháp mà DAST sử dụng để tìm ra các vulnerabilities bổ sung (chẳng hạn như **XSS - Cross Site Scripting**) là tạo ra các **sources** và **sinks**. Khi một công cụ DAST nhập dữ liệu vào một trường (field) trong ứng dụng, nó đánh dấu đó là một source. Khi dữ liệu được ứng dụng trả về, nó sẽ tìm lại parameter cụ thể này và nếu thấy, nó sẽ đánh dấu đó là một sink. Sau đó, nó có thể gửi các dữ liệu mang tính độc hại (malicious data) vào source và dựa trên những gì hiển thị ở sink để xác định xem có tồn tại vulnerability như XSS hay không. Giống như SAST, các công cụ DAST cũng có thể được tích hợp vào CI/CD pipeline dưới dạng security gates.

## Penetration Testing (Pentest)

Đáng tiếc là các công cụ SAST và DAST không thể thay thế hoàn toàn manual testing (kiểm thử thủ công), điển hình như **Penetration Testing**. Đã có những tiến bộ đáng kể trong automated testing, thậm chí các kỹ thuật này còn được kết hợp với những phương pháp hiện đại hơn để tạo ra các kỹ thuật kiểm thử mới như **IAST** (Interactive Application Security Testing) và **RASP** (Runtime Application Self-Protection).

Tuy nhiên, vấn đề cốt lõi vẫn là các công cụ này không hoạt động tốt khi đối mặt với các **contextual vulnerabilities** (lỗ hổng theo ngữ cảnh). Lấy luồng quy trình thanh toán làm ví dụ. Một vulnerability phổ biến là khi một phần của quy trình có thể bị bypass, chẳng hạn như bước xác thực thẻ tín dụng. Đây là một test case rất dễ thực hiện thủ công, nhưng vì nó đòi hỏi ngữ cảnh (context), nên ngay cả công cụ DAST cũng sẽ rất khó để phát hiện ra lỗi bypass này. Tương tự, các lỗi liên quan đến **business logic** và **access control** cũng rất khó bị phát hiện bởi automated tools, trong khi manual testing có thể tìm ra chúng khá nhanh chóng. Việc sử dụng manual testing cho các lỗi này đơn giản là tối ưu chi phí (cost-effective) hơn.

## Common Tools

Có một số công cụ phổ biến có thể được sử dụng cho automated testing. Cả GitHub và GitLab đều có sẵn bộ công cụ SAST tích hợp. Các công cụ như **Snyk** và S**onarQube** cũng rất thịnh hành cho việc triển khai SAST và DAST.

## Case Study: Quá tải hiệu năng hệ thống

Một vấn đề phổ biến đối với các công cụ SAST và DAST là chúng đơn giản chỉ được deploy vào pipeline, đôi khi chỉ để làm một **Proof-of-Concept (PoC)**. Tuy nhiên, bạn cần xem xét một vài yếu tố sau:

* Performance cost (Chi phí hiệu năng)
* Integration points (Các điểm tích hợp)
* Calibration of results (Hiệu chuẩn kết quả)
* Quality và security gate implementation (Triển khai các cổng chất lượng và bảo mật)

Điểm đầu tiên và cuối cùng rất quan trọng, nếu phớt lờ thì cái giá phải trả có thể rất đắt. Quá trình PoC ban đầu của công cụ có lẽ nên diễn ra ngoài giờ làm việc vì nó sẽ phải scan toàn bộ code. Quá trình này có thể làm ảnh hưởng đáng kể đến hiệu năng của công cụ kiểm soát mã nguồn. Hãy tưởng tượng điều này xảy ra ngay trước một đợt release lớn và các developer không thể stage hay push các **commits** mới nhất của họ.

Hơn nữa, khi ngày càng có nhiều tổ chức chuyển sang phương pháp Agile trong phát triển, hầu hết các **repos** nhận hàng trăm commits mỗi ngày. Nếu bạn đưa vào một security gate mới (dù chỉ để làm PoC) có nhiệm vụ scan từng **merge request** để tìm vulnerabilities trước khi approve, điều này có thể gây ra một mức "performance cost" nặng nề cho hạ tầng (infrastructure) cũng như làm chậm tốc độ thực hiện merge request của developer.

Khi áp dụng các công cụ automated testing mới, cần cân nhắc kỹ lưỡng cách thức thực hiện PoC để đảm bảo không gây ra gián đoạn, nhưng đồng thời vẫn đảm bảo bản PoC mang tính đại diện cho cách công cụ đó sẽ tương tác khi được tích hợp chính thức.

# 5-CI/CD
Trong các pipeline hiện đại, phần mềm không còn được chuyển đổi thủ công giữa các môi trường khác nhau. Thay vào đó, một quy trình tự động hóa sẽ được áp dụng để biên dịch (compile), build, tích hợp (integrate) và triển khai (deploy) các tính năng mới. Quy trình này được gọi là **CI/CD**.

> **Lưu ý:** Thuật ngữ CI/CD đã thay đổi khá nhiều trong những năm gần đây. Ban đầu, trọng tâm chỉ là đảm bảo việc phát triển (development) được thực hiện theo phương pháp Agile, trong khi việc bàn giao sản phẩm vẫn diễn ra theo mô hình thác nước (waterfall) — tức chỉ triển khai các bản release cuối cùng. Giai đoạn này, CI/CD thường được hiểu là *Continuous Integration và Continuous Development*.
> Tuy nhiên, người ta nhanh chóng nhận ra rằng bản thân việc deployment cũng có thể áp dụng Agile, và cụm từ viết tắt này được đổi thành *Continuous Integration và Continuous Deployment*, đưa development thành một phần bên trong component Integration. Cuối cùng, họ nhận thấy vấn đề không chỉ dừng lại ở deployment, mà là toàn bộ các khía cạnh xoay quanh việc phân phối giải pháp và cách chúng ta giám sát (monitor) nó sau khi bàn giao. Vì vậy, cụm từ này hiện mang nghĩa là *Continuous Integration và Continuous Delivery*. Dù bạn có thể thấy các thuật ngữ này được dùng thay thế cho nhau, thực chất chúng đều chỉ cùng một quy trình.

## CI/CD

Vì liên tục xây dựng các tính năng mới cho hệ thống hoặc dịch vụ, chúng ta cần đảm bảo các tính năng này tương thích với ứng dụng hiện tại. Thay vì chờ đến cuối chu kỳ phát triển mới tích hợp toàn bộ, giờ đây chúng ta có thể liên tục tích hợp các tính năng mới và kiểm thử chúng ngay trong quá trình phát triển.

Chúng ta có thể thiết lập một **CI/CD pipeline**. Các pipeline này thường bao gồm các thành phần riêng biệt sau:

* **Starting Trigger:** Hành động kích hoạt quy trình pipeline. Ví dụ: một thao tác push code vào một branch cụ thể.
* **Building Actions:** Các tác vụ được thực hiện để build cả dự án lẫn tính năng mới.
* **Testing Actions:** Các tác vụ kiểm thử dự án nhằm đảm bảo tính năng mới không gây xung đột với bất kỳ tính năng hiện có nào.
* **Deployment Actions:** Khi pipeline chạy thành công, các hành động deployment sẽ xác định bước tiếp theo cho bản build (ví dụ: đẩy bản build lên Testing Environment).
* **Delivery Actions:** Trọng tâm hiện mở rộng ra toàn bộ quá trình phân phối giải pháp, bao gồm cả các hành động như monitoring ứng dụng sau khi triển khai.

CI/CD pipeline cần hạ tầng build (**build-infrastructure**) để thực thi các tác vụ này, thường bao gồm **build orchestrators** và **agents**. Một build orchestrator sẽ điều phối các agents khác nhau thực hiện các hành động của pipeline theo yêu cầu.

> [!NOTE]
> Đây cũng là nơi tập trung phần lớn tính năng tự động hóa. Do đó, CI/CD pipeline thường là bề mặt tấn công (**attack surface**) lớn nhất và là nơi dễ phát sinh các lỗi cấu hình sai (**misconfigurations**) nhất.

> [!NOTE] ví dụ cụ thể
> Giả sử bạn phát triển tính năng **"Đổi mật khẩu"** cho một ứng dụng web bằng Java Spring Boot, sử dụng GitHub và GitHub Actions để quản lý pipeline:
> 
> **1. Starting Trigger (Kích hoạt)**
> Bạn hoàn thành code trên máy cá nhân và gõ lệnh:
> 
> ```bash
> git push origin main
> 
> ```
> 
> Thao tác đẩy code lên nhánh `main` tạo ra một sự kiện (event). GitHub bắt được sự kiện này và tự động kích hoạt workflow CI/CD được cấu hình sẵn trong file `.github/workflows/ci-cd.yml`.
> 
> **2. Hạ tầng tiếp nhận (Orchestrator & Agent)**
> 
> * **Build Orchestrator (GitHub Actions Engine):** Nhận diện trigger, đọc kịch bản workflow, xác định các bước cần chạy và điều phối tài nguyên.
> * **Build Agent (Ubuntu Runner):** Orchestrator khởi tạo một máy ảo Ubuntu sạch, kéo mã nguồn mới nhất từ repo về máy ảo này để chuẩn bị thực thi các lệnh.
> 
> **3. Building Actions (Biên dịch & Đóng gói)**
> Agent bắt đầu chạy các lệnh đóng gói ứng dụng:
> 
> * Chạy `mvn clean package -DskipTests` để tải các thư viện phụ thuộc (dependencies) từ kho lưu trữ và biên dịch code thành file `app.jar`.
> * Chạy lệnh `docker build -t my-app:v1.2 .` để đóng gói file `app.jar` đó thành một Docker image hoàn chỉnh.
> 
> **4. Testing Actions (Kiểm thử tự động - Cổng bảo vệ)**
> Trước khi cho phép xuất xưởng, Agent lần lượt chạy các công cụ kiểm tra:
> 
> * **Unit Test:** Chạy `mvn test` để kiểm tra logic (ví dụ: mật khẩu mới dưới 8 ký tự thì hàm phải trả về lỗi).
> * **Security Test (SAST):** Tích hợp công cụ SonarQube hoặc Snyk quét toàn bộ mã nguồn vừa push xem có hardcode database password hay có lỗ hổng SQL Injection nào không.
> * *Kịch bản lỗi:* Nếu 1 bài test logic bị fail hoặc Snyk phát hiện lộ secret key, pipeline lập tức **chặn lại (fail)**, gửi thông báo cảnh báo qua Telegram/Slack cho bạn và dừng hoàn toàn quá trình build.
> 
> **5. Deployment Actions (Triển khai)**
> Khi toàn bộ test đều pass (màu xanh):
> 
> * Agent tự động đẩy (push) Docker image `my-app:v1.2` lên kho chứa Docker Hub.
> * Agent kết nối qua SSH hoặc gọi API vào máy chủ (hoặc cụm Kubernetes) ở môi trường Thử nghiệm (Staging/UAT) để kéo image mới về và khởi chạy container thay thế phiên bản cũ.
> 
> **6. Delivery Actions (Bàn giao & Giám sát vận hành)**
> Sau khi container mới khởi động:
> 
> * Pipeline tự động bắn một request kiểm tra sức khỏe hệ thống: `curl [https://staging.example.com/actuator/health](https://staging.example.com/actuator/health)`.
> * Nếu phản hồi `200 OK`, pipeline thông báo triển khai thành công. Hệ thống giám sát (như Prometheus/Grafana) theo dõi tỷ lệ lỗi trong 10 phút đầu; nếu phát hiện mã lỗi `500 Internal Server Error` tăng đột biến, hệ thống sẽ kích hoạt lệnh tự động rollback (quay lui) về phiên bản `v1.1` trước đó để tránh gián đoạn dịch vụ.

## Common Tools

GitHub và GitLab đều cung cấp khả năng tích hợp CI/CD pipeline rất phổ biến. **GitHub cung cấp sẵn các build agents, trong khi GitLab cung cấp ứng dụng GitLab Runner để cài đặt lên máy chủ và biến nó thành một build agent**. Với các quy trình build phức tạp hơn, phần mềm điều phối như Jenkins thường được sử dụng.

## Case Study: Sự nhập nhằng giữa Dev và Prod

Một lỗi cấu hình phổ biến trong CI/CD pipeline là dùng chung **build agents** cho cả hai bản build Development (DEV) và Production (PROD). Điều này tạo ra một rủi ro lớn: phần lớn developer đều có quyền kích hoạt starting trigger cho một bản DEV build, nhưng không có quyền với PROD build.

Nếu tài khoản của một developer bị xâm phạm, kẻ tấn công có thể lợi dụng quyền này để tạo ra một bản DEV build độc hại nhằm chiếm quyền kiểm soát build agent. Rủi ro sẽ dừng lại ở đó nếu agent này chỉ chạy DEV. Tuy nhiên, vì agent dùng chung cho cả PROD, kẻ tấn công có thể duy trì sự hiện diện (**persist**) trên agent đó cho đến khi một PROD build được kích hoạt, từ đó chèn mã độc vào sản phẩm thực tế (production build) của ứng dụng.
![[Pasted image 20260922201156.png]]

# 6-environment
Hãy cùng đi sâu hơn vào phần **Môi trường (Environments)** trong pipeline. Phần lớn các pipeline đều bao gồm nhiều môi trường khác nhau. Mỗi môi trường phục vụ một mục đích cụ thể và mức độ bảo mật (security posture) của chúng cũng khác nhau:

| Môi trường                                                                   | Mô tả                                                                                                                                                                                                                                                                                                                                                                                       | Độ ổn định                        | Mức độ bảo mật                    | Chứa dữ liệu khách hàng? |     |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- | --------------------------------- | ------------------------ | --- |
| **DEV**<br><br>  <br>  <br><br>_(Development)_                               | Môi trường thử nghiệm cho developer. Đây là nơi kém ổn định nhất do code mới liên tục được đẩy lên và test. Mức độ bảo mật tại đây là yếu nhất, kiểm soát truy cập lỏng lẻo và lập trình viên thường có quyền truy cập trực tiếp vào hạ tầng. Khả năng DEV bị xâm nhập là rất cao, nhưng nếu phân tách (segregation) tốt thì mức độ ảnh hưởng sẽ ở mức thấp.                                | Không ổn định _(Unstable)_        | Yếu nhất _(Weakest)_              | Không                    |     |
| **UAT**<br><br>  <br>  <br><br>_(User Acceptance Testing)_                   | Dùng để kiểm thử ứng dụng hoặc các tính năng chọn lọc trước khi đưa lên production, bao gồm cả unit test và security test để đảm bảo chức năng hoạt động đúng. Môi trường này ổn định hơn DEV nhưng đôi khi vẫn khá chập chờn. Các biện pháp gia cố (hardening) đã bắt đầu được áp dụng nhưng chưa chặt chẽ bằng PreProd hay PROD.                                                          | Tương đối ổn định _(Semi-Stable)_ | Yếu thứ hai _(Second Weakest)_    | Không                    |     |
| **PreProd**<br><br>  <br>  <br><br>_(Pre-Production)_                        | Dùng để mô phỏng môi trường production nhưng không chứa dữ liệu thật của khách hàng. Môi trường này được giữ ổn định để chạy các bài test cuối cùng trước khi release. Về mặt kỹ thuật, mức độ bảo mật của PreProd phải tương đương (mirror) với PROD, dù thực tế không phải lúc nào cũng đạt được.                                                                                         | Ổn định _(Stable)_                | Mạnh thứ hai _(Second Strongest)_ | Không                    |     |
| **PROD**<br><br>  <br>  <br><br>_(Production)_                               | Môi trường nhạy cảm nhất, đang trực tiếp phục vụ người dùng và khách hàng thật. Cần duy trì tính ổn định tuyệt đối và không được cập nhật nếu thiếu quy trình quản lý thay đổi (change management). Mức độ bảo mật tại đây là cao nhất; chỉ một số ít nhân sự/dịch vụ được cấp quyền thao tác. Hệ thống cũng phải được gia cố tối đa để phòng ngừa cả các cuộc tấn công từ bên ngoài.       | Ổn định _(Stable)_                | Mạnh nhất _(Strongest)_           | Có                       |     |
| **DR / HA**<br><br>  <br>  <br><br>_(Disaster Recovery / High Availability)_ | Tùy vào độ trọng yếu của hệ thống mà sẽ có môi trường DR hoặc HA. Nếu quá trình chuyển đổi dự phòng diễn ra tức thì, đó là HA (thường dùng cho Online Banking để tránh bị phạt nặng do downtime). Nếu chấp nhận gián đoạn trong thời gian ngắn, đó là DR (dùng để khôi phục khi PROD gặp thảm họa). Cả hai môi trường này đều phải là bản sao chuẩn xác của PROD từ độ ổn định đến bảo mật. | Ổn định _(Stable)_                | Mạnh nhất _(Strongest)_           | Có                       |     |

> [!NOTE]
> **High Availability (HA)** tập trung vào việc giữ cho hệ thống hoạt động liên tục không gián đoạn (uptime 99.99%), trong khi **Disaster Recovery (DR)** tập trung vào việc hồi sinh lại hệ thống sau khi toàn bộ hạ tầng chính bị phá hủy hoàn toàn.
> 
> 
> ### 1. Ví dụ về High Availability (HA) - Cổng thanh toán ngân hàng
> 
> Ngân hàng yêu cầu dịch vụ chuyển tiền và quẹt thẻ phải hoạt động 24/7/365, không được phép sập dù chỉ 1 phút.
> 
> * **Thiết lập:** Ngân hàng thuê 2 Trung tâm dữ liệu (Data Center - DC): DC1 đặt tại Hà Nội và DC2 đặt tại TP.HCM. Cả 2 DC đều chạy song song (mô hình Active - Active), dữ liệu cơ sở dữ liệu được đồng bộ liên tục theo thời gian thực (real-time).
> * **Kịch bản sự cố:** Một vụ cháy lớn xảy ra tại trạm biến áp gần DC1 Hà Nội khiến toàn bộ nguồn điện và đường cáp quang của DC1 bị ngắt hoàn toàn.
> * **Cách HA xử lý:**
> 1. Bộ cân bằng tải toàn cầu (Global Load Balancer) phát hiện DC1 không còn phản hồi qua cơ chế kiểm tra sức khỏe (Health Check).
> 2. Lưu lượng truy cập (traffic) ngay lập tức được tự động điều hướng 100% sang DC2 TP.HCM trong vòng **vài mili-giây** (Auto-Failover).
> 
> 
> * **Kết quả:** Người dùng đang quẹt thẻ mua hàng tại siêu thị hoàn toàn không nhận ra sự cố. Giao dịch vẫn thành công bình thường, không mất dữ liệu, không có thời gian chết (Zero Downtime).
> 
> 
> 
> ### 2. Ví dụ về Disaster Recovery (DR) - Hệ thống ERP của doanh nghiệp sản xuất
> 
> Doanh nghiệp có nhà máy và toàn bộ hệ thống máy chủ cơ sở dữ liệu nội bộ (On-Premises) đặt tại khu công nghiệp.
> 
> * **Thiết lập:** Doanh nghiệp duy trì một môi trường dự phòng (Cold/Warm Standby) trên đám mây (ví dụ: AWS). Cứ đúng 00:00 mỗi đêm, bản sao lưu toàn bộ dữ liệu (Snapshot/Backup) từ máy chủ nhà máy sẽ được mã hóa và tải lên AWS.
> * **Kịch bản sự cố:** Khu công nghiệp gặp lũ lụt nặng làm ngập toàn bộ phòng server, thiết bị phần cứng chập cháy và hư hỏng vĩnh viễn. Trong tình huống này, HA không giải quyết được vì hạ tầng vật lý chính đã bị xóa sổ.
> * **Cách DR xử lý:**
> 1. Đội ngũ IT kích hoạt **Quy trình ứng phó thảm họa (Disaster Recovery Plan)**.
> 2. Khởi tạo một cụm máy chủ ảo mới trên AWS từ các mẫu cấu hình có sẵn (Infrastructure as Code).
> 3. Bung bản sao lưu dữ liệu của đêm hôm trước vào máy chủ mới trên AWS.
> 4. Trỏ lại tên miền (DNS) để nhân viên công ty truy cập vào hệ thống ERP mới trên đám mây.
> 
> 
> * **Kết quả:**
> * Sau **4 tiếng** thực hiện, hệ thống hoạt động trở lại (chỉ số **RTO - Recovery Time Objective** = 4 giờ).
> * Doanh nghiệp chấp nhận mất các chứng từ nhập/xuất kho được tạo từ sáng đến thời điểm ngập lụt vì chưa đến khung giờ backup đêm (chỉ số **RPO - Recovery Point Objective** = dữ liệu phát sinh trong ngày).
> 
> 
> 
> 
> 
> ### Bảng đối chiếu nhanh HA và DR
> 


> | Đặc điểm                | High Availability (HA)                                 | Disaster Recovery (DR)                                                             |     |
> | ----------------------- | ------------------------------------------------------ | ---------------------------------------------------------------------------------- | --- |
> | **Mục đích**            | Ngăn chặn hệ thống bị sập khi lỗi linh kiện/máy chủ.   | Cứu vãn dữ liệu và khôi phục hoạt động sau thảm họa lớn.                           |     |
> | **Phạm vi bảo vệ**      | Lỗi phần cứng, sập nguồn, đứt cáp cục bộ.              | Động đất, lũ lụt, cháy nổ toàn bộ Data Center, mã độc Ransomware xóa sạch dữ liệu. |     |
> | **Thời gian gián đoạn** | Gần như bằng 0 (tức thì).                              | Có thời gian chờ (vài chục phút đến vài giờ/ngày tùy thỏa thuận).                  |     |
> | **Dữ liệu thất thoát**  | Không mất (hoặc chỉ vài giao dịch đang bay trên mạng). | Chấp nhận mất một phần dữ liệu tính từ thời điểm bản backup gần nhất được tạo.     |     |
> | **Chi phí**             | Rất đắt (hạ tầng chạy song song 24/7).                 | Tiết kiệm hơn (tận dụng backup đám mây, chỉ bật máy chủ dự phòng khi có biến cố).  |     |

## Các môi trường đáng chú ý khác

### Môi trường Blue và Green (Blue/Green Environments)

Được sử dụng trong chiến lược triển khai Blue/Green khi đưa bản cập nhật lên PROD. Thay vì chỉ có một máy chủ PROD duy nhất, hệ thống duy trì hai môi trường song song: **Blue** chạy phiên bản hiện tại, còn **Green** chạy phiên bản mới hơn.

Thông qua một proxy hoặc router, toàn bộ lưu lượng truy cập (traffic) sẽ được chuyển hướng sang Green khi đội ngũ đã sẵn sàng. Môi trường Blue vẫn được giữ lại một thời gian; nếu có sự cố phát sinh với bản mới, traffic lập tức được định tuyến ngược trở lại Blue. Đây được xem như một bản backup High-Availability của PROD giúp quá trình rollback diễn ra tức thì mà không cần can thiệp phức tạp vào hệ thống chính.
### Môi trường Canary (Canary Environments)

Tương tự như Blue/Green, mục tiêu của Canary là giúp quá trình nâng cấp PROD diễn ra êm xuôi và hạn chế rủi ro downtime. Người dùng sẽ được chuyển dần từng phần sang môi trường mới: ban đầu chỉ điều hướng khoảng 10% lượng traffic; nếu hệ thống vận hành ổn định, tỷ lệ này sẽ tăng dần (20%, 30%...) cho đến khi đạt 100%.

## Common Tools

Cơ sở hạ tầng môi trường hiện đại đã chuyển dịch mạnh mẽ nhờ công nghệ ảo hóa (virtualisation) và container hóa (containerisation):

- Thay vì thiết lập trực tiếp trên máy chủ vật lý, môi trường có thể được tự động tạo lập dưới dạng máy ảo thông qua các công cụ như **Vagrant** hoặc **Terraform**.
  
- Có thể loại bỏ hoàn toàn các máy chủ truyền thống để chuyển sang container với **Docker** hoặc cụm pod với **Kubernetes**.
     
- Áp dụng quy trình **Infrastructure as Code (IaC)** để định nghĩa, khởi tạo và quản lý toàn bộ vòng đời môi trường bằng mã lệnh.
Dưới đây là một ví dụ thực tế minh họa sự chuyển dịch từ cách làm truyền thống sang công nghệ hiện đại khi xây dựng hệ thống cho một ứng dụng web (gồm Backend Spring Boot và Database PostgreSQL):

> [!NOTE] ví dụ
> 
> ### 1. Máy ảo tự động (Vagrant & Terraform) thay vì cài đặt thủ công
> 
> * **Cách làm cũ (Thủ công):** Quản trị viên phải mua server vật lý hoặc tự mở phần mềm tạo máy ảo, gắn file ISO cài Ubuntu, cấu hình IP, cài Java, cấu hình firewall từng bước bằng tay. Khi cần thêm 5 máy chủ thử nghiệm, họ phải lặp lại thao tác này 5 lần, rất tốn thời gian và dễ nhầm lẫn thông số.
> * **Hiện đại với Vagrant (cho máy cá nhân lập trình viên):**
> Lập trình viên chỉ cần một file cấu hình duy nhất tên `Vagrantfile`:
> ```ruby
> Vagrant.configure("2") do |config|
>   config.vm.box = "ubuntu/22.04"
>   config.vm.network "forwarded_port", guest: 8080, host: 8080
>   config.vm.provision "shell", inline: <<-SHELL
>     apt-get update
>     apt-get install -y openjdk-17-jdk postgresql
>   SHELL
> end
> 
> ```
> 
> 
> Chỉ cần gõ lệnh `vagrant up`, toàn bộ máy ảo Ubuntu với đầy đủ Java và PostgreSQL sẽ tự động tải về, khởi động và sẵn sàng hoạt động trong vài phút.
> * **Hiện đại với Terraform (cho hạ tầng Cloud như AWS/Azure):**
> Khai báo tài nguyên máy chủ bằng file code `main.tf`:
> ```hcl
> resource "aws_instance" "web_server" {
>   ami           = "ami-0c55b159cbfafe1f0" # Ubuntu Server
>   instance_type = "t3.micro"
>   tags = {
>     Name = "Dev-Environment-Server"
>   }
> }
> 
> ```
> 
> 
> Chỉ cần chạy lệnh `terraform apply`, Terraform sẽ tự động gọi API của AWS để tạo đúng máy chủ đó mà không cần ai phải đăng nhập vào giao diện web AWS Console để click chuột.
> 
>
> 
> ### 2. Container (Docker) & Điều phối (Kubernetes) thay thế máy ảo cồng kềnh
> 
> * **Docker (Đóng gói môi trường):**
> Trước đây, máy ảo đòi hỏi phải cài nguyên một hệ điều hành riêng biệt (ngốn vài GB RAM và ổ cứng chỉ để chạy hệ điều hành khách). Với Docker, ứng dụng chia sẻ chung nhân kernel của máy chủ mẹ:
> * Lập trình viên viết một file `Dockerfile` quy định: lấy base image Java 17, copy file `app.jar` vào, mở cổng 8080.
> * Chỉ cần chạy `docker run -d -p 8080:8080 my-backend:1.0`, ứng dụng lập tức khởi chạy độc lập trong vài giây, chiếm rất ít tài nguyên và đảm bảo chạy trên máy dev hay đưa lên server đều hoàn toàn giống nhau 100%.
> 
> 
> * **Kubernetes (Quản trị cụm Pod quy mô lớn):**
> Thay vì phải theo dõi từng container bằng tay:
> * Bạn khai báo: *"Tôi muốn ứng dụng backend này luôn có 3 bản sao (replicas) chạy cùng lúc."*
> * Nếu một bản sao bị tràn bộ nhớ (Out-Of-Memory) và crash, Kubernetes sẽ tự động phát hiện và khởi tạo một container/pod mới thay thế ngay lập tức (Self-healing).
> * Vào dịp khuyến mãi lượng truy cập tăng vọt, Kubernetes có thể tự động tăng số lượng container từ 3 lên 10 bản sao (Auto-scaling) để chia tải.
> 
> 
> 
>
> 
> ### 3. Quy trình Infrastructure as Code (IaC) - Quản lý hạ tầng bằng code
> 
> Thay vì để kỹ sư mạng hay sysadmin tự ý sửa đổi server trong âm thầm, toàn bộ kiến trúc hạ tầng được chuyển hóa thành file mã nguồn và lưu trữ trong Git.
> 
> * **Kiểm soát thay đổi (Audit Log):** Nếu cần nâng RAM server từ 4GB lên 8GB, kỹ sư không remote vào server để sửa. Thay vào đó, họ sửa file Terraform từ `memory = 4096` thành `memory = 8192`, sau đó tạo một **Pull Request (PR)** trên GitHub/GitLab.
> * **Bình duyệt (Review):** Tech Lead hoặc Trưởng nhóm bảo mật vào kiểm tra, thấy hợp lý thì duyệt (Approve) và merge code.
> * **Tự động áp dụng:** Ngay sau khi merge, pipeline CI/CD chạy lệnh cập nhật hạ tầng tự động. Lịch sử commit trên Git sẽ lưu lại chính xác: *Ai là người tăng RAM, sửa vào lúc mấy giờ và vì lý do gì.*

## Security Considerations

Môi trường càng tiến gần đến PROD thì yêu cầu bảo mật càng khắt khe. Cơ sở hạ tầng nền tảng cũng chính là một phần của bề mặt tấn công (**attack surface**); bất kỳ lỗ hổng hạ tầng nào cũng có thể giúp kẻ tấn công chiếm quyền kiểm soát máy chủ lưu trữ (host) và ứng dụng. Quá trình gia cố bảo mật (**hardening**) cho hạ tầng
- Gỡ bỏ các dịch vụ không cần thiết (unnecessary services).
- Cập nhật bản vá định kỳ cho hệ điều hành và các phần mềm phụ trợ.
- Sử dụng tường lửa (firewall) để chặn triệt để các port không dùng đến.
## Case Study: Cơ chế Bypass của Developer lọt vào PROD

Một sự cố phổ biến giữa các môi trường là những đoạn mã cấu hình thử nghiệm vốn chỉ nên tồn tại ở DEV lại vô tình bị đẩy lên các môi trường cao hơn. Các lập trình viên thường tạo sẵn các cơ chế bỏ qua (developer bypass) ở môi trường DEV nhằm đẩy nhanh tiến độ kiểm thử, chẳng hạn như:
- Bỏ qua xác thực đa yếu tố (MFA).
- Vượt qua mã CAPTCHA.
- Bỏ qua các bước xác minh khi đặt lại mật khẩu hoặc cổng đăng nhập.

Một ví dụ phổ biến là hardcode sẵn một mã OTP cố định (ví dụ: `123456`) để hệ thống luôn chấp nhận đăng nhập thành công mà không cần kiểm tra OTP gửi về điện thoại. Nếu thiếu khâu làm sạch (sanitisation) kỹ lưỡng trước khi đưa ứng dụng qua các cổng kiểm soát, đoạn mã này có thể trôi thẳng lên PROD. Kẻ tấn công có thể lợi dụng chính cửa hậu này để vượt qua lớp bảo vệ MFA và chiếm đoạt tài khoản người dùng thực tế.

Đây là lý do môi trường phải được phân tách rạch ròi, và các **security gates** bắt buộc phải được kích hoạt song song với quality gates để đảm bảo ứng dụng hoàn toàn sạch sẽ trước khi được phép chuyển môi trường.

> [!NOTE]
> sẽ có những cái đoạn mã giúp dev debug nhanh, vượt qua các cơ chế rườm rả, hoạt động ở môi trường dev.Nói chung là phải locj thật sạch khi dưa lên môi trường production

