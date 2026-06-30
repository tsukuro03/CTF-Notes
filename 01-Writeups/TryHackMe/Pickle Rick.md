---
title: "TryHackMe-Pickle Rick"
author: Thien Nguyen
date: "2026-06-25"
keywords: [THM, CTF, TryHackMe, Security, Offensive]
lang: "vi"

---
# Recon
## Wallplyzer
![](../../05-Assets/Pasted%20image%2020260625213414.png)
## Nmap
```bash
sudo nmap -sC -sV -sS --reason -oA recon_nmap 10.48.189.107
# Nmap 7.95 scan initiated Thu Jun 25 10:31:46 2026 as: /usr/lib/nmap/nmap -sC -sV -sS --reason -oA recon_nmap 10.48.189.107
Nmap scan report for 10.48.189.107
Host is up, received reset ttl 62 (0.094s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 62 OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 81:86:42:c4:27:58:a4:4f:18:7f:d4:ed:70:17:db:c6 (RSA)
|   256 a4:a6:e5:06:4b:e2:9e:36:d9:98:6f:ac:bd:e2:80:66 (ECDSA)
|_  256 55:02:d9:61:d2:11:16:d4:87:03:e2:48:39:41:70:a2 (ED25519)
80/tcp open  http    syn-ack ttl 62 Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Rick is sup4r cool
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

| port    | service     | version                                                       |
| ------- | ----------- | ------------------------------------------------------------- |
| 22      | ssh         | OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0) |
| 80      | http        | Apache httpd 2.4.41 ((Ubuntu))                                |
# Enumeration
## gobuster
### dir
```bash
sudo gobuster dir -u http://10.48.189.107/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -o gobuster_directionary.txt
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.48.189.107/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/assets               (Status: 301) [Size: 315] [--> http://10.48.189.107/assets/]
/server-status        (Status: 403) [Size: 278]
Progress: 29999 / 29999 (100.00%)
===============================================================
Finished
===============================================================

```
### dns
```bash
sudo gobuster dns --domain 10.48.189.107 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -o gobuster-dns.txt  
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Domain:     10.48.189.107
[+] Threads:    10
[+] Timeout:    1s
[+] Wordlist:   /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
===============================================================
Starting gobuster in DNS enumeration mode
===============================================================
Progress: 635 / 4989 (12.73%)[ERROR] error on word b2b: lookup b2b.10.48.189.107.: i/o timeout
[ERROR] error on word www.store: lookup www.store.10.48.189.107.: i/o timeout
[ERROR] error on word m2: lookup m2.10.48.189.107.: i/o timeout
Progress: 3570 / 4989 (71.56%)[ERROR] error on word bid: lookup bid.10.48.189.107.: i/o timeout
[ERROR] error on word agri: lookup agri.10.48.189.107.: i/o timeout
[ERROR] error on word routernet: lookup routernet.10.48.189.107.: i/o timeout
Progress: 4989 / 4989 (100.00%)
===============================================================
Finished
===============================================================

```
## **Web Application Enumeration**.
![](../../05-Assets/Pasted%20image%2020260625220108.png)
--> Có 1 user name là R1ckRul3s
![](../../05-Assets/Pasted%20image%2020260625220208.png)
![](../../05-Assets/Pasted%20image%2020260625220220.png)
![](../../05-Assets/Pasted%20image%2020260625220258.png)

# Vulnerability Scanning
## nuclei
```bash
sudo nuclei -u 10.48.189.107                                                                                                     

                     __     _
   ____  __  _______/ /__  (_)
  / __ \/ / / / ___/ / _ \/ /
 / / / / /_/ / /__/ /  __/ /
/_/ /_/\__,_/\___/_/\___/_/   v3.6.1

                projectdiscovery.io

[INF] Your current nuclei-templates v10.3.5 are outdated. Latest is v10.4.5
[INF] cleaned up 16 orphaned template file(s)
[INF] Successfully updated nuclei-templates (v10.4.5) to /root/.local/nuclei-templates. GoodLuck!

Nuclei Templates v10.4.5 Changelog
┌───────┬───────┬──────────┬─────────┐
│ TOTAL │ ADDED │ MODIFIED │ REMOVED │
├───────┼───────┼──────────┼─────────┤
│ 5833  │ 1602  │ 4201     │ 30      │
└───────┴───────┴──────────┴─────────┘
[INF] Current nuclei version: v3.6.1 (outdated)
[INF] Current nuclei-templates version: v10.4.5 (latest)
[INF] New templates added in latest release: 86
[INF] Templates loaded for current scan: 10447
[INF] Executing 10430 signed templates from projectdiscovery/nuclei-templates
[WRN] Loading 17 unsigned templates for scan. Use with caution.
[INF] Targets loaded for current scan: 1
[INF] Running httpx on input host
[INF] Found 1 URL from httpx
[INF] Templates clustered: 2389 (Reduced 2256 Requests)
[INF] Using Interactsh Server: oast.fun
[waf-detect:apachegeneric] [http] [info] http://10.48.189.107
[ssh-server-enumeration] [javascript] [info] 10.48.189.107:22 ["SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.11"]
[ssh-sha1-hmac-algo] [javascript] [info] 10.48.189.107:22
[ssh-auth-methods] [javascript] [info] 10.48.189.107:22 ["["publickey"]"]
[openssh-detect] [tcp] [info] 10.48.189.107:22 ["SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.11"]
[http-missing-security-headers:strict-transport-security] [http] [info] http://10.48.189.107
[http-missing-security-headers:content-security-policy] [http] [info] http://10.48.189.107
[http-missing-security-headers:x-frame-options] [http] [info] http://10.48.189.107
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://10.48.189.107
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://10.48.189.107
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://10.48.189.107
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://10.48.189.107
[http-missing-security-headers:permissions-policy] [http] [info] http://10.48.189.107
[http-missing-security-headers:x-content-type-options] [http] [info] http://10.48.189.107
[http-missing-security-headers:referrer-policy] [http] [info] http://10.48.189.107
[tech-detect:bootstrap] [http] [info] http://10.48.189.107
[options-method] [http] [info] http://10.48.189.107 ["GET,POST,OPTIONS,HEAD"]
[apache-detect] [http] [info] http://10.48.189.107 ["Apache/2.4.41 (Ubuntu)"]
[INF] Scan completed in 1m. 18 matches found.
```
# Exploitation  

mr. meeseek hair





# Conclusion
In the conclusion sections I like to write a little bit about how the box seemed to me overall, where I struggled, and what I learned.

# References
1. [https://ryankozak.com/how-i-do-my-ctf-writeups/](https://ryankozak.com/how-i-do-my-ctf-writeups/)
2. [https://github.com/Wandmalfarbe/pandoc-latex-template](https://github.com/Wandmalfarbe/pandoc-latex-template)
3. [https://hackthebox.eu](https://hackthebox.eu)
4. [https://forum.hackthebox.eu](https://forum.hackthebox.eu)