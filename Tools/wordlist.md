
## 1. PASSWORD — Common Credentials (phổ biến nhất)

Thư mục: `/usr/share/seclists/Passwords/Common-Credentials/`

| Đường dẫn                                                          | Ghi chú                                 |
| :----------------------------------------------------------------- | :-------------------------------------- |
| `10k-most-common.txt`                                              | 10k mật khẩu phổ biến nhất              |
| `xato-net-10-million-passwords-10.txt`                             | Top 10                                  |
| `xato-net-10-million-passwords-100.txt`                            | Top 100                                 |
| `xato-net-10-million-passwords-1000.txt`                           | Top 1000 (nhanh)                        |
| `xato-net-10-million-passwords-10000.txt`                          | Top 10,000                              |
| `xato-net-10-million-passwords-100000.txt`                         | Top 100,000                             |
| `xato-net-10-million-passwords-1000000.txt`                        | Top 1,000,000                           |
| `xato-net-10-million-passwords.txt`                                | Toàn bộ 10 triệu                        |
| `xato-net-10-million-passwords-dup.txt`                            | Bản có trùng lặp                        |
| `darkweb2017_top-10.txt` ... `darkweb2017_top-10000.txt`           | Top mật khẩu darkweb 2017               |
| `probable-v2_top-207.txt`                                          | Top 207                                 |
| `probable-v2_top-1575.txt`                                         | Top 1575                                |
| `probable-v2_top-12000.txt`                                        | Top 12,000                              |
| `Pwdb_top-1000.txt` ... `Pwdb_top-10000000.txt`                    | Bộ Pwdb nhiều cấp độ (bản lớn nhất 10M) |
| `100k-most-used-passwords-NCSC.txt`                                | NCSC 100k                               |
| `top-passwords-shortlist.txt`                                      | Danh sách ngắn                          |
| `top-20-common-SSH-passwords.txt`                                  | Top SSH                                 |
| `2023-200_most_used_passwords.txt` / `2024-197...` / `2025-199...` | Theo năm                                |
| `1900-2020.txt`                                                    | Mật khẩu theo năm                       |
| `common-passwords-win.txt`                                         | Windows                                 |
| `500-worst-passwords.txt`                                          | 500 tệ nhất                             |
| `worst-passwords-2017-top100-slashdata.txt`                        | Worst 2017                              |
| `medical-devices.txt`                                              | Thiết bị y tế                           |

> **Mẹo:** dùng `xato-net-10-million-passwords-1000.txt` ≈ 999 dòng (~6s) để brute thử nhanh, rồi nâng `-10000` nếu cần.

---

## 2. PASSWORD — Default Credentials (thiết bị/dịch vụ mặc định)

Thư mục: `/usr/share/seclists/Passwords/Default-Credentials/`

| Đường dẫn | Dịch vụ |
| :--- | :--- |
| `default-passwords.txt` | Tổng hợp chung |
| `cirt-net_collection.txt` | Bộ sưu tập CIRT |
| `ssh-betterdefaultpasslist.txt` | SSH |
| `ftp-betterdefaultpasslist.txt` | FTP |
| `tomcat-betterdefaultpasslist.txt` | Apache Tomcat |
| `tomcat-betterdefaultpasslist_base64encoded.txt` | Tomcat base64 |
| `mysql-betterdefaultpasslist.txt` | MySQL |
| `mssql-betterdefaultpasslist.txt` | MSSQL |
| `postgres-betterdefaultpasslist.txt` | PostgreSQL |
| `oracle-betterdefaultpasslist.txt` | Oracle |
| `db2-betterdefaultpasslist.txt` | IBM DB2 |
| `windows-betterdefaultpasslist.txt` | Windows |
| `vnc-betterdefaultpasslist.txt` | VNC |
| `telnet-betterdefaultpasslist.txt` | Telnet |
| `telnet-phenoelit.txt` | Telnet (phenoelit) |
| `citrix.txt` | Citrix |
| `cryptominers.txt` | Cryptominer |
| `avaya_defaultpasslist.txt` | Avaya |

---

## 3. PASSWORD — Leaked Databases (rockyou & các vụ rò rỉ)

Thư mục: `/usr/share/seclists/Passwords/Leaked-Databases/`

| Đường dẫn | Ghi chú |
| :--- | :--- |
| `rockyou-05.txt` → `rockyou-75.txt` | **rockyou phân đoạn** (05–75 = mức "ratio"). Không có rockyou.txt đầy đủ. |
| `phpbb.txt` | PHPBB |
| `myspace.txt` | MySpace |
| `000webhost.txt` | 000webhost |
| `adobe100.txt` | Adobe |
| `alleged-gmail-passwords.txt` | Gmail |
| `Ashley-Madison.txt` | Ashley Madison |
| `hotmail.txt` | Hotmail |
| `twitter-banned.txt` | Twitter |
| `elitehacker.txt` / `hak5.txt` / `honeynet.txt` | Bộ hacker |
| `fortinet-2021_passwords.txt` | Fortinet 2021 |
| `NordVPN.txt` | NordVPN |
| `Lizard-Squad.txt` | Lizard Squad |
| `carders.cc.txt` | Carders |

> **Lưu ý rockyou:** chỉ có phân đoạn (`rockyou-05` = 104 bytes, nhỏ; `rockyou-75` = 478KB). Không có bản đầy đủ 14M dòng.

---

## 4. PASSWORD — Chuyên dụng khác

| Đường dẫn | Ghi chú |
| :--- | :--- |
| `/usr/share/seclists/Passwords/corporate_passwords.txt` | Mật khẩu corporate (rất hợp lab "Support") |
| `/usr/share/seclists/Passwords/openwall.net-all.txt` | Openwall all |
| `/usr/share/seclists/Passwords/clarkson-university-82.txt` | Đại học Clarkson |
| `/usr/share/seclists/Passwords/darkc0de.txt` | darkc0de |
| `/usr/share/seclists/Passwords/unkown-azul.txt` | azul |
| `/usr/share/seclists/Passwords/stupid-ones-in-production.txt` | Mật khẩu "ngớ ngẩn" |
| `/usr/share/seclists/Passwords/mssql-passwords-nansh0u-guardicore.txt` | MSSQL |
| `/usr/share/seclists/Passwords/days.txt` / `months.txt` / `seasons.txt` | Ngày/tháng/mùa |
| `/usr/share/seclists/Passwords/Malware/mirai-botnet.txt` | Mirai botnet |
| `/usr/share/seclists/Passwords/Malware/conficker.txt` | Conficker |
| `/usr/share/seclists/Passwords/Software/john-the-ripper.txt` | JtR |
| `/usr/share/seclists/Passwords/Software/cain-and-abel.txt` | Cain & Abel |
| `/usr/share/seclists/Passwords/Software/bt4-password.txt` | BT4 |
| `/usr/share/seclists/Passwords/Cracked-Hashes/milw0rm-dictionary.txt` | milw0rm dictionary |
| `/usr/share/seclists/Passwords/Permutations/password-permutations.txt` | Permutations |
| `/usr/share/seclists/Passwords/Permutations/1337speak.txt` | 1337 speak |
| `/usr/share/seclists/Passwords/Keyboard-Walks/Keyboard-Combinations.txt` | Keyboard walks |
| `/usr/share/seclists/Passwords/Keyboard-Walks/walk-the-line.txt` | Walk the line |
| `/usr/share/seclists/Passwords/Most-Popular-Letter-Passes.txt` | Chữ phổ biến |
| `/usr/share/seclists/Passwords/WiFi-WPA/probable-v2-wpa-top447.txt` | WPA-2 WiFi |
| `/usr/share/seclists/Passwords/Honeypot-Captures/Sucuri-Top-Wordpress-Passwords.txt` | WordPress |
| `/usr/share/seclists/Passwords/Honeypot-Captures/multiplesources-passwords-fabian-fingerle.de.txt` | Honeypot tổng hợp |

---

## 5. USERNAMES

Thư mục: `/usr/share/seclists/Usernames/`

| Đường dẫn | Ghi chú |
| :--- | :--- |
| `top-usernames-shortlist.txt` | Danh sách ngắn (nhanh) |
| `xato-net-10-million-usernames.txt` | 10 triệu usernames |
| `xato-net-10-million-usernames-dup.txt` | Bản trùng |
| `cirt-default-usernames.txt` | Default CIRT |
| `CommonAdminBase64.txt` | Admin base64 |
| `sap-default-usernames.txt` | SAP |
| `mssql-usernames-nansh0u-guardicore.txt` | MSSQL |
| `names.txt` | Tên gọi |
| `Names/malenames-usa-top1000.txt` | Tên nam (Mỹ) |
| `Names/femalenames-usa-top1000.txt` | Tên nữ (Mỹ) |
| `Names/familynames-usa-top1000.txt` | Tên họ (Mỹ) |
| `Names/forenames-india-top1000.txt` | Tên Ấn Độ |
| `Names/names-brazil-top100000.txt` | Tên Brazil |
| `Honeypot-Captures/multiplesources-users-fabian-fingerle.de.txt` | Honeypot users |

---

## 6. WEB DISCOVERY (dùng cho gobuster / ffuf / dirsearch)

Thư mục: `/usr/share/seclists/Discovery/Web-Content/`

### Dir buster / tổng hợp
| Đường dẫn | Ghi chú |
| :--- | :--- |
| `common.txt` | Phổ biến (bắt đầu nên dùng) |
| `big.txt` | Lớn hơn |
| `combined_directories.txt` | Gộp directories |
| `combined_words.txt` | Gộp words |
| `common_directories.txt` | Directories thường |
| `common-api-endpoints-mazen160.txt` | Endpoint API |
| `Common-DB-Backups.txt` | Backup DB |
| `DirBuster-2007_directory-list-2.3-small.txt` | DirBuster small |
| `DirBuster-2007_directory-list-2.3-medium.txt` | DirBuster medium |
| `DirBuster-2007_directory-list-2.3-big.txt` | DirBuster big |
| `quickhits.txt` | Quick hits |

### Raft (dùng riêng file / directory / extensions, có bản lowercase)
| Đường dẫn | Ghi chú |
| :--- | :--- |
| `raft-small-files.txt` / `raft-small-directories.txt` / `raft-small-words.txt` | Small |
| `raft-medium-files.txt` / `raft-medium-directories.txt` / `raft-medium-words.txt` | Medium |
| `raft-large-files.txt` / `raft-large-directories.txt` / `raft-large-words.txt` | Large |
| `raft-*-extensions.txt` | Extensions (php, asp...) |
| (thêm hậu tố `-lowercase` cho bản chữ thường) | |

> **Mẹo:** `gobuster dir -u URL -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt -x php,txt,html,bak`

### Fuzz / mở rộng
| Đường dẫn | Ghi chú |
| :--- | :--- |
| `web-extensions.txt` | Đuôi web |
| `web-extensions-big.txt` | Đuôi web lớn |
| `web-all-content-types.txt` | Content types |
| `web-mutations.txt` | Mutations |
| `burp-parameter-names.txt` | Parameter names |
| `url-params_from-top-55-most-popular-apps.txt` | URL params |
| `graphql.txt` | GraphQL |
| `Logins.fuzz.txt` | Login paths |
| `Passwords.fuzz.txt` | Password paths |
| `LinuxFileList.txt` | File Linux |
| `UnixDotfiles.fuzz.txt` | Dotfiles Unix |
| `versioning_metafiles.txt` | Metafiles |
| `dsstorewordlist.txt` | .DS_Store |
| `default-web-root-directory-linux.txt` | Document root Linux |
| `default-web-root-directory-windows.txt` | Document root Windows |
| `vulnerability-scan_j2ee-websites_WEB-INF.txt` | J2EE WEB-INF |

### Công nghệ cụ thể
| Đường dẫn | Ghi chú |
| :--- | :--- |
| `JavaServlets-Common.fuzz.txt` | Java Servlets |
| `OracleAppServer.fuzz.txt` / `Oracle9i.fuzz.txt` | Oracle |
| `SAP-NetWeaver.txt` / `sap-analytics-cloud.txt` | SAP |
| `coldfusion.txt` | ColdFusion |
| `hashicorp-vault.txt` / `hashicorp-consul-api.txt` | HashiCorp |
| `oauth-oidc-scopes.txt` | OAuth/OIDC |
| `Microsoft-Frontpage.txt` | FrontPage |
| `ntlm-directories.txt` | NTLM |
| `wso2-enterprise-integrator.txt` | WSO2 |
| `domino-dirs-coldfusion39.txt` / `domino-endpoints-coldfusion39.txt` | Domino |
| `AdobeXML.fuzz.txt` | Adobe XML |
| `mcp-server.txt` | MCP server |

---

## 7. DISCOVERY KHÁC (Non-Web)

| Đường dẫn | Ghi chú |
| :--- | :--- |
| `/usr/share/seclists/Discovery/DNS/` | DNS subdomain brute |
| `/usr/share/seclists/Discovery/SNMP/` | SNMP community strings |
| `/usr/share/seclists/Discovery/Web_Content/` | (nếu có, bản cũ) |
| `/usr/share/seclists/Miscellaneous/` | lời mở đầu, DNS resolvers, ngôn ngữ |

---

## 8. CREDENTIAL COMBINATIONS & CÁC BỘ KHÁC

| Đường dẫn | Ghi chú |
| :--- | :--- |
| `/usr/share/seclists/Passwords/Default-Credentials/` | Kết hợp user:pass cho services |
| `/usr/share/seclists/Usernames/CommonAdminBase64.txt` | Admin e.g. base64 |
| `/usr/share/seclists/Passwords/scraped-JWT-secrets.txt` | JWT secrets |
| `/usr/share/seclists/Passwords/Books/` | Mật khẩu theo sách (leet variants...) |

---

## 9. GỢI Ý DÙNG NHANH (cheat-sheet)

```bash
# Brute login web (nhanh, ~6s với top-1000)
hydra -l help@support.thm -P /usr/share/seclists/Passwords/Common-Credentials/xato-net-10-million-passwords-1000.txt $IP http-post-form "/:email=^USER^&password=^PASS^:Invalid credentials"

# Directory brute web
gobuster dir -u http://$IP/ -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,txt,html,bak

# ffuf subdomain / vhost
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://$IP -H "Host: FUZZ.$IP"

# Crack hash (john)
john --wordlist=/usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt hash.txt
```

---

## 10. RÚT GỌN — CÁC FILE HAY DÙNG NHẤT

1. **Password top-1000:** `/usr/share/seclists/Passwords/Common-Credentials/xato-net-10-million-passwords-1000.txt`
2. **Password 10k:** `/usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt`
3. **Password corporate (lab THM):** `/usr/share/seclists/Passwords/corporate_passwords.txt`
4. **Username ngắn:** `/usr/share/seclists/Usernames/top-usernames-shortlist.txt`
5. **Web dir common:** `/usr/share/seclists/Discovery/Web-Content/common.txt`
6. **Web file raft medium:** `/usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt`
7. **Web extensions:** `/usr/share/seclists/Discovery/Web-Content/web-extensions.txt`
8. **rockyou (phân đoạn):** `/usr/share/seclists/Passwords/Leaked-Databases/rockyou-*.txt`