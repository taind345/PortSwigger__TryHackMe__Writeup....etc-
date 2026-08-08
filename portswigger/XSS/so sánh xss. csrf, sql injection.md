### So sánh XSS, CSRF, SQL Injection

| Lỗ hổng       | Chạy ở đâu?              | Mục tiêu               |
| ------------- | ------------------------ | ---------------------- |
| XSS           | Client/browser           | Người dùng khác        |
| CSRF          | Server thực hiện request | Hành động của nạn nhân |
| SQL Injection | Server/database          | Database               |

Hiểu ngắn gọn:

```text
XSS:
Attacker → JavaScript → Browser nạn nhân

CSRF:
Attacker → Ép browser nạn nhân → Gửi request

SQLi:
Attacker → Input → Server → Database
```

---
