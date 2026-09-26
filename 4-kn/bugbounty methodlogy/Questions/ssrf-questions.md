# Câu Hỏi Định Hướng Khai Thác Lỗ Hổng SSRF (Server-Side Request Forgery)

> **SSRF (Server-Side Request Forgery)** là kỹ thuật tấn công ép buộc máy chủ mục tiêu (vulnerable server) thực hiện một yêu cầu HTTP mà chủ sở hữu hệ thống không hề mong muốn hoặc không lường trước.

---

## 1. TẠI SAO bạn lại thực hiện cuộc tấn công? (WHY)
- **Giả mạo yêu cầu HTTP** gửi tới các dịch vụ hoặc tài nguyên nội bộ (internal services/resources) nhằm trích xuất thông tin nhạy cảm (như cloud metadata, config, internal token).
- **Khai thác SSRF để quét và rà soát mạng nội bộ (Port Scanning / Network Enumeration)** hoặc khám phá các dịch vụ ẩn không công khai ra internet.
- **Thao túng SSRF để vượt qua các lớp kiểm soát bảo mật** (firewall, IP whitelist) nhằm truy cập dữ liệu hoặc tính năng bị giới hạn.
- **Chuyển hướng (Redirect) các yêu cầu từ mạng nội bộ ra máy chủ độc hại bên ngoài** của kẻ tấn công, phục vụ cho việc đánh cắp / trích xuất dữ liệu (Data Exfiltration).
- **Tạo payload SSRF để thực hiện các hành động trái phép** nhân danh người dùng hợp pháp hoặc tài khoản dịch vụ của hệ thống.
- **Chuyển hướng yêu cầu tới các tài nguyên bên ngoài** để lạm dụng chức năng ứng dụng hoặc gây gián đoạn dịch vụ (DoS).
- **Tạo ra các yêu cầu HTTP độc hại nhân danh tổ chức mục tiêu** nhằm phá hoại uy tín thương hiệu và gây thiệt hại tài chính.

---

## 2. Nạn nhân là AI? (WHO)
- Tổ chức / doanh nghiệp sở hữu và vận hành hạ tầng máy chủ đó.

---

## 3. Bạn đang khai thác CÔNG NGHỆ nào? (WHAT)
- **Mã nguồn phía máy chủ (Server-Side Code) trong ứng dụng:** Các thư viện HTTP client (ví dụ: `axios`, `HttpURLConnection`, `HttpClient`, `cURL`, v.v.).
- **Lỗ hổng Command Injection trên máy chủ:** Ứng dụng gọi lệnh hệ thống (`curl`, `wget`, v.v.).
- **Lỗ hổng CVE đã công bố** trong hạ tầng / phần mềm (ví dụ: proxy, gateway, CMS).
- **Dịch vụ Cloud bị cấu hình sai (Misconfigured Cloud Service):** Điển hình là việc không chặn truy cập vào Endpoint Metadata của Cloud (AWS IMDSv1 `169.254.169.254`, GCP, Azure).

---

## 4. KHI NÀO nên thực hiện cuộc tấn công? (WHEN)
- **Nếu tấn công có chủ đích (Targeted Attack):** Khi dịch vụ mục tiêu đang hoạt động bình thường (up).
- **Nếu rà quét / liệt kê (Enumerating):** Quá trình dò quét mạng nội bộ sẽ tạo ra rất nhiều lưu lượng (noisy) – nên chạy vào ban đêm để tránh bị phát hiện? Hay chạy vào ban ngày khi lưu lượng truy cập chung của hệ thống cao để ẩn mình giữa các request khác?

---

## 5. Có thể thực hiện cuộc tấn công ở ĐÂU? (WHERE)
- Bất kỳ đâu có kết nối Internet.

---

## 6. Bạn sẽ đưa Payload vào ứng dụng NHƯ THẾ NÀO? (HOW)
- Qua **HTTP Request**.
- Qua **bất kỳ dữ liệu đầu vào nào do người dùng kiểm soát** (tham số URL query, body param JSON/XML/form, HTTP header như `Referer`, `X-Forwarded-For`, cookie, webhook URL, v.v.).

---

## 7. Các Ví Dụ Về Mã Nguồn Dính Lỗi (Vulnerable Code Examples)

### JavaScript (Node.js - Axios)
```javascript
const axios = require('axios');

// URL này lấy trực tiếp từ input người dùng (ví dụ: query parameter trên request)
const url = req.query.url; 

axios.get(url)
  .then((response) => {
    console.log(response.data);
  })
  .catch((error) => {
    console.error(error);
  });
```

### Java (`HttpURLConnection`)
```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;

public class SSRFExample {
    public static void main(String[] args) {
        String inputURL = args[0]; // URL lấy từ input người dùng
        try {
            URL url = new URL(inputURL);
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            connection.setRequestMethod("GET");
            BufferedReader reader = new BufferedReader(new InputStreamReader(connection.getInputStream()));
            String line;
            StringBuilder response = new StringBuilder();
            while ((line = reader.readLine()) != null) {
                response.append(line);
            }
            reader.close();
            System.out.println(response.toString());
            connection.disconnect();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### C# / .NET (`HttpClient`)
```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

class Program
{
    static async Task Main(string[] args)
    {
        string inputURL = args[0]; // Giả định inputURL lấy từ input người dùng

        using (HttpClient client = new HttpClient())
        {
            try
            {
                HttpResponseMessage response = await client.GetAsync(inputURL);
                response.EnsureSuccessStatusCode();
                string responseBody = await response.Content.ReadAsStringAsync();
                Console.WriteLine(responseBody);
            }
            catch (HttpRequestException e)
            {
                Console.WriteLine($"Lỗi request: {e.Message}");
            }
        }
    }
}
```

### PHP (`cURL`)
```php
<?php
$inputURL = $_GET['url']; // Giả định inputURL lấy từ query parameter người dùng

$ch = curl_init($inputURL);

curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

$response = curl_exec($ch);

if ($response === false) {
    echo 'cURL error: ' . curl_error($ch);
} else {
    echo $response;
}

curl_close($ch);
?>
```
