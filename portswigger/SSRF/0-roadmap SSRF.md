### 1-ROADMAP SSRF
SSRF (Server-Side Request Forgery)
│
├── **1. Core Concept (Nền tảng)**
│   │
│   ├── Server-side request **-->** [[định nghĩa SSRF]]
│   │   ├── Client gửi URL cho server
│   │   ├── Server tự gửi request thay client
│   │   └── Attacker điều khiển destination
│   │
│   ├── Trust Boundary -->[[Hướng tư duy khai thác SSRF]]->
│   │   ├── External user
│   │   ├── Vulnerable server
│   │   └── Internal network
│   │
│   ├── Impact **-->**[[impact của SSRF ]]
│   │   ├── Read internal data
│   │   ├── Access admin panel
│   │   ├── Scan internal network
│   │   ├── Cloud metadata theft
│   │   └── Remote Code Execution (chain)
│   │
│   └── Common Entry Point **-->** [[các entry point trong SSRF]]
│       ├── URL parameter
│       │   └── stockApi=https://example.com
│       │
│       ├── Image fetch
│       ├── PDF generator
│       ├── Webhook
│       ├── Import URL
│       └── File upload URL
│
├── **2. SSRF Against Server Itself**
│   │ **==>** [[SSRF-Blacklist bypass]] 
│   ├── Loopback
│   │   ├── localhost
│   │   ├── 127.0.0.1
│   │   └── ::1
│   │
│   ├── Internal Services
│   │   ├── Admin panel
│   │   ├── Debug interface
│   │   ├── Monitoring service
│   │   └── Internal API
│   │
│   └── Attack Flow
│       │
│       ├── Find URL input
│       ├── Replace URL
│       ├── Access localhost
│       └── Abuse internal privilege
│
├── **3. SSRF Against Other Back-end Systems**
│   │ **=>**  [[dạng SSRF lấy dữ liệu từ backend system-internal system]] 
│   ├── Internal IP Range
│   │   ├── 10.0.0.0/8
│   │   ├── 172.16.0.0/12
│   │   └── 192.168.0.0/16
│   │
│   ├── Network Enumeration
│   │   ├── Scan IP
│   │   ├── Scan Port
│   │   └── Identify service
│   │
│   ├── Tools
│   │   ├── Burp Intruder
│   │   ├── ffuf
│   │   └── custom script
│   │
│   └── Goal
│       ├── Find hidden admin server
│       ├── Access internal API
│       └── Extract sensitive data
│
├── **4. SSRF Filter Bypass**
│   │
│   ├── **Blacklist Filter** **=>** [[Blacklist filter bypass]]  , 
│   │   │ **-->** [[SSRF-Blacklist bypass]] 
│   │   ├── Block:
│   │   │   ├── localhost
│   │   │   └── 127.0.0.1
│   │   │
│   │   ├── Bypass IP Encoding
│   │   │   ├── Decimal IP
│   │   │   │   └── 2130706433
│   │   │   ├── Octal IP
│   │   │   └── Hex IP
│   │   │
│   │   ├── URL Encoding
│   │   ├── Case variation
│   │   └── DNS rebinding
│   │
│   ├── **Whitelist Filter**
│   │   │ **==>** [[whitelist bypass]] , [[ssrf-whitelist bypass]]
│   │   ├── URL Parser Confusion
│   │   │
│   │   ├── @ bypass
│   │   │   └── trusted.com@evil.com
│   │   │
│   │   ├── # Fragment bypass
│   │   │
│   │   ├── Subdomain bypass
│   │   │
│   │   └── Encoding mismatch
│   │
│   └── Open Redirect Chain
│       │
│       ├── Allowed domain
│       ├── Redirect
│       └── Internal target
│
├── **5. Blind SSRF**  **==>** [[blind SSRF]]
│   │
│   ├── Definition
│   │   ├── Server sends request
│   │   └── No response returned
│   │
│   ├── Detection
│   │   ├── Burp Collaborator
│   │   ├── DNS callback
│   │   └── HTTP callback
│   │
│   ├── Exploitation
│   │   ├── Data exfiltration
│   │   ├── Internal interaction
│   │   └── RCE chain
│   │
│   └── Advanced
│       └── Blind SSRF + Shellshock
│
├── **6. Hidden SSRF Attack Surface** ==>[[hidden ssrf attack surface]]
│   │
│   ├── Partial URL
│   │   ├── hostname only
│   │   └── path injection
│   │
│   ├── Data Formats
│   │   ├── JSON
│   │   ├── XML
│   │   └── YAML
│   │
│   ├── HTTP Headers
│   │   └── Referer header
│   │
│   └── File Processing
│       ├── PDF generation
│       ├── Image processing
│       └── Document preview
│
├── **7. Cloud SSRF** (Quan trọng khi pentest)
│   │
│   ├── AWS Metadata
│   │   └── 169.254.169.254
│   │
│   ├── Azure Metadata
│   │
│   ├── GCP Metadata
│   │
│   └── Impact
│       ├── IAM credentials
│       ├── API keys
│       └── Cloud takeover
│
└── **8. Defense** (Hiểu để bypass)
    │
    ├── Do not trust user URL
    ├── URL parser chuẩn
    ├── Allowlist destination
    ├── Block internal IP
    ├── Network segmentation
    ├── Disable unnecessary protocols
    └── Monitor outbound requests


**Thứ tự học lab PortSwigger nên đi:**

```
1. Basic SSRF localhost
        ↓
2. SSRF internal backend scan
        ↓
3. SSRF blacklist bypass
        ↓
4. SSRF open redirect bypass
        ↓
5. Blind SSRF + Collaborator
        ↓
6. Whitelist bypass
        ↓
7. SSRF + Cloud Metadata
```


> [!NOTE]
> SSRF = "Tôi không truy cập được mục tiêu,
> nhưng server có quyền truy cập,
> nên tôi bắt server truy cập hộ tôi."
> 

Sau SSRF nên học tiếp **XXE → File Upload → Deserialization → Cloud Security**, vì các lỗi này thường chain với SSRF trong pentest thực tế. [PortSwigger](https://portswigger.net/web-security/all-topics?utm_source=chatgpt.com)


### 2-LAB-SSRF






