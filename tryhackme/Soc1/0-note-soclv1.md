
# 1-Intro to blue team
## 1.1 SOC role in blue team
==> https://tryhackme.com/room/socroleinblueteam
- [ ] đầu tiên là nói về các role trong blue team
 - [ ] soc lv1->2->3-> CIRT
 - [ ] MSSP -> nhà cung cấp dịch vụ an ninh mạng
	 - [ ] manage security services provider
	 - [ ] đại khái là SOC nhưng hoạt động theo kiểu dịch vụ; khác với SOC nội bộ thường êm đềm hơn
- [ ] các role khác cần để tâm
	- [ ]  CERT lead 
	- [ ] GRC auditor
	- [ ] Threat researcher
## 1.2 Human as Attack Vectors
==>https://tryhackme.com/room/humansattackvectors
- [ ] yếu tố của con người trong cyber secutity?
	- [ ] thường là yếu tố nguy hiểm nhất, hệ thống thì dễ fix nhưng con người thì ko
		- [ ] --> con người có thể cung cấp access( quyền truy cập) một cách dễ dàng mà ko cần nhiều kỹ thuật tấn công cao siêu
	- [ ] => social engineering -> phần này mình cũng đã biết rồi.bài đề cập tới các kỹ thuật như fisshing , malware download, hay deepfakes.Hay đề cập tới thuật ngữ impersonation: mạo danh

- [ ] giải pháp khắc phục đơn giản là "phòng ngừa"
	- [ ] đào tạo nhân viên, antivirus, dùng các took block phising

## 1.3 System attack vector
==> https://tryhackme.com/room/systemsattackvectors
- [ ] luôn update các lỗ hổng mới--> xem liệu hệ thống có include cái lỗ hổng đó ko--> đợi bản patch
- [ ] vài note về misconfiguration
	- [ ] cái này mình đã biết được khi học về attack, như rate limit, hay robot.txt, hay các lỗi như giữ nguyên các hàm giúp test trong phase deverloper và đưa luôn cái đó lên production
	- [ ] khắc phục misconfiguration:
		- [ ]  pentest ; vul scans , configuration audit-> tức review thủ công sao cho nó theo tiêu chuẩn như  [CIS benchmark](https://www.cisecurity.org/cis-benchmarks)

# 2-SOC Team Internals
## 2.1 SOC l1 alert triage
==> https://tryhackme.com/room/socl1alerttriage
-đầu tiên làm quen với khái niệm "Alert"
-sau đó tiến hành lên SOC simulator để làm
- [ ] từ **Event** tới  **Alert**
	- [ ] ban đầu có 1 evern--> được ghi log lại--> log này được đi qua các hệ thống security operation như SIEM hay EDR--> sau đó hệ thống sẽ Alert,SOC bắt đầu phân tích

- [ ] các hệ thống Alert management:
	- [ ] SIEM -> splunk es; elastic SIEM
		- --> thu thập log+ tạo alert+ quản lý alert
	- [ ] EDR ; NDR => MS Defender, CrowdStrike
		- -> giám sát enpoint/network=> sau đó đẩy alert về SIEM/SOAR
	- [ ] SOAR system->Splunk SOAR, Cortex SOAR
		- -> Tự động hóa — gom alert từ nhiều nguồn, tự xử lý alert lặp lại/thông thường
	- [ ] ITSM system -> Jira , theHive
		- ->Quản lý alert như ticket, theo dõi trạng thái (New → In Progress → Resolved)

- [ ] khái niệm **Triage**=> Phân loại alert nào nguy hiểm,...
![[Pasted image 20260926164500.png]]


**-8 thuộc tính chính cần hiểu khi xem alert:** 

| #   | Thuộc tính            | Ý nghĩa                                                                       |
| --- | --------------------- | ----------------------------------------------------------------------------- |
| 1   | **Alert Time**        | Thời gian tạo alert (thường muộn hơn vài phút so với sự kiện thật)            |
| 2   | **Alert Name**        | Tên tóm tắt sự việc (VD: Đăng nhập bất thường, Bruteforce RDP...)             |
| 3   | **Alert Severity**    | Mức độ nghiêm trọng: 🟢 Low → 🟡 Medium → 🟠 High → 🔴 Critical               |
| 4   | **Alert Status**      | Trạng thái xử lý: New → In Progress → Closed                                  |
| 5   | **Alert Verdict**     | Phân loại: 🔴 **True Positive** (threat thật) / 🟢 **False Positive** (noise) |
| 6   | **Alert Assignee**    | Analyst được giao phụ trách alert (chịu trách nhiệm xử lý)                    |
| 7   | **Alert Description** | Mô tả gồm 3 phần: logic rule, vì sao là dấu hiệu tấn công, cách xử lý         |
| 8   | **Alert Fields**      | Giá trị kích hoạt alert: hostname, commandline, comments...                   |

**-Alert Prioritisation** ==> quá trình ưu tiên cảnh báo khi mà trong thực tế có rất nhiều cảnh báo đổ về từ SIEM
-> ta sẽ lọc các cảnh báo theo các tiêu chí như dựa theo **assignee-->severity-->time**

| Bước | Tiêu chí         | Nguyên tắc                          |
| :--- | :--------------- | :---------------------------------- |
| 1    | Lọc              | Chỉ lấy cảnh báo mới, chưa ai xử lý |
| 2    | Mức nghiêm trọng | Critical trước, Low sau             |
| 3    | Thời gian        | Cũ trước, mới sau                   |

Trong thực tế, các SOC thường **tự định nghĩa quy tắc ưu tiên riêng** và tự động hóa chúng bằng cách cấu hình logic sắp xếp cảnh báo ngay trong SIEM hoặc EDR — phần trên chỉ là phương pháp đơn giản và phổ biến nhất.

**-Các quy trình cơ bản khi xử lý Alert**
- [ ] assign --> in progress--> nắm thông tin tổng quan-> investigations
	- [ ] investigation
	- [ ] workbook -> tài liệu hướng dẫn điều tra từng loại cụ thể 
- [ ] sau khi investigation xong thì đưa ra final action
![[Pasted image 20260926173924.png]]
=> ở lab này mình đã được handon vào cách để phân loại các cảnh báo.Nó chỉ là một lab đơn giản thôi!
![[Pasted image 20260926175422.png]]

# 2.2 