Đây, tao vẽ cho mày sơ đồ quy trình chuẩn để không bị ngợp. Cứ nhìn cái này mà đi từng bước một:

```mermaid
graph TD
    A[Bắt đầu: Có IP Target] --> B[Bước 1: RECON]
    B -->|nmap -sC -sV -p-| C{Phát hiện dịch vụ gì?}
    
    C -->|Web 80/443| D[Enum Web: Gobuster, dirb, xem source]
    C -->|SMB 445| E[Enum SMB: smbclient -L, enum4linux]
    C -->|SSH/FTP| F[Enum SSH/FTP: default creds, anonymous login]
    
    D --> G{Thấy lỗ hổng?}
    E --> G
    F --> G
    
    G -->|Chưa thấy| B
    G -->|Có rồi| H[Bước 2 & 3: KHAI THÁC]
    
    H -->|Web Upload / RCE| I[Nhận Shell]
    H -->|SMB psexec / Lỗi dịch vụ| I
    H -->|Có Credentials| I
    
    I --> J[Bước 4: HẬU KHAI THÁC]
    J --> K[Ổn định Shell: Python PTY / rlwrap]
    K --> L[Leo quyền: sudo -l, winPEAS, getsystem]
    L --> M[Tìm Flag: search -f flag*]
    M --> N{Có máy khác?}
    N -->|Có| A
    N -->|Không| O((Xong!))
    
    style A fill:#f9f,stroke:#333,stroke-width:2px
    style O fill:#9f9,stroke:#333,stroke-width:2px
    style G fill:#ff9,stroke:#333,stroke-width:2px
```

1- Cách nhìn sơ đồ: Mày cứ đi từ trên xuống. Gặp nút hình thoi (dấu hỏi) thì dừng lại tự hỏi "Có hay không?". Nếu "Chưa thấy" thì quay lại bước 1 (RECON) làm lại.

2- Ví dụ thực tế: 
- nmap thấy port 80 -> vào Web dò.
- Gobuster ra file `upload.php` -> thử upload shell.
- Nhận shell -> ổn định -> leo root -> lấy cờ -> quét tiếp máy khác.

3- Mẹo để không ngợp: Đừng bao giờ chuyển bước khi chưa ghi chép lại. Đang ở bước nào, làm xong chưa, nghi ngờ gì. Cứ giấy bút mà táng. Làm 5-10 box là tự khắc thành phản xạ.

