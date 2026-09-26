**==>** [[note-kubernet]]


# 2 - cluster Architecture

Vậy là chúng ta đã nắm được Kubernetes là gì, tại sao nó lại cần thiết và mang lại những lợi ích gì cho DevSecOps. Giờ là lúc đi sâu hơn vào bản chất — cùng "mổ xẻ" xem bên dưới hệ thống (under the hood), Kubernetes thực sự hoạt động ra sao. Đúng vậy, đã đến phần kiến trúc rồi! Chúng ta sẽ lần lượt điểm qua từng thành phần cốt lõi cấu thành nên Kubernetes, sau đó ráp nối tất cả lại để thấy được bức tranh toàn cảnh về cách chúng liên kết với nhau. Bắt đầu thôi!

### Kubernetes Pod

Pod là đơn vị tính toán nhỏ nhất mà bạn có thể tạo và quản lý trong Kubernetes. Khi làm việc với DevSecOps trên nền tảng Kubernetes, bạn sẽ bắt gặp thuật ngữ này liên tục. Hãy hình dung Pod như một nhóm gồm một hoặc nhiều container. Các container trong cùng một Pod sẽ chia sẻ chung tài nguyên lưu trữ (storage) và mạng (network). Nhờ đó, chúng có thể giao tiếp với nhau cực kỳ dễ dàng — hệt như đang chạy chung trên một máy chủ vật lý — trong khi vẫn duy trì được mức độ cô lập nhất định. Trong Kubernetes, Pod đóng vai trò là đơn vị sao chép (unit of replication): khi ứng dụng cần mở rộng quy mô (scale up), thứ bạn tăng lên chính là số lượng Pod đang chạy.

![pod diagram|700](https://cdn-images.tryhackme.com/user-uploads/6228f0d4ca8e57005149c3e3/room-content/98b54b6b8427fae556be0cdd516b9e36.svg)

> [!NOTE]
> pod gồm các container.Trong b 1 pod các container cso thể giao tiếp vs nhau

### Kubernetes Nodes
Các workload (ứng dụng) trong Kubernetes chạy bên trong container, container được đặt trong Pod, và các Pod này lại chạy trên các Node. Khi bàn về kiến trúc Node, có hai loại chính bạn cần phân biệt: **Control Plane** (trước đây thường gọi là "master node") và các **Worker Node**. Mỗi loại đều sở hữu kiến trúc và các thành phần riêng biệt, chúng ta sẽ tìm hiểu chi tiết ngay sau đây. Một Node có thể là máy ảo (VM) hoặc máy chủ vật lý. Hãy tư duy đơn giản thế này: **nếu ứng dụng chạy trong container đặt trong Pod, thì Node chính là nơi cung cấp toàn bộ môi trường và dịch vụ cần thiết để những Pod đó có thể vận hành.**

> [!NOTE] Title
> pod chạy trong node.
> có 2 loại node là control Plane và worker node

### Kubernetes Cluster
Ở tầng cao nhất, chúng ta có Kubernetes Cluster (cụm). Hiểu một cách ngắn gọn, một Cluster đơn giản là một tập hợp các Node kết nối lại với nhau.

### Kubernetes Control Plane (Bộ điều khiển trung tâm)

Control Plane đóng vai trò là bộ não điều hành, quản lý toàn bộ các Worker Node và các Pod trong Cluster. Nhiệm vụ này được thực hiện thông qua nhiều thành phần chuyên biệt. Hãy cùng xem từng thành phần đảm nhận vai trò gì trước khi ghép chúng lại trên sơ đồ kiến trúc tổng thể.

**Kube-apiserver** 

API server là giao diện tiền sảnh (front-end) của Control Plane, chịu trách nhiệm phơi bày (expose) Kubernetes API ra bên ngoài. Kube-apiserver có khả năng mở rộng quy mô (scale) theo chiều ngang — nghĩa là bạn có thể tạo nhiều bản thể (instance) để cân bằng tải lưu lượng truy cập.

> [!NOTE]
> đây là api của controlplane?

**Etcd** 

Etcd là một kho lưu trữ dạng key/value cực kỳ ổn định và nhất quán (highly available & consistent), đóng vai trò lưu toàn bộ dữ liệu cấu hình và trạng thái hiện tại (current state) của cả Cluster. **Bất kỳ thay đổi nào diễn ra trong cụm — ví dụ như có một Pod mới được dựng lên — đều được ghi nhận lại trong etcd.** Các thành phần khác của Control Plane đều dựa vào etcd như một "chân lý nguồn" để tra cứu thông tin (như kiểm tra lượng tài nguyên còn trống).

> [!NOTE]
> etcd giúp ghi lại và lưu các cấu hình hiện tại cả cụm cluster

**Kube-scheduler** 

Kube-scheduler là thành phần liên tục giám sát trạng thái của Cluster. Nhiệm vụ của nó là phát hiện các Pod mới được tạo nhưng chưa được phân công cho Node nào, từ đó quyết định xem nên điều phối Pod đó về Node nào phù hợp nhất. Quyết định phân bổ dựa trên các tiêu chí cụ thể: yêu cầu tài nguyên của ứng dụng so với tài nguyên khả dụng trên các Worker Node, các ràng buộc về phần cứng/chính sách, v.v.

> [!NOTE]
> kube-scheduler đúng như tên của nó--> vai trò là giám sát các node chưa dược dùng, và điều phối lưu lượng về node đó sao cho phù hợp

**Kube-controller-manager** 
Thành phần này chịu trách nhiệm chạy các tiến trình điều khiển (controller processes). Có rất nhiều loại controller khác nhau, ví dụ như **node controller** — chuyên theo dõi và phát hiện khi có Node gặp sự cố ngắt kết nối. *Lúc đó, controller manager sẽ liên lạc với scheduler để lên lịch điều phối khởi tạo các Pod/Node mới nhằm bù đắp lại.*

> [!NOTE]
> chịu trách nhiệm cho các tiến trình controller
> --> do có nhiều loại controler khác nhau, nên cần thằng này để quản lý

**Cloud-controller-manager** 

<u>Thành phần này cho phép Kubernetes kết nối và tích hợp trực tiếp với API của nhà cung cấp dịch vụ đám mây </u>(AWS, GCP, Azure...). Mục đích là tách biệt rạch ròi giữa các thành phần giao tiếp nội bộ trong cụm với các thành phần tương tác ra các dịch vụ cloud bên ngoài (như quản lý Load Balancer, Storage Volume trên cloud). Điều này cũng giúp các nhà cung cấp cloud dễ dàng nâng cấp tính năng mà không ảnh hưởng tới core Kubernetes.

![Kubernetes uses a Hub and Spoke API pattern|700](https://cdn-images.tryhackme.com/user-uploads/6228f0d4ca8e57005149c3e3/room-content/84f898f84e80373f4a4a34e1f658d914.svg)

> [!NOTE]
> Cái này có lẽ là cái giúp kubernet nó kết nối với API của cloud service
> -vẫn chưa hiểu thực tế nó ntn?

### Kubernetes Worker Node (Các node thực thi)

Worker Node là những máy trực tiếp thực thi và duy trì sự sống cho các Pod. Trên mỗi Worker Node đều có các thành phần cốt lõi sau:

**Kubelet** 
Kubelet là một tiến trình đại diện (agent) chạy trên từng Node trong Cluster, đảm bảo các container bên trong Pod đang chạy đúng như kỳ vọng. Kubelet tiếp nhận bản mô tả chi tiết của Pod (pod specification / PodSpec) và kiểm tra xem các container được khai báo có đang hoạt động khỏe mạnh (healthy) hay không. Nó thực thi các chỉ thị được giao từ controller manager — chẳng hạn như kéo image và khởi chạy container trong Pod.

**Kube-proxy** 

Kube-proxy quản lý toàn bộ việc định tuyến và giao tiếp mạng nội bộ trong Cluster. Nó thiết lập các quy tắc mạng (networking rules) để lưu lượng truy cập có thể được chuyển hướng chính xác đến Pod (dù là traffic đến từ bên trong hay bên ngoài cụm). Trên thực tế, traffic thường không gọi trực tiếp vào Pod mà sẽ đi qua một thực thể gọi là Service (đại diện cho một nhóm Pod), sau đó kube-proxy sẽ điều hướng request đến một trong các Pod thích hợp. Chúng ta sẽ tìm hiểu sâu hơn về Service ở phần tiếp theo!

**Container runtime**

Pod cần có container chạy bên trong, và để làm được điều đó thì mỗi Node bắt buộc phải cài đặt một Container Runtime. Trong module này, chúng ta đã làm quen với Docker — cái tên phổ biến nhất. Tuy nhiên, có nhiều giải pháp thay thế khác chuẩn CRI (Container Runtime Interface) như containerd, CRI-O, runC hoặc rkt.

![Container Runtime Diagram](https://cdn-images.tryhackme.com/user-uploads/6228f0d4ca8e57005149c3e3/room-content/6228f0d4ca8e57005149c3e3-1770976373364.svg)

### Cách các thành phần giao tiếp với nhau (Communication Between Components)

Chúng ta vừa điểm qua khá nhiều thành phần rồi. Giờ hãy lùi lại một bước để quan sát cách tất cả những mảnh ghép rời rạc đó phối hợp nhịp nhàng với nhau tạo nên kiến trúc Kubernetes. Tóm lại: một Kubernetes Cluster gồm nhiều Node; Kubernetes chạy ứng dụng bằng cách đặt các container vào trong Pod, và đưa các Pod đó lên chạy trên các Node. Hãy xem sơ đồ tổng quan bên dưới để thấy cách toàn bộ hệ thống liên kết:

![Cluster Diagram|700](https://cdn-images.tryhackme.com/user-uploads/6228f0d4ca8e57005149c3e3/room-content/6228f0d4ca8e57005149c3e3-1770976740047.svg)

# 3 - Toàn cảnh hệ sinh thái Kubernetes (Kubernetes Landscape)

### Bức tranh tổng quan (The Lay of the Land)

Chúng ta vừa tìm hiểu về kiến trúc và cách thức vận hành hậu trường của Kubernetes! Bây giờ là lúc bước vào thực tế công việc: Với tư cách là một kỹ sư DevSecOps, hằng ngày bạn sẽ thao tác và làm việc với những đối tượng (object/resource) nào? Hãy cùng điểm qua những khái niệm phổ biến nhất trong hệ sinh thái Kubernetes.

**Namespaces**

Trong Kubernetes, Namespace được dùng để phân vùng và cô lập các nhóm tài nguyên bên trong cùng một Cluster. Chẳng hạn, bạn muốn nhóm các tài nguyên thuộc cùng một dịch vụ lại với nhau, hoặc nếu dùng chung một cụm cho nhiều dự án/khách hàng (multi-tenant), bạn có thể tách biệt tài nguyên theo từng tenant. Tên của tài nguyên bắt buộc phải là duy nhất trong cùng một Namespace, nhưng bạn hoàn toàn có thể dùng lại tên đó ở các Namespace khác nhau mà không sợ xung đột.

**ReplicaSet**

Đúng như tên gọi của nó, ReplicaSet trong Kubernetes có nhiệm vụ duy trì một số lượng Pod bản sao (replica pods) cố định và đảm bảo luôn có đủ số lượng bản sao giống hệt nhau sẵn sàng hoạt động (rất hữu ích khi cần chia tải công việc cho nhiều Pod). Thông thường, người ta hiếm khi khai báo trực tiếp ReplicaSet (và cũng ít khi tạo Pod đơn lẻ trực tiếp), mà thay vào đó sẽ quản lý chúng thông qua Deployment — khái niệm tiếp theo của chúng ta.

**Deployments** 

Deployment trong Kubernetes được sử dụng để định nghĩa "trạng thái mong muốn" (desired state) của ứng dụng. Khi bạn định nghĩa xong trạng thái này, Deployment Controller (một trong các tiến trình điều khiển) sẽ tự động đồng bộ hóa trạng thái thực tế của hệ thống sao cho khớp với trạng thái mong muốn. Deployment cho phép cập nhật Pod và ReplicaSet theo cách khai báo (declarative updates). Nói cách khác, bạn chỉ cần mô tả: "Tôi muốn có một Deployment tên là test-nginx-deployment với một ReplicaSet gồm 3 Pod chạy nginx". Sau khi áp dụng cấu hình, ReplicaSet sẽ âm thầm tự tạo và duy trì 3 Pod đó dưới nền.

**StatefulSets** 

Để hiểu được StatefulSet, trước tiên bạn cần phân biệt rõ giữa ứng dụng có lưu trạng thái (stateful) và ứng dụng không lưu trạng thái (stateless):
- **Ứng dụng Stateful:** Lưu trữ và ghi nhận dữ liệu người dùng, cho phép hệ thống quay lại trạng thái trước đó. Ví dụ: Bạn đang mở một ứng dụng email và đọc 3 lá thư, nhưng phiên làm việc đột ngột bị gián đoạn. Khi mở lại, trạng thái vẫn được bảo toàn và 3 lá thư đó vẫn hiển thị là đã đọc.
- **Ứng dụng Stateless:** Không lưu giữ thông tin nào về các tương tác trước đó của người dùng. Ví dụ: Bạn dùng công cụ tìm kiếm gõ một câu hỏi. Nếu phiên làm việc bị ngắt, bạn chỉ việc gõ lại câu hỏi đó từ đầu mà không cần phụ thuộc vào bất kỳ dữ liệu phiên nào trước đó.

Đối với các ứng dụng stateless (như ví dụ công cụ tìm kiếm), Deployment là lựa chọn hoàn hảo để quản lý các Pod bản sao. Do đặc tính không giữ trạng thái, các Pod có thể được đặt tên ngẫu nhiên, và khi cần giảm tải thì xóa ngẫu nhiên bất kỳ Pod nào cũng không làm mất dữ liệu.

Thế nhưng với các ứng dụng stateful (như cơ sở dữ liệu hay ứng dụng email), mọi chuyện hoàn toàn khác. Ứng dụng cần truy cập và cập nhật trạng thái liên tục. Hãy tưởng tượng dữ liệu trạng thái này đang được lưu trên cơ sở dữ liệu chạy trên 3 Pod (tức DB được nhân bản 3 lần). Điều gì sẽ xảy ra nếu một Pod cập nhật dữ liệu mới còn 2 Pod kia thì không? Dữ liệu sẽ lập tức bị lệch pha (desync). Đó chính là lý do StatefulSet ra đời và được dùng thay thế cho Deployment khi cần triển khai ứng dụng stateful trên Kubernetes.

StatefulSet cho phép các ứng dụng stateful vận hành mượt mà trên Kubernetes. Khác với Pod trong Deployment, các Pod trong StatefulSet không thể tạo ra một cách ngẫu nhiên bừa bãi. Mỗi Pod sẽ được gắn một định danh duy nhất và bền vững (persistent ID — nếu một Pod chết đi và được phục hồi lại, nó vẫn giữ nguyên ID cũ). Nói cách khác, dù được sinh ra từ cùng một cấu hình mẫu, các Pod này không thể hoán đổi tùy tiện cho nhau. Trong một cụm StatefulSet, thường sẽ có một Pod chính (Master Pod) đảm nhận việc Đọc/Ghi dữ liệu (Read/Write) — nếu tất cả Pod cùng ghi thì dữ liệu sẽ hỗn loạn và mâu thuẫn ngay. Các Pod còn lại đóng vai trò phụ (Slave/Replica Pod), chỉ có quyền Đọc (Read-only) và có bản sao lưu trữ riêng được đồng bộ liên tục từ Master để đảm bảo dữ liệu luôn nhất quán.

![statefulset diagram](https://cdn-images.tryhackme.com/user-uploads/6228f0d4ca8e57005149c3e3/room-content/3c0a1d5471d84bd4b86ae3e5a9fefa1f.svg)  

**Services** 

Để hiểu thấu đáo về Service trong Kubernetes, điều quan trọng là phải nắm rõ bài toán mà nó giải quyết. Các Pod trong Kubernetes có tính chất tạm thời (ephemeral) — vòng đời của chúng rất ngắn, được tạo ra và bị tiêu hủy thường xuyên. Hãy tưởng tượng bây giờ có nhu cầu kết nối tới các Pod này: đó có thể là từ nội bộ cụm (chẳng hạn ứng dụng frontend cần gọi API tới backend chạy trên các Pod), hoặc request đến từ trình duyệt web của người dùng bên ngoài. Để kết nối thành công, bắt buộc phải có địa chỉ IP. Nếu gắn IP trực tiếp vào từng Pod, mỗi khi Pod chết đi và tạo mới, IP sẽ thay đổi liên tục, gây ra vô vàn lỗi đứt đoạn kết nối.

Service sinh ra để giải quyết vấn đề này: nó cung cấp một địa chỉ IP tĩnh duy nhất đại diện cho một nhóm Pod bản sao. Nói cách khác, Service đứng chắn phía trước các Pod, đóng vai trò làm điểm truy cập (access point) duy nhất. Việc có một đầu mối truy cập tập trung này giúp các request được tự động cân bằng tải (load balancing) đến các Pod bản sao phía sau.

Có nhiều loại Service khác nhau mà bạn có thể định nghĩa:
- **ClusterIP:** Loại mặc định, chỉ mở IP truy cập trong nội bộ cụm.
- **NodePort:** Mở một cổng trên mỗi Node để truy cập từ ngoài vào.
- **LoadBalancer:** Tích hợp với dịch vụ cân bằng tải của nhà cung cấp đám mây.
- **ExternalName:** Ánh xạ Service tới một tên miền CNAME bên ngoài.

Để tìm hiểu sâu hơn về các loại Service, bạn có thể tham khảo [tài liệu chính thức của Kubernetes](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types).

![The source of the traffic will depend on the type of service|358](https://cdn-images.tryhackme.com/user-uploads/6228f0d4ca8e57005149c3e3/room-content/263deb8228845aed0db78319097bd63b.svg)  

**Ingress** 

Ở phần trên, chúng ta đã lấy ví dụ về việc dùng Service để mở quyền truy cập cho ứng dụng web (tạm gọi là Service A). Bây giờ, giả sử ứng dụng web có thêm một tính năng mới. Tính năng mới này đòi hỏi một ứng dụng độc lập, chạy trên một tập hợp Pod riêng và được expose qua một Service riêng biệt khác (gọi là Service B).

Lúc này, khi người dùng gửi yêu cầu truy cập tính năng mới đó, hệ thống cần một cơ chế định tuyến lưu lượng (traffic routing) để dẫn request đến đúng Service B. Đó chính là nơi Ingress tỏa sáng. Ingress đóng vai trò như một cửa ngõ duy nhất (single access point) dẫn vào Cluster, giúp bạn gom toàn bộ các quy tắc định tuyến HTTP/HTTPS về quản lý tập trung trong một tài nguyên duy nhất.

![Ingress diagram](https://cdn-images.tryhackme.com/user-uploads/6228f0d4ca8e57005149c3e3/room-content/1b0ab1f0b6ee779c1ee6f2a3f4d4924c.svg)  

### Phân định trách nhiệm: DevOps vs DevSecOps trong K8s

Phần này đã giới thiệu cho bạn các tài nguyên cốt lõi tạo nên một cụm Kubernetes. Tiếp theo, chúng ta sẽ học cách tương tác với cụm để khởi tạo những thành phần này. Nhưng trước đó, hãy dành chút thời gian nói về sự phân công lao động trong thế giới K8s — một điều cực kỳ quan trọng nếu bạn đang định hướng lộ trình nghề nghiệp:
- **Nhiệm vụ của DevOps:** Tập trung vào việc **xây dựng, triển khai và vận hành** cụm (Build & Run).
- **Nhiệm vụ của DevSecOps:** Tập trung vào việc **bảo mật và siết chặt an toàn** cho cụm (Secure).

Tùy vào quy mô và cơ cấu từng công ty mà hai vai trò này có thể đan xen hoặc gộp chung, nhưng về bản chất, trách nhiệm của chúng được phân biệt rõ ràng như vậy. Vì bài viết này mang tính chất nhập môn nên trước hết sẽ hướng dẫn bạn những "viên gạch" nền móng của một cụm K8s và cách thao tác với chúng (công việc của DevOps). Khi đã nắm vững nền tảng này, bạn sẽ dễ dàng tiếp thu các nội dung bảo mật cụm ở những phần tiếp theo (công việc của DevSecOps).

# 4 - Cấu hình Kubernetes

### Đã đến lúc thực hành cấu hình (Time to Configure)

Sau khi đã nắm được các thành phần chính trong Kubernetes, giờ hãy cùng ghép nối chúng lại thông qua một ví dụ cấu hình thực tế! Kịch bản của chúng ta là: Một Deployment điều khiển một ReplicaSet, ReplicaSet quản lý các Pod, và các Pod này được expose ra ngoài thông qua một Service.

![Interfacing with deployment diagram](https://cdn-images.tryhackme.com/user-uploads/6228f0d4ca8e57005149c3e3/room-content/a0872c63c555b82e63ce1ea3dbed6c42.svg)  

Để thiết lập kịch bản này, chúng ta cần hai tệp cấu hình: một cho Deployment và một cho Service. Trước khi đi vào chi tiết từng file, hãy nắm qua một số nguyên tắc cơ bản trong cấu hình Kubernetes:

Đầu tiên là về định dạng tệp: Các file cấu hình Kubernetes thường được viết bằng **YAML**. Bạn cũng có thể dùng định dạng JSON tương đương, nhưng theo khuyến nghị chính thức từ tài liệu Kubernetes, YAML là chuẩn mực thực hành tốt nhất (best practice) vì tính rõ ràng, dễ đọc cho con người (chỉ cần chú ý thụt dòng / indentation cho chuẩn!).

**Các trường bắt buộc (Required Fields)**

Trong bất kỳ file cấu hình YAML nào của Kubernetes, luôn có 4 trường bắt buộc sau:

- **apiVersion:** Phiên bản của Kubernetes API được dùng để khởi tạo đối tượng này. Phiên bản nào sẽ tùy thuộc vào loại đối tượng bạn định nghĩa (bạn có thể tham khảo cheatsheet các API version [tại đây](https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-apiversion-definition-guide.html)).
- **kind:** Loại đối tượng bạn muốn tạo (ví dụ: `Deployment`, `Service`, `StatefulSet`).
- **metadata:** Chứa các thông tin giúp định danh đối tượng (bao gồm `name` và `namespace` nếu có).
- **spec:** Định nghĩa trạng thái mong muốn (desired state) của đối tượng (ví dụ đối với Deployment thì muốn có 3 Pod chạy nginx).

**Cấu hình tài nguyên (Configuring Resources)** 

Đó là những điểm cơ bản nhất. Bây giờ chúng ta sẽ xem xét hai tệp cấu hình thực tế. Theo thực hành chuẩn (best practice), khi khai báo Deployment và Service đi cùng nhau, bạn nên định nghĩa Service trước rồi mới đến Deployment/ReplicaSet (lý do là khi Kubernetes khởi động container, nó sẽ tự động sinh các biến môi trường tương ứng với các Service đang chạy tại thời điểm đó).

Dưới đây là nội dung tệp `example-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: example-nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
  type: ClusterIP
```

Hãy cùng "giải phẫu" file này:
- **apiVersion:** đặt là `v1` (phiên bản chuẩn phù hợp cho cấu hình Service đơn giản này).
- **kind:** đặt là `Service`.
- **metadata:** đặt tên service là `example-nginx-service`.
- **spec:** Đây là phần thú vị nhất:
  - Trong `selector`, có trường `app: nginx`. Đây là nhãn (label) cực kỳ quan trọng để liên kết với Deployment mà chúng ta sẽ định nghĩa ngay sau đây.
  - Phần `ports`: Thiết lập giao thức TCP. Lưu ý phân biệt rõ ràng giữa **`port`** và **`targetPort`**:
    - `targetPort`: Cổng mà Service sẽ chuyển tiếp yêu cầu đến — tức là cổng mà các container trong Pod đang lắng nghe (ở đây là cổng `80`).
    - `port`: Cổng mà chính Service này mở ra (expose) để các client khác gọi vào (ở đây là cổng `8080`).
  - Cuối cùng, `type` được đặt là `ClusterIP` — loại Service mặc định dùng cho giao tiếp nội bộ trong cụm.

Tiếp theo, hãy xem file YAML của Deployment để định nghĩa phần backend mà Service này trỏ tới:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

Điều đầu tiên bạn có thể nhận thấy là bên trong trường `spec`, có một trường lồng nhau là `template`, và bên trong `template` lại chứa các trường `metadata` và `spec` con.

Để hiểu điều này, hãy nhớ lại sơ đồ ở đầu bài: Chúng ta đang định nghĩa một Deployment quản lý một ReplicaSet.
- Ở trường `spec` bên ngoài: Ta yêu cầu Kubernetes duy trì `replicas: 3` (3 Pod giống hệt nhau).
- Trường `template`: Chính là khuôn mẫu (blueprint) mà Kubernetes sẽ dùng để đúc ra từng Pod đó. Vì vậy, nó cần có `metadata` riêng (để gán nhãn `app: nginx` cho Pod) và `spec` riêng (để Kubernetes biết kéo image `nginx:latest` nào và lắng nghe ở cổng `containerPort: 80`).
- Hãy để ý rằng `containerPort: 80` này trùng khớp với `targetPort: 80` trong file Service ở trên.
- Đồng thời, `matchLabels: { app: nginx }` ở `spec` bên ngoài cũng khớp với `selector: { app: nginx }` của Service. Nhờ việc khớp nhãn này mà Service biết chính xác cần phải phân phối lưu lượng tới những Pod nào.

Chỉ với hai tệp YAML này, chúng ta đã định nghĩa trọn vẹn một Deployment quản lý một ReplicaSet với 3 Pod, tất cả đều được đưa ra ngoài thông qua một Service. Mọi thứ đã ăn khớp hoàn hảo!

Các file cấu hình này chính là bản mô tả "trạng thái mong muốn" (desired state). Kubernetes sẽ liên tục đối chiếu trạng thái mong muốn này với trạng thái thực tế của cụm thông qua etcd (kho dữ liệu của Control Plane). Ví dụ, nếu bạn yêu cầu có 3 Pod nginx đang chạy, nhưng Kubernetes nhận thấy hiện chỉ có 2 Pod sống, nó sẽ ngay lập tức tự động kích hoạt tiến trình tạo thêm một Pod mới để bù lại.

# 5 - Làm chủ công cụ kubectl (Kubectl to the Rescue)

### Kubectl - Cứu tinh của chúng ta

Chúng ta vừa biết cách định nghĩa trạng thái mong muốn của cụm bằng các tệp cấu hình YAML, nhưng hiện tại chúng vẫn chỉ nằm yên trên ổ đĩa dưới dạng tệp văn bản. Để biến cấu hình này thành các tiến trình thực sự chạy trong cụm, chúng ta cần tương tác với Control Plane.

Có nhiều cách để giao tiếp với kube-apiserver:
- Dùng giao diện web (UI): nếu bạn cài đặt Kubernetes Dashboard.
- Dùng API: nếu bạn viết script tự động hóa.
- Dùng dòng lệnh (CLI): thông qua công cụ quyền lực mang tên **kubectl**.

Phần này sẽ tập trung vào kubectl. Nói đơn giản, `kubectl` là công cụ dòng lệnh chính thức của Kubernetes giúp chúng ta tương tác trực tiếp với Control Plane của cụm — đây là vũ khí không thể thiếu trong hành trang DevSecOps của bạn. Bạn sẽ học cách áp dụng (apply) các file cấu hình từ phần trước, cũng như nắm vững những câu lệnh cơ bản nhất để tự tin điều hướng một cụm Kubernetes như trong lòng bàn tay!

**Kubectl apply** 

Sau khi đã soạn xong cấu hình Deployment và Service vào các file YAML, bước tiếp theo là áp dụng chúng để Kubernetes đọc cấu hình và khởi chạy các tiến trình thực tế. Ta sử dụng lệnh `apply`:

Ví dụ, để áp dụng file cấu hình:

```bash
kubectl apply -f example-deployment.yaml
```

**Kubectl get** 

Khi đã áp dụng cấu hình xong, bạn sẽ muốn kiểm tra xem mọi thứ có đang chạy ổn định như mong đợi hay không. Đây là lúc dùng lệnh `kubectl get`. Đây là một lệnh cực kỳ đa năng mà bạn sẽ dùng liên tục hàng ngày:

Cú pháp cơ bản: Sau `get` là loại tài nguyên, kèm theo cờ `-n` hoặc `--namespace` để chỉ định Namespace (trừ các tài nguyên ở cấp Cluster như Node thì không cần namespace).

Ví dụ, để kiểm tra danh sách các Pod:

```bash
kubectl get pods -n example-namespace
```

Kết quả trả về trên Terminal sẽ có dạng:

```shell-session
user@tryhackme$ kubectl get pods -n example-namespace
NAME          READY   STATUS              RESTARTS   AGE
example-pod   1/1     Running             0          2m18s
```

Bạn có thể dùng lệnh này để tra cứu nhiều loại tài nguyên khác nhau như: `deployments`, `services`, `pods`, `replicasets`...

**Kubectl describe** 

Lệnh này dùng để xem thông tin chi tiết của một tài nguyên cụ thể (hoặc một nhóm tài nguyên). Những chi tiết này cực kỳ hữu ích trong quá trình khắc phục sự cố (troubleshooting) hoặc phân tích bảo mật.

Ví dụ: Giả sử một Pod trong cụm bị lỗi hoặc crash, bạn muốn tìm hiểu nguyên nhân tại sao, bạn chạy lệnh sau:

```bash
kubectl describe pod example-pod -n example-namespace
```

Lệnh sẽ hiển thị thông tin chi tiết về Pod bị lỗi, ví dụ:

```shell-session
user@tryhackme$ kubectl describe pod example-pod -n example-namespace

Name:             example-pod
Namespace:        example-namespace
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Mon, 22 Jan 2024 14:01:14 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.244.0.21
...
...
```

Đặc biệt, ở phần cuối của kết quả `describe` luôn có mục **Events** (sự kiện). Các sự kiện này là manh mối vàng giúp bạn nhận diện nguyên nhân lỗi (như kéo image thất bại, thiếu bộ nhớ, liveness probe fail...). Để đào sâu hơn nữa nguyên nhân bên trong ứng dụng, chúng ta có thể xem log của container bằng lệnh tiếp theo.

**Kubectl logs** 

Khi cần xem log ứng dụng của các Pod đang gặp sự cố, bạn dùng lệnh `kubectl logs`:

Ví dụ:

```bash
kubectl logs example-pod -n example-namespace
```

Những dòng log này không chỉ cung cấp thông tin quý giá cho việc debug ứng dụng mà còn phục vụ đắc lực cho việc phân tích bảo mật (như phát hiện các hành vi bất thường, tấn công dò quét).

**Kubectl exec**

Nếu việc đọc log vẫn chưa giải quyết được hết các nghi vấn và bạn muốn trực tiếp truy cập vào bên trong container để điều tra, lệnh `kubectl exec` sẽ mở một shell cho bạn làm điều đó!

Nếu một Pod có nhiều container, bạn có thể chỉ định container cụ thể bằng cờ `-c` hoặc `--container`. Cú pháp lệnh như sau (cờ `-it` giúp mở chế độ tương tác terminal, phần sau dấu `--` là lệnh sẽ chạy bên trong container):

```bash
kubectl exec -it example-pod -n example-namespace -- sh
```

Từ đây, bạn có thể chạy bất kỳ lệnh nào từ bên trong container: kiểm tra môi trường, tìm kiếm dấu vết lạ phục vụ phân tích bảo mật hoặc test kết nối mạng nội bộ.

**Kubectl port-forward** 

Một lệnh cực kỳ tiện lợi khác là `kubectl port-forward`. Lệnh này tạo một đường hầm bảo mật (secure tunnel) giữa máy tính cục bộ của bạn và một Pod đang chạy trong cụm.

Tình huống thực tế rất hay dùng: Bạn muốn kiểm thử một ứng dụng web trước khi công khai rộng rãi. Giả sử ta có ứng dụng nginx chạy trên 3 Pod và được kết nối qua một Service. Để truy cập thử, ta có thể ánh xạ cổng của Service vào cổng máy local.

Ví dụ: Ánh xạ cổng của Service (trong ví dụ trước là `8080`) sang cổng `8090` trên máy local, giúp bạn có thể mở trình duyệt truy cập ngay tại `http://localhost:8090`. Cú pháp tài nguyên có dạng `resource-type/resource-name`:

```bash
kubectl port-forward service/example-service 8090:8080
```

Dĩ nhiên, Kubernetes còn rất nhiều lệnh `kubectl` khác để khám phá, nhưng các lệnh cơ bản trên đã cung cấp cho bạn một nền tảng vững chắc để thực hiện hầu hết các tác vụ DevSecOps hằng ngày. Bạn sẽ có cơ hội trực tiếp thực hành những câu lệnh này trong bài lab thực hành ở Task 8.

---

### Câu hỏi củng cố:
Từ khóa lệnh nào sẽ đi sau `kubectl` nếu bạn muốn...

# 6 - Kubernetes và DevSecOps (Kubernetes and DevSecOps)
Được rồi, đến thời điểm này, bạn đã khá quen thuộc với Kubernetes; nó không còn là một người quen xa lạ mà đang dần trở thành một người bạn thân thiết. Bạn đã tìm hiểu tường tận về công cụ này và sẽ sớm có cơ hội thực hành demo, nhưng trước hết, hãy cùng xem chính xác cách BẠN sẽ sử dụng công cụ này với tư cách là một kỹ sư DevSecOps. Sau tất cả, đó rất có thể là lý do tại sao bạn ở đây!

**Kubernetes và Bảo mật**

Trước khi đi sâu vào các trách nhiệm của một kỹ sư DevSecOps trong một cụm (cluster) Kubernetes, hãy cùng xem xét Kubernetes dưới góc độ bảo mật để có thêm ngữ cảnh. Nói một cách tương đối, Kubernetes vẫn còn khá mới mẻ, hay có thể nói là một công nghệ đang nổi. Dù mới nổi nhưng nó lại cực kỳ phổ biến, với nhiều công ty đang áp dụng, đặc biệt là các công ty công nghệ trẻ và start-up.

Việc đưa bất kỳ công cụ nào vào hệ thống đều được coi là làm tăng rủi ro bảo mật, vì một công cụ mới đồng nghĩa với một con đường tiềm ẩn mới để xâm nhập vào hệ thống đó. Những rủi ro này càng tăng lên gấp bội đối với một công cụ như Kubernetes, nơi bạn có một mạng lưới các pod có thể giao tiếp chéo với nhau. Thiết lập mặc định cho phép bất kỳ pod nào cũng có thể giao tiếp với pod khác. Điều này dẫn đến đủ loại vấn đề cần cân nhắc về bảo mật. Với tư cách là một kỹ sư DevSecOps, trách nhiệm của bạn là đảm bảo các kênh giao tiếp này được an toàn.

**Tăng cường bảo mật (Hardening) Kubernetes**

Tăng cường bảo mật container (Container hardening) là một cách để các kỹ sư DevSecOps bảo mật các kênh giao tiếp này, và chúng ta sẽ tìm hiểu kỹ hơn về nó ở phần sau của mô-đun này trong phòng Container Hardening. Đây là quy trình sử dụng các công cụ quét container để phát hiện các lỗ hổng (CVE) tồn tại trong cụm và khắc phục chúng nhằm đảm bảo rủi ro vi phạm bảo mật là thấp nhất.

Tăng cường bảo mật Kubernetes chính xác là việc đảm bảo an toàn cho các kênh này bằng cách củng cố cụm của bạn theo các thực hành bảo mật container tốt nhất (best practices) mà bạn, với tư cách là kỹ sư DevSecOps, sẽ thực hiện. Nhiều công ty và cơ quan chính phủ đã xác định các quy chuẩn này; hãy cùng điểm qua từng lĩnh vực mà chúng ta có thể tăng cường bảo mật container và cách thực hiện.

**Bảo mật các Pod của bạn!**

Hãy bắt đầu với một vài cách để bảo mật chính các pod. Một số thực hành tốt nhất cho bảo mật pod bao gồm:

* Các container chạy ứng dụng không được có quyền root.
* Các container nên có hệ thống tệp bất biến (immutable filesystem), nghĩa là chúng không thể bị thay đổi hoặc thêm vào (tùy thuộc vào mục đích của container, điều này có thể không khả thi).
* Image của container nên được quét thường xuyên để tìm lỗ hổng hoặc cấu hình sai.
* Nên ngăn chặn các container có đặc quyền (privileged containers).
* Tham khảo thêm về Tiêu chuẩn Bảo mật Pod (Pod Security Standards) và Kiểm soát Nhập học Bảo mật Pod (Pod Security Admission).

**Tăng cường bảo mật và Phân tách Mạng!**

Trong phần giới thiệu tác vụ này, có một điều đặc biệt được cảnh báo là rủi ro bảo mật lớn: giao tiếp. Giao tiếp đó diễn ra qua mạng và công việc của bạn với tư cách là kỹ sư DevSecOps là đảm bảo giao tiếp này an toàn. Điều này có thể được thực hiện bằng các thực hành tốt nhất sau:

* Quyền truy cập vào nút điều khiển (control plane node) nên được hạn chế bằng tường lửa và kiểm soát truy cập dựa trên vai trò (RBAC) trong một mạng cô lập.
* Các thành phần của control plane nên giao tiếp bằng chứng chỉ Bảo mật tầng truyền tải (TLS).
* Nên tạo chính sách từ chối rõ ràng (explicit deny policy).
* Thông tin đăng nhập và thông tin nhạy cảm không nên được lưu trữ dưới dạng văn bản thuần túy (plain text) trong các tệp cấu hình. Thay vào đó, chúng nên được mã hóa và lưu trong Kubernetes secrets.

**Sử dụng Xác thực và Ủy quyền một cách tối ưu**

Tất nhiên, đây sẽ không phải là một bài học về bảo mật nếu chúng ta không nói về xác thực (authentication) và ủy quyền (authorisation)! Kubernetes cũng không ngoại lệ. Dưới đây là một số thực hành tốt nhất có thể giúp đảm bảo bạn đang sử dụng hiệu quả các tính năng xác thực và ủy quyền của Kubernetes:

* Nên vô hiệu hóa quyền truy cập ẩn danh (anonymous access).
* Nên sử dụng phương thức xác thực người dùng mạnh.
* Nên tạo các chính sách RBAC cho các nhóm khác nhau sử dụng cụm và các tài khoản dịch vụ (service accounts) được sử dụng.

**Luôn mắt quan sát**

Bạn không thể an tâm kê cao gối ngủ khi biết cụm Kubernetes của mình an toàn nếu bạn không biết chuyện gì đang xảy ra bên trong đó. Dưới đây là một số thực hành tốt nhất về ghi nhật ký (logging) để đảm bảo bạn biết chính xác chuyện gì đang xảy ra trong cụm của mình và có thể phát hiện các mối đe dọa khi chúng xuất hiện:

* Nên bật ghi nhật ký kiểm tra (audit logging).
* Nên triển khai hệ thống giám sát nhật ký và cảnh báo.

**Bảo mật không bao giờ ngủ**

Điều này hoàn toàn không có nghĩa là ủng hộ lối sống thiếu ngủ; các kỹ sư DevSecOps thực sự cần phải ngủ! Trở nên an toàn là một chuyện, duy trì sự an toàn lại là một chuyện khác. Dưới đây là một số thực hành tốt nhất để đảm bảo cụm của bạn luôn là một nơi trú ẩn an toàn:

* Các bản vá bảo mật và cập nhật nên được áp dụng nhanh chóng.
* Quét lỗ hổng và kiểm tra xâm nhập (penetration tests) nên được thực hiện bán định kỳ.
* Nên loại bỏ bất kỳ thành phần lỗi thời nào trong cụm.

**Thực hành tốt nhất về Bảo mật Kubernetes trong Hành động**

Thông tin trên cho thấy có rất nhiều cách để tăng cường bảo mật hạ tầng Kubernetes. Trên thực tế, có quá nhiều cách đến nỗi nếu phân tích chi tiết từng thực hành sẽ biến phòng này thành một cuốn sách điện tử tự xuất bản. Vì cơ hội để bạn thực hành Kubernetes chỉ còn cách một tác vụ nữa thôi, hãy kết thúc phần này bằng cách phân tích chỉ ba trong số đó.

**RBAC**

RBAC (Role-Based Access Control - Kiểm soát truy cập dựa trên vai trò) trong Kubernetes điều chỉnh quyền truy cập vào cụm Kubernetes và các tài nguyên của nó dựa trên các vai trò và quyền hạn được xác định. Các quyền này (quyền tạo/xóa tài nguyên x, v.v.) được gán cho người dùng, nhóm hoặc tài khoản dịch vụ. RBAC là một cách tốt để đảm bảo rằng các tài nguyên trong cụm của bạn chỉ có thể được truy cập bởi những người cần truy cập chúng. RBAC có thể được cấu hình bằng tệp YAML (giống như định nghĩa một tài nguyên) nơi các quy tắc cụ thể có thể được xác định bằng cách khai báo loại tài nguyên và các động từ (verbs). Động từ là các hành động bị hạn chế, chẳng hạn như 'create' (tạo) và 'get' (lấy).

**Quản lý Secrets**

Kubernetes secret là một đối tượng được sử dụng để lưu trữ thông tin nhạy cảm (như thông tin đăng nhập, OAuth token hoặc SSH key). Secrets là một cách tốt để đảm bảo rằng dữ liệu nhạy cảm không bị rò lề và cho phép kiểm soát nhiều hơn cách sử dụng thông tin này. Secrets được lưu trữ dưới dạng chuỗi mã hóa base64, không được mã hóa theo mặc định. Để bảo mật, tốt nhất là cấu hình mã hóa khi lưu trữ (encryption at rest). Một cách khác để thúc đẩy quản lý secrets an toàn trong cụm Kubernetes của bạn là cấu hình quyền truy cập tối thiểu (least privilege access) vào secrets bằng RBAC.

**PSA (Pod Security Admission) và PSS (Pod Security Standards)**

Tiêu chuẩn Bảo mật Pod (Pod Security Standards) được sử dụng để xác định các chính sách bảo mật ở 3 cấp độ (privileged - có đặc quyền, baseline - cơ bản và restricted - bị hạn chế) ở cấp độ namespace hoặc toàn cụm. Ý nghĩa của các cấp độ này:

* **Privileged:** Đây là chính sách gần như không bị hạn chế (cho phép các leo thang đặc quyền đã biết).
* **Baseline:** Đây là chính sách hạn chế tối thiểu và sẽ ngăn chặn các leo thang đặc quyền đã biết (cho phép triển khai các pod với cấu hình mặc định).
* **Restricted:** Chính sách bị hạn chế nghiêm ngặt này tuân theo các thực hành tốt nhất về tăng cường bảo mật pod hiện tại.

Kiểm soát Nhập học Bảo mật Pod (Pod Security Admission - sử dụng bộ điều khiển Pod Security Admission) thực thi các Tiêu chuẩn Bảo mật Pod này bằng cách chặn các yêu cầu đến API server và áp dụng các chính sách này.

*Lưu ý: Trước đây, vai trò của PSA và PSS được thực hiện bằng PSP (Pod Security Policies); tuy nhiên, kể từ Kubernetes v1.25, chúng đã bị loại bỏ. Lưu ý điều này để tránh nhầm lẫn nếu bạn tình cờ gặp PSP khi tự nghiên cứu thêm về bảo mật Kubernetes!*

Như bạn có thể thấy, có rất nhiều việc phải làm để củng cố một cụm Kubernetes và đủ để giữ cho một kỹ sư DevSecOps luôn bận rộn. Đây là một số thực hành tốt nhất liên quan đến việc tăng cường bảo mật Kubernetes. Các kỹ sư DevSecOps, trong khi tuân theo các thực hành này và sử dụng các kỹ thuật như kiểm tra bảo mật tự động, có thể đảm bảo môi trường Kubernetes của họ được bảo vệ khỏi các mối đe dọa mạng.