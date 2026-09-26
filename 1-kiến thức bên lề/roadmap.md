```text
RED TEAM ROADMAP
│
├── 0. FOUNDATION — NỀN MÓNG BẮT BUỘC
│   │
│   ├── Networking
│   │   ├── OSI Model
│   │   ├── TCP/IP
│   │   ├── TCP vs UDP
│   │   ├── IP Address
│   │   ├── Subnetting
│   │   ├── MAC Address
│   │   ├── ARP
│   │   ├── DNS
│   │   ├── DHCP
│   │   ├── NAT / PAT
│   │   ├── Routing
│   │   ├── VLAN
│   │   ├── VPN
│   │   ├── Proxy
│   │   ├── Firewall
│   │   ├── Load Balancer
│   │   │
│   │   ├── Common Protocols
│   │   │   ├── HTTP / HTTPS
│   │   │   ├── FTP
│   │   │   ├── SMB
│   │   │   ├── SSH
│   │   │   ├── RDP
│   │   │   ├── LDAP
│   │   │   ├── Kerberos
│   │   │   ├── DNS
│   │   │   ├── SMTP
│   │   │   └── SNMP
│   │   │
│   │   └── Packet Analysis
│   │       ├── Wireshark
│   │       ├── tcpdump
│   │       └── PCAP Analysis
│   │
│   ├── Linux
│   │   ├── Filesystem
│   │   ├── Permissions
│   │   │   ├── chmod
│   │   │   ├── chown
│   │   │   └── SUID / SGID
│   │   ├── Users / Groups
│   │   ├── Processes
│   │   ├── Services
│   │   ├── systemd
│   │   ├── Environment Variables
│   │   ├── Bash
│   │   ├── Pipes / Redirection
│   │   ├── grep
│   │   ├── awk
│   │   ├── sed
│   │   ├── find
│   │   ├── cron
│   │   ├── SSH
│   │   └── Linux Privilege Escalation
│   │
│   └── Windows
│       ├── Filesystem
│       ├── NTFS Permissions
│       ├── Users / Groups
│       ├── Services
│       ├── Registry
│       ├── Processes
│       ├── Event Logs
│       ├── PowerShell
│       ├── CMD
│       ├── Windows Defender
│       └── Windows Privilege Escalation
│
├── 1. PROGRAMMING FOR RED TEAM
│   │
│   ├── Python ⭐⭐⭐⭐⭐
│   │   ├── Syntax
│   │   ├── Functions
│   │   ├── Classes
│   │   ├── Files
│   │   ├── Requests
│   │   ├── Sockets
│   │   ├── subprocess
│   │   ├── Threading
│   │   └── Automation
│   │
│   ├── Bash
│   │   ├── Automation
│   │   ├── Enumeration
│   │   └── Linux Scripts
│   │
│   ├── PowerShell ⭐⭐⭐⭐⭐
│   │   ├── Windows Enumeration
│   │   ├── AD Enumeration
│   │   ├── Object Pipeline
│   │   ├── .NET Interaction
│   │   └── Automation
│   │
│   ├── C / C++
│   │   ├── Memory
│   │   ├── Pointers
│   │   ├── Windows API
│   │   ├── Process
│   │   └── Native Tools
│   │
│   └── C#
│       ├── .NET
│       ├── Windows API
│       ├── Assemblies
│       └── Red Team Tool Development
│
├── 2. WEB SECURITY — INITIAL ACCESS
│   │
│   ├── HTTP Deep Understanding
│   │   ├── Request
│   │   ├── Response
│   │   ├── Headers
│   │   ├── Cookies
│   │   ├── Sessions
│   │   ├── Authentication
│   │   └── Authorization
│   │
│   ├── Burp Suite
│   │   ├── Proxy
│   │   ├── Repeater
│   │   ├── Intruder
│   │   ├── Scanner
│   │   ├── Collaborator
│   │   └── Extensions
│   │
│   ├── Vulnerabilities
│   │   ├── SQL Injection
│   │   ├── XSS
│   │   ├── CSRF
│   │   ├── SSRF
│   │   ├── XXE
│   │   ├── SSTI
│   │   ├── File Upload
│   │   ├── Path Traversal
│   │   ├── Command Injection
│   │   ├── Authentication Bugs
│   │   ├── Access Control
│   │   ├── IDOR
│   │   ├── Race Conditions
│   │   ├── Deserialization
│   │   └── API Security
│   │
│   └── Web → Initial Foothold
│       ├── Web Shell
│       ├── RCE
│       ├── Reverse Shell
│       └── Pivot into Internal Network
│
├── 3. PENTEST METHODOLOGY
│   │
│   ├── Reconnaissance
│   │   ├── Passive Recon
│   │   ├── Active Recon
│   │   ├── OSINT
│   │   └── Attack Surface Mapping
│   │
│   ├── Enumeration
│   │   ├── Hosts
│   │   ├── Ports
│   │   ├── Services
│   │   ├── Users
│   │   ├── Shares
│   │   ├── Applications
│   │   └── Versions
│   │
│   ├── Exploitation
│   ├── Privilege Escalation
│   ├── Lateral Movement
│   ├── Pivoting
│   └── Reporting
│
├── 4. ACTIVE DIRECTORY ⭐⭐⭐⭐⭐⭐⭐
│   │
│   ├── AD Fundamentals
│   │   ├── Domain
│   │   ├── Forest
│   │   ├── Tree
│   │   ├── Domain Controller
│   │   ├── OU
│   │   ├── Users
│   │   ├── Groups
│   │   └── Trusts
│   │
│   ├── Authentication
│   │   ├── NTLM
│   │   ├── Kerberos
│   │   ├── Tickets
│   │   │   ├── TGT
│   │   │   └── TGS
│   │   └── LDAP
│   │
│   ├── AD Enumeration
│   │   ├── BloodHound
│   │   ├── PowerView
│   │   ├── LDAP Queries
│   │   ├── SharpHound
│   │   └── Manual Enumeration
│   │
│   ├── Credential Attacks
│   │   ├── Password Spraying
│   │   ├── Kerberoasting
│   │   ├── AS-REP Roasting
│   │   └── Credential Dumping
│   │
│   ├── Privilege Escalation
│   │   ├── ACL Abuse
│   │   ├── Delegation
│   │   ├── GPO Abuse
│   │   ├── AD CS
│   │   └── Misconfigurations
│   │
│   └── Domain Dominance
│       ├── Domain Admin
│       ├── DCSync
│       ├── Golden Ticket
│       └── Persistence
│
├── 5. INTERNAL NETWORK ATTACKS
│   │
│   ├── Network Enumeration
│   │   ├── Nmap
│   │   ├── SMB
│   │   ├── LDAP
│   │   └── SNMP
│   │
│   ├── Lateral Movement
│   │   ├── SMB
│   │   ├── PsExec
│   │   ├── WMI
│   │   ├── WinRM
│   │   ├── RDP
│   │   └── SSH
│   │
│   ├── Pivoting
│   │   ├── SOCKS Proxy
│   │   ├── SSH Tunneling
│   │   ├── Port Forwarding
│   │   └── Proxychains
│   │
│   └── C2 Communication
│       ├── HTTP/HTTPS
│       ├── DNS
│       ├── SMB
│       └── Custom Channels
│
├── 6. RED TEAM OPERATIONS
│   │
│   ├── MITRE ATT&CK ⭐⭐⭐⭐⭐
│   │   ├── Reconnaissance
│   │   ├── Resource Development
│   │   ├── Initial Access
│   │   ├── Execution
│   │   ├── Persistence
│   │   ├── Privilege Escalation
│   │   ├── Defense Evasion
│   │   ├── Credential Access
│   │   ├── Discovery
│   │   ├── Lateral Movement
│   │   ├── Collection
│   │   ├── Command & Control
│   │   ├── Exfiltration
│   │   └── Impact
│   │
│   ├── Engagement Planning
│   │   ├── Scope
│   │   ├── Rules of Engagement
│   │   ├── Objectives
│   │   └── OPSEC
│   │
│   ├── Attack Path
│   │   ├── External Attack Surface
│   │   ├── Initial Access
│   │   ├── Internal Pivot
│   │   └── Objective
│   │
│   └── Reporting
│       ├── Executive Report
│       ├── Technical Report
│       ├── Attack Narrative
│       └── Detection Recommendations
│
├── 7. COMMAND & CONTROL (C2)
│   │
│   ├── Concepts
│   │   ├── Implant / Agent
│   │   ├── Listener
│   │   ├── Beacon
│   │   ├── Team Server
│   │   └── Infrastructure
│   │
│   ├── C2 Frameworks
│   │   ├── Cobalt Strike
│   │   ├── Sliver
│   │   ├── Mythic
│   │   └── Havoc
│   │
│   ├── Payload Concepts
│   │   ├── Staged
│   │   ├── Stageless
│   │   ├── Shellcode
│   │   └── Loaders
│   │
│   └── Communication
│       ├── HTTP(S)
│       ├── DNS
│       ├── SMB
│       └── Redirectors
│
├── 8. WINDOWS INTERNALS ⭐⭐⭐⭐⭐
│   │
│   ├── Architecture
│   │   ├── User Mode
│   │   └── Kernel Mode
│   │
│   ├── Processes
│   │   ├── Process
│   │   ├── Thread
│   │   └── Handles
│   │
│   ├── Memory
│   │   ├── Virtual Memory
│   │   ├── Heap
│   │   ├── Stack
│   │   └── Memory Protection
│   │
│   ├── Windows API
│   ├── DLL
│   ├── PE Format
│   │   ├── DOS Header
│   │   ├── NT Header
│   │   ├── Sections
│   │   ├── Import Table
│   │   └── Export Table
│   │
│   └── Security
│       ├── Access Token
│       ├── SID
│       ├── Integrity Level
│       └── UAC
│
├── 9. DEFENSE EVASION / OPSEC
│   │
│   ├── Detection Awareness
│   │   ├── Antivirus
│   │   ├── EDR
│   │   ├── SIEM
│   │   └── Logging
│   │
│   ├── Windows Telemetry
│   │   ├── Event Logs
│   │   ├── Sysmon
│   │   ├── PowerShell Logging
│   │   └── ETW
│   │
│   ├── OPSEC
│   │   ├── Infrastructure Separation
│   │   ├── Identity Separation
│   │   ├── Traffic Awareness
│   │   └── Artifact Awareness
│   │
│   └── Detection Engineering Knowledge
│       ├── Understand What Blue Team Sees
│       ├── Sigma Rules
│       ├── SIEM Queries
│       └── IOC vs Behavior Detection
│
├── 10. CREDENTIAL ACCESS
│   │
│   ├── Windows Credentials
│   │   ├── SAM
│   │   ├── NTDS.dit
│   │   ├── LSASS
│   │   └── Cached Credentials
│   │
│   ├── Browser Credentials
│   ├── Password Managers
│   ├── Kerberos Tickets
│   └── Tokens / Secrets
│
├── 11. PERSISTENCE
│   │
│   ├── Windows
│   │   ├── Services
│   │   ├── Scheduled Tasks
│   │   ├── Registry
│   │   ├── Startup
│   │   ├── WMI
│   │   └── AD Persistence
│   │
│   └── Linux
│       ├── SSH Keys
│       ├── Cron
│       ├── systemd
│       └── Startup Scripts
│
├── 12. REVERSE ENGINEERING
│   │
│   ├── Assembly
│   │   ├── x86
│   │   ├── x64
│   │   └── ARM (Optional)
│   │
│   ├── Tools
│   │   ├── Ghidra
│   │   ├── IDA
│   │   ├── x64dbg
│   │   └── WinDbg
│   │
│   ├── Concepts
│   │   ├── Calling Convention
│   │   ├── Stack
│   │   ├── Registers
│   │   ├── Control Flow
│   │   └── API Analysis
│   │
│   └── Malware Analysis Basics
│       ├── Static Analysis
│       ├── Dynamic Analysis
│       └── Behavioral Analysis
│
├── 13. CLOUD RED TEAM
│   │
│   ├── AWS
│   │   ├── IAM
│   │   ├── EC2
│   │   ├── S3
│   │   ├── Lambda
│   │   └── CloudTrail
│   │
│   ├── Azure ⭐⭐⭐⭐⭐
│   │   ├── Azure AD / Entra ID
│   │   ├── RBAC
│   │   ├── Managed Identity
│   │   ├── Azure VMs
│   │   └── Hybrid Identity
│   │
│   ├── GCP
│   └── Cloud Enumeration
│
├── 14. SOCIAL ENGINEERING
│   │
│   ├── OSINT
│   ├── Phishing Awareness
│   ├── Pretexting Concepts
│   └── Human Attack Surface
│
├── 15. RED TEAM INFRASTRUCTURE
│   │
│   ├── VPS
│   ├── Domains
│   ├── DNS
│   ├── Reverse Proxy
│   ├── Redirectors
│   ├── HTTPS Certificates
│   └── Infrastructure Separation
│
├── 16. AI + RED TEAM ⭐⭐⭐⭐⭐
│   │
│   ├── AI-assisted Recon
│   ├── Code Analysis
│   ├── Log Analysis
│   ├── Automation
│   ├── Script Generation
│   ├── Attack Path Analysis
│   ├── Vulnerability Research
│   ├── Custom Agents
│   └── AI Security
│       ├── Prompt Injection
│       ├── LLM Data Exposure
│       ├── Agent Security
│       └── AI Attack Surface
│
├── 17. PRACTICAL LABS
│   │
│   ├── Beginner
│   │   ├── TryHackMe Jr Penetration Tester
│   │   ├── Linux Fundamentals
│   │   ├── Windows Fundamentals
│   │   └── Networking
│   │
│   ├── Intermediate
│   │   ├── Hack The Box
│   │   ├── PortSwigger
│   │   └── Active Directory Labs
│   │
│   ├── Advanced
│   │   ├── HTB Pro Labs
│   │   ├── Enterprise AD Labs
│   │   └── Purple Team Labs
│   │
│   └── CTF
│       ├── Web
│       ├── Pwn
│       ├── Crypto
│       ├── Reverse Engineering
│       └── Forensics
│
├── 18. CERTIFICATION PATH
│   │
│   ├── Foundation
│   │   ├── Security+
│   │   └── eJPT
│   │
│   ├── Pentest
│   │   ├── PNPT
│   │   ├── CPTS
│   │   └── OSCP
│   │
│   ├── Red Team
│   │   ├── CRTO
│   │   ├── CRTO II
│   │   └── Advanced Red Team Certifications
│   │
│   └── Specialist
│       ├── Cloud Security
│       ├── AD Security
│       └── Exploit Development
│
└── FINAL SKILL — RED TEAM OPERATOR
    │
    ├── Understand Infrastructure
    ├── Find Attack Surface
    ├── Gain Initial Access
    ├── Enumerate Environment
    ├── Escalate Privileges
    ├── Steal / Access Credentials
    ├── Move Laterally
    ├── Pivot Across Networks
    ├── Reach Objective
    ├── Maintain Operational Security
    ├── Understand Detection
    └── Write Professional Attack Narrative
```

**Thứ tự học thực tế nên ưu tiên cho bạn:**

```text
Networking + Linux + Windows
        ↓
Python + Bash + PowerShell
        ↓
Web Pentest + PortSwigger
        ↓
Pentest Methodology + Enumeration
        ↓
Active Directory ⭐
        ↓
Internal Network + Pivoting
        ↓
MITRE ATT&CK
        ↓
C2 Frameworks
        ↓
Windows Internals + .NET
        ↓
Detection / EDR Awareness + OPSEC
        ↓
Red Team Operations
        ↓
Cloud / Advanced AD / Malware / RE
```

Với nền hiện tại của bạn đang học **Web Security + CTF + Forensics**, hướng hiệu quả nhất là: **Web Pentest → Active Directory → Internal Pentest → Red Team Operations**. Không nên nhảy ngay vào Cobalt Strike, malware development hay defense evasion khi chưa làm chắc AD và Windows internals.