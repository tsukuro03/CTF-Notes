# 🕵️ OSINT Advanced Cheatsheet — Pentest & Skill Development Edition

> Tổng hợp kỹ thuật, workflow, lộ trình nâng cao trình độ từ IppSec, TCM Security, NahamSec, Bellingcat, Michael Bazzell & cộng đồng pentest / OSINT uy tín toàn cầu

---

## 📌 Mục lục

1. [1. Mindset & Tư duy OSINT](#1.%20Mindset%20&%20Tư%20duy%20OSINT)
2. [2. Lộ trình nâng cao trình độ](#2.%20Lộ%20trình%20nâng%20cao%20trình%20độ)
3. [3. Domain / DNS / Certificate Recon](#3.%20Domain%20/%20DNS%20/%20Certificate%20Recon)
4. [4. Email Harvesting & Verification](#4.%20Email%20Harvesting%20&%20Verification)
5. [5. Google Dorks & Search Engine OSINT](#5.%20Google%20Dorks%20&%20Search%20Engine%20OSINT)
6. [6. Shodan / Censys / FOFA / ZoomEye](#6.%20Shodan%20/%20Censys%20/%20FOFA%20/%20ZoomEye)
7. [7. Subdomain Enumeration](#7.%20Subdomain%20Enumeration)
8. [8. Web Technology Fingerprinting](#8.%20Web%20Technology%20Fingerprinting)
9. [9. GitHub / Code / Secret Recon](#9.%20GitHub%20/%20Code%20/%20Secret%20Recon)
10. [10. Metadata & Document Recon](#10.%20Metadata%20&%20Document%20Recon)
11. [11. Username & Social Media OSINT](#11.%20Username%20&%20Social%20Media%20OSINT)
12. [12. People & Identity OSINT](#12.%20People%20&%20Identity%20OSINT)
13. [13. Network / IP / ASN Recon](#13.%20Network%20/%20IP%20/%20ASN%20Recon)
14. [14. Wayback Machine & URL Recon](#14.%20Wayback%20Machine%20&%20URL%20Recon)
15. [15. Breach Data & Credential Recon](#15.%20Breach%20Data%20&%20Credential%20Recon)
16. [16. Image & Geolocation OSINT](#16.%20Image%20&%20Geolocation%20OSINT)
17. [17. Cloud & Infrastructure OSINT](#17.%20Cloud%20&%20Infrastructure%20OSINT)
18. [18. Automated OSINT Frameworks](#18.%20Automated%20OSINT%20Frameworks)
19. [19. Pentest OSINT Workflow Chuẩn](#19.%20Pentest%20OSINT%20Workflow%20Chuẩn)
20. [20. Bug Bounty OSINT Workflow](#20.%20Bug%20Bounty%20OSINT%20Workflow)
21. [21. Advanced Tips & Tricks](#21.%20Advanced%20Tips%20&%20Tricks)
22. [22. Tài nguyên & Cộng đồng học OSINT](#22.%20Tài%20nguyên%20&%20Cộng%20đồng%20học%20OSINT)

---

## 1. Mindset & Tư duy OSINT

### The OSINT Cycle (vòng lặp không bao giờ dừng)

```
  PLAN ──→ COLLECT ──→ PROCESS ──→ ANALYZE ──→ DISSEMINATE
    ↑                                                │
    └────────────────── FEEDBACK ───────────────────┘
```

### Nguyên tắc cốt lõi

**"Think like a stalker, act like an analyst"** — Michael Bazzell

> Đặt câu hỏi: _Nếu tôi muốn tìm thông tin về người này / tổ chức này, tôi sẽ để lại dấu vết ở đâu?_

**"OSINT is not about tools, it's about process"** — TCM Security

> Công cụ thay đổi, nhưng tư duy phân tích không thay đổi.

**"Pivot, pivot, pivot"** — NahamSec (Bug Bounty mindset)

> Mỗi thông tin tìm được là điểm xuất phát cho tìm kiếm tiếp theo. Email → username → social media → company → new domains...

### Phân loại OSINT theo mức độ tương tác

```
PASSIVE (an toàn nhất)
├── Google, Shodan, Censys, crt.sh
├── WHOIS history, DNS records
├── Wayback Machine, GitHub public
├── Social media public profiles
└── Breach database lookups

SEMI-PASSIVE (ít dấu vết)
├── DNS resolution (dùng resolver của mình)
├── HTTP header request (1 lần)
└── Certificate lookup trực tiếp

ACTIVE (để lại dấu vết — cần phép)
├── Port scanning (Nmap)
├── Directory brute-force (Gobuster)
├── DNS zone transfer attempt
└── Vulnerability scanning
```

### Checklist trước khi bắt đầu

```
□ Xác định scope: domain, IP range, công ty, cá nhân?
□ Xác định mục tiêu: credential, infrastructure, employee, tech stack?
□ Thiết lập môi trường ẩn danh nếu cần (VPN, VM, sock puppet)
□ Tạo workspace / folder structure để lưu output
□ Chọn tool phù hợp với loại target
□ Ghi chép mọi thứ — pivot trail rất quan trọng
```

---

## 2. Lộ trình nâng cao trình độ

### Level 1 — Beginner (0–3 tháng)

```
Mục tiêu: Hiểu khái niệm, thành thạo công cụ cơ bản

Học:
├── WHOIS, DNS records (dig, nslookup)
├── Google Dorks cơ bản
├── theHarvester, Maltego Community
├── Shodan basic search
├── ExifTool metadata
└── Subdomain enum với subfinder

Thực hành:
├── TryHackMe: "OSINT" learning path
├── CTF: PicoCTF (OSINT challenges)
└── Tự recon domain của chính mình

Tài nguyên:
└── TCM Security "Practical Ethical Hacking" (OSINT section)
```

### Level 2 — Intermediate (3–9 tháng)

```
Mục tiêu: Xây dựng workflow, kết hợp nhiều nguồn

Học:
├── Amass advanced (passive + active + viz)
├── SpiderFoot automation
├── GitHub dorks + Trufflehog + Gitleaks
├── Waybackurls + gau + URL analysis
├── Holehe, Maigret (identity OSINT)
├── Certificate Transparency khai thác sâu
├── Shodan + Censys API scripting
└── FOFA / ZoomEye cho Asian targets

Thực hành:
├── HackTheBox: các box có OSINT phase (Resolute, Forest...)
├── Bug Bounty: bắt đầu với scope rộng (*.example.com)
├── OSINT CTF: TraceLabs, Tracelabs Search Party
└── Tự build script tổng hợp output từ nhiều tool

Tài nguyên:
├── NahamSec Recon Series (YouTube)
├── Michael Bazzell "OSINT Techniques" book
└── Bellingcat "Online Investigation Toolkit"
```

### Level 3 — Advanced (9–18 tháng)

```
Mục tiêu: Custom tooling, niche techniques, adversary simulation

Học:
├── Viết custom Python script cho Shodan/Censys API
├── Build automated recon pipeline (bash/Python)
├── Advanced pivot techniques (IP → ASN → org → new IPs)
├── Dark web OSINT (Tor, I2P — cẩn thận pháp lý)
├── Cloud OSINT (S3, Azure Blob, GCS enumeration)
├── Mobile app OSINT (APK reverse, API endpoint extract)
├── HUMINT kết hợp OSINT (social engineering research)
└── Custom Maltego transforms

Thực hành:
├── TraceLabs OSINT CTF (thi đấu thực chiến)
├── HackerOne / Bugcrowd bug bounty (recon là core skill)
├── Red team engagement (với phép) — full kill chain
└── Đóng góp tool OSINT open source

Tài nguyên:
├── "Open Source Intelligence Techniques" — Michael Bazzell (mỗi năm update)
├── Bellingcat workshops
├── SANS FOR578 (OSINT course)
└── IppSec HTB walkthroughs (xem phần recon)
```

### Level 4 — Expert / Red Team OSINT

```
Mục tiêu: Intelligence-grade OSINT, adversary emulation

Kỹ năng:
├── Threat intelligence integration (MISP, OpenCTI)
├── Attribution analysis
├── Sock puppet management cho long-term ops
├── OPSEC cho OSINT operator
├── Custom scraping framework
├── Natural language processing trên OSINT data
└── Phân tích mạng lưới quan hệ (network graph analysis)
```

### Cách luyện tập hiệu quả nhất

```bash
# 1. Làm CTF OSINT thường xuyên
# https://ctftime.org  → filter "osint" category
# https://app.hackthebox.com → OSINT challenges
# https://tryhackme.com → OSINT rooms

# 2. Tự recon các bug bounty program (hợp pháp)
# https://www.hackerone.com/bug-bounty-programs
# https://www.bugcrowd.com/programs

# 3. Follow các researcher và đọc write-up
# https://pentester.land/list-of-bug-bounty-writeups.html
# https://github.com/nahamsec/Resources-for-Beginner-Bug-Bounty-Hunters

# 4. Xây dựng home lab OSINT
# - VM Kali/Parrot (isolated)
# - Note-taking: Obsidian / CherryTree / Notion
# - Bookmark manager: Raindrop.io

# 5. Theo dõi Twitter/X researchers
# @NahamSec, @TomNomNom, @jhaddix, @ippsec, @TCMSecurity
# @micahflee, @bellingcat, @osinttechniques
```

---

## 3. Domain / DNS / Certificate Recon

### WHOIS & Registration Data

```bash
whois example.com
whois 10.10.10.10

# Tìm tất cả domain cùng email đăng ký
# https://viewdns.info/reversewhois/?q=admin@example.com
# https://www.whoxy.com/reverse-whois/

# WHOIS history
# https://www.whoishistory.com
# https://domaintools.com  (trả phí nhưng mạnh nhất)
```

### DNS Records đầy đủ

```bash
# Một lệnh lấy tất cả
dig example.com ANY +noall +answer
for type in A AAAA MX NS TXT SOA CNAME CAA; do
  echo "=== $type ===" && dig example.com $type +short
done

# Kiểm tra SPF, DMARC, DKIM (email security → clue về mail server)
dig example.com TXT | grep -i "spf\|dmarc\|dkim"
dig _dmarc.example.com TXT
dig _domainkey.example.com TXT

# Zone Transfer
dig axfr @ns1.example.com example.com
for ns in $(dig example.com NS +short); do
  echo "=== Trying $ns ===" && dig axfr @$ns example.com
done
```

### Certificate Transparency — Kỹ thuật nâng cao

```bash
# crt.sh — nhiều kết quả nhất
curl -s "https://crt.sh/?q=%25.example.com&output=json" \
  | jq -r '.[].name_value' \
  | sed 's/\*\.//g' \
  | sort -u \
  | grep -v "^#"

# Lọc subdomain valid
curl -s "https://crt.sh/?q=%25.example.com&output=json" \
  | jq -r '.[].name_value' \
  | sed 's/\*\.//g' \
  | sort -u \
  | grep "\.example\.com$" > crt_subs.txt

# Facebook Certificate Transparency (dữ liệu riêng, khác crt.sh)
curl -s "https://graph.facebook.com/certificates?query=example.com&fields=domains&limit=10000" \
  | jq -r '.data[].domains[]' \
  | sort -u

# certspotter
curl -s "https://api.certspotter.com/v1/issuances?domain=example.com&include_subdomains=true&expand=dns_names" \
  | jq -r '.[].dns_names[]' \
  | sort -u
```

### Passive DNS — Tìm IP lịch sử

```bash
# SecurityTrails API (tốt nhất)
curl -s "https://api.securitytrails.com/v1/domain/example.com/dns/a" \
  -H "apikey: YOUR_KEY" | jq .

# VirusTotal
curl "https://www.virustotal.com/api/v3/domains/example.com/resolutions" \
  -H "x-apikey: YOUR_KEY" | jq '.data[].attributes.ip_address'

# Xem IP cũ trước khi dùng Cloudflare → tìm origin IP thật
# https://www.shodan.io → search "Ssl.cert.subject.cn:example.com"
```

---

## 4. Email Harvesting & Verification

### theHarvester nâng cao

```bash
# Tất cả nguồn
theHarvester -d example.com -b all -f output.html

# Nguồn mạnh nhất hiện tại
theHarvester -d example.com -b google,bing,linkedin,dnsdumpster,virustotal,certspotter,crtsh

# Dùng với proxy (tránh rate limit)
theHarvester -d example.com -b google -p  # dùng proxy list
```

### Hunter.io — Pattern Discovery

```bash
# Tìm email format của công ty
curl "https://api.hunter.io/v2/domain-search?domain=example.com&limit=100&api_key=KEY" \
  | jq -r '.data.emails[].value'

# Verify email cụ thể
curl "https://api.hunter.io/v2/email-verifier?email=john@example.com&api_key=KEY"

# Pattern thường gặp sau khi biết format:
# {first}.{last}@company.com
# {f}{last}@company.com
# {first}{l}@company.com
# → Combine với danh sách LinkedIn employees để generate email list
```

### LinkedIn → Email Generation Workflow

```bash
# 1. Tìm nhân viên trên LinkedIn
# https://www.linkedin.com/search/results/people/?keywords=company+name

# 2. Export tên → generate email
# https://www.linkedin2username.py
pip3 install linkedin2username  # hoặc clone từ GitHub
python3 linkedin2username.py -u YOUR_LINKEDIN -c "Company Name"

# 3. Verify email list
# https://app.proofy.io
# https://verify-email.org
# Bulk verify qua Hunter.io API

# 4. Dùng cho password spray (nếu trong scope)
```

### Holehe — Check Email Registration

```bash
pip3 install holehe
holehe email@example.com
# Kiểm tra email đã đăng ký dịch vụ nào: Twitter, Instagram, GitHub...
# Rất hữu ích để map identity

# Bonus: nếu dùng "Have I Been Pwned" style
curl "https://emailrep.io/email@example.com" -H "Key: YOUR_KEY"
```

---

## 5. Google Dorks & Search Engine OSINT

### Dorks theo mục tiêu

**Tìm file nhạy cảm:**

```
site:example.com filetype:pdf "confidential"
site:example.com filetype:xls OR filetype:xlsx
site:example.com filetype:sql
site:example.com filetype:env OR filetype:cfg OR filetype:conf
site:example.com filetype:log
site:example.com filetype:bak OR filetype:backup OR filetype:old
site:example.com ext:php.bak OR ext:asp.bak
site:example.com filetype:pem OR filetype:key OR filetype:ppk
```

**Login & Admin panels:**

```
site:example.com inurl:login OR inurl:signin OR inurl:admin
site:example.com intitle:"admin" OR intitle:"dashboard" OR intitle:"control panel"
site:example.com inurl:wp-admin OR inurl:wp-login
site:example.com inurl:joomla OR inurl:drupal
site:example.com inurl:phpmyadmin
site:example.com inurl:cpanel
site:example.com inurl:webmail
site:example.com inurl:portal
```

**Exposed directories:**

```
site:example.com intitle:"index of"
site:example.com intitle:"index of /" "parent directory"
site:example.com intitle:"index of" ".git"
site:example.com intitle:"index of" "backup"
site:example.com intitle:"index of" "config"
site:example.com intitle:"index of" "upload"
```

**Error messages / Debug info:**

```
site:example.com "Warning: mysql_"
site:example.com "Fatal error:"
site:example.com "Uncaught exception"
site:example.com "SQL syntax"
site:example.com "stack trace"
site:example.com "SQLSTATE"
site:example.com "ORA-" (Oracle errors)
site:example.com "Microsoft OLE DB Provider"
```

**Credentials leaked online:**

```
"@example.com" "password" site:pastebin.com
"@example.com" site:ghostbin.com
"@example.com" site:paste.ee
"example.com" "password" site:github.com
"example.com" api_key site:github.com
"example.com" "BEGIN RSA PRIVATE KEY" site:github.com
```

**Cloud storage:**

```
site:s3.amazonaws.com "example" OR "company-name"
site:blob.core.windows.net "example"
site:storage.googleapis.com "example"
site:drive.google.com "example.com" (nếu share public)
```

**Kỹ thuật nâng cao:**

```bash
# Tìm tất cả subdomain không phải www
site:*.example.com -site:www.example.com

# Tìm trang có form đăng nhập
site:example.com inurl:login filetype:php

# Tìm file với nội dung cụ thể (RDP, VNC files)
site:example.com filetype:rdp
site:example.com filetype:vnc

# Combine dorks
site:example.com (inurl:admin OR inurl:login) -inurl:help -inurl:blog
```

### Bing & DuckDuckGo Dorks

```bash
# Bing có thể index khác Google — dùng cả hai
# Bing: ip:10.10.10.10
# Bing: domain:example.com

# DuckDuckGo: tương tự Google nhưng không track
# Kết hợp với Startpage.com (Google results, no track)
```

### GHDB — Google Hacking Database

```
# https://www.exploit-db.com/google-hacking-database
# 6000+ dorks phân loại theo:
# - Files containing passwords
# - Sensitive directories
# - Web server detection
# - Vulnerable files
# - Error messages
# → Dùng khi cần dork chuyên biệt cho CMS, framework, device
```

---

## 6. Shodan / Censys / FOFA / ZoomEye

### Shodan — Nâng cao

```bash
# Cài & config
pip3 install shodan
shodan init YOUR_API_KEY

# Search cơ bản
shodan search "org:\"Company Name\""
shodan search "ssl.cert.subject.cn:\"example.com\""
shodan search "hostname:example.com"

# Tìm IP thật sau Cloudflare
shodan search "ssl:\"example.com\"" --fields ip_str,port,org,hostnames

# Tìm theo tech stack
shodan search "http.component:\"WordPress\" org:\"Company\""
shodan search "product:\"Apache httpd\" org:\"Company\""
shodan search "http.title:\"GitLab\" org:\"Company\""
shodan search "http.title:\"Jira\" country:VN"

# Tìm lỗ hổng đã biết
shodan search "vuln:CVE-2021-44228"    # Log4Shell
shodan search "vuln:CVE-2019-0708"     # BlueKeep RDP
shodan search "vuln:CVE-2021-26855"    # ProxyLogon Exchange

# Tìm thiết bị yếu
shodan search "\"default password\" port:23"
shodan search "port:5900 \"RFB 003.003\""  # VNC no auth
shodan search "port:27017 -authentication" # MongoDB open
shodan search "port:9200 -authentication" # Elasticsearch open
shodan search "port:6379 -authentication" # Redis open
shodan search "\"Set-Cookie: PHPSESSID\" country:VN"

# Shodan API scripting
python3 << 'EOF'
import shodan
api = shodan.Shodan("YOUR_KEY")
results = api.search('org:"Company Name"')
for r in results['matches']:
    print(f"{r['ip_str']}:{r['port']} - {r.get('org', 'N/A')}")
EOF

# Shodan Monitor (alert khi IP mới xuất hiện)
shodan alert create "company-monitor" "org:\"Company Name\""
```

### Censys nâng cao

```bash
pip3 install censys
censys config

# Search hosts
censys search "services.tls.certificates.leaf_data.subject.common_name:example.com" --index-type hosts

# Tìm theo ASN
censys search "autonomous_system.asn:12345" --index-type hosts

# Certificates
censys search "parsed.names: example.com" --index-type certificates

# Python API
python3 << 'EOF'
from censys.search import CensysHosts
h = CensysHosts()
for page in h.search("services.tls.certificates.leaf_data.subject.common_name:example.com"):
    for host in page:
        print(host["ip"], host.get("services", []))
EOF
```

### FOFA

```bash
# Cú pháp FOFA
domain="example.com"
cert="example.com"
cert.subject="O=Company Name"
header="X-Custom-Header"
body="unique_string_from_site"
title="Login - Company"
org="Company Name"
ip="10.10.10.0/24"
port="8443" && country="VN"
app="Cisco-IOS" && country="VN"
protocol="rdp" && country="VN"

# Tìm IP thật sau WAF/CDN
cert="example.com" && org!="Cloudflare"
cert="example.com" && org!="Fastly"
```

---

## 7. Subdomain Enumeration

### Passive — Nhiều nguồn nhất

```bash
# Subfinder với tất cả nguồn
subfinder -d example.com -all -recursive -o subfinder.txt

# Amass passive
amass enum -passive -d example.com \
  -config ~/.config/amass/config.ini \
  -o amass_passive.txt

# amass config.ini (thêm API keys)
# [data_sources.Shodan]
# [data_sources.SecurityTrails]
# [data_sources.VirusTotal]
# → Với nhiều API key, amass tìm được nhiều subdomain hơn

# Assetfinder
assetfinder --subs-only example.com

# crt.sh
curl -s "https://crt.sh/?q=%25.example.com&output=json" \
  | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u

# Findomain
findomain -t example.com -u findomain.txt

# GitHub subdomain search
curl -s "https://api.github.com/search/code?q=example.com+in:file&per_page=100" \
  -H "Authorization: token YOUR_TOKEN" \
  | jq -r '.items[].html_url'

# Kết hợp tất cả
cat subfinder.txt amass_passive.txt findomain.txt crt_subs.txt \
  | sort -u > all_passive_subs.txt
```

### Active — Brute-force & Permutation

```bash
# dnsx resolve và filter live
cat all_passive_subs.txt | dnsx -silent -o live_subs.txt

# Amass brute-force
amass enum -active -d example.com -brute \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt \
  -o amass_brute.txt

# Puredns (nhanh nhất cho brute-force)
puredns bruteforce \
  /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt \
  example.com \
  -r resolvers.txt \
  -w puredns.txt

# Permutation — QUAN TRỌNG (nhiều hacker bỏ qua)
# Từ subdomain đã biết → generate variations
# api.example.com → api-dev, api-staging, api-v2, api-test...
pip3 install dnsgen
cat live_subs.txt | dnsgen - | dnsx -silent >> more_subs.txt

# Gotator (permutation engine)
gotator -sub live_subs.txt -perm permutations.txt -depth 1 -numbers 3 | dnsx -silent
```

### Resolve & Probe

```bash
# Chỉ giữ subdomain đang sống
cat all_subs.txt | dnsx -silent -a -resp-only > ips.txt

# HTTP probe (tìm web service)
cat live_subs.txt | httpx -silent -status-code -title -tech-detect -o httpx.txt

# Screenshot tất cả (review nhanh)
gowitness file -f live_subs.txt -P screenshots/ --delay 2
eyewitness --web -f live_subs.txt --timeout 10 -d eyewitness/
```

---

## 8. Web Technology Fingerprinting

```bash
# WhatWeb aggressive
whatweb -a 3 -v http://example.com 2>/dev/null

# httpx tech detect
echo "http://example.com" | httpx -tech-detect -status-code -title -server -content-type

# Curl header analysis
curl -sI http://example.com | grep -i "server\|x-powered-by\|x-generator\|x-cms\|x-drupal\|x-wordpress"

# Wappalyzer CLI
npx wappalyzer https://example.com

# CMSeek (CMS detection)
python3 cmseek.py -u http://example.com

# Nikto basic fingerprint
nikto -h http://example.com -Tuning 0 -nossl

# Nmap service fingerprint
nmap -sV --version-intensity 9 -p 80,443,8080,8443 example.com
```

---

## 9. GitHub / Code / Secret Recon

### GitHub Dorks thực chiến

```bash
# Credential trong code
org:CompanyName "password" language:python
org:CompanyName "api_key" language:javascript
org:CompanyName "secret" extension:env
org:CompanyName "BEGIN RSA PRIVATE KEY"
org:CompanyName "BEGIN OPENSSH PRIVATE KEY"
org:CompanyName "AKIA" extension:py   # AWS Access Key
org:CompanyName "aws_secret_access_key"
org:CompanyName "smtp_password"
org:CompanyName "DB_PASSWORD"
org:CompanyName "database_password"
org:CompanyName "redis_password"

# Theo domain
"example.com" "password" language:python
"@example.com" extension:sql
"internal.example.com" (internal domains/IPs)

# File cụ thể
filename:.env "example.com"
filename:config.yml "example.com"
filename:secrets.yml
filename:id_rsa extension:pem
filename:.htpasswd
filename:wp-config.php "DB_PASSWORD"
filename:.npmrc "_authToken"
filename:.dockercfg OR filename:.docker/config.json
filename:*.pem "PRIVATE KEY"
path:/.ssh extension:pem

# Sensitive files
extension:sql "INSERT INTO" "users"
extension:log "password"
extension:txt "password" "username" "example.com"
```

### Trufflehog

```bash
# Scan repo cụ thể
trufflehog git https://github.com/company/repo --only-verified

# Scan toàn bộ org
trufflehog github --org=CompanyName --only-verified

# Scan local
trufflehog filesystem /path/to/repo --only-verified

# Docker image scan
trufflehog docker --image=company/image:latest

# Scan với output JSON
trufflehog git https://github.com/company/repo --json | jq .
```

### Gitleaks

```bash
gitleaks detect --source /path/to/repo -v
gitleaks detect --source . --report-format json --report-path leaks.json
gitleaks detect --source . --config custom_rules.toml

# Custom rule ví dụ
# [rules]
# [[rules.rules]]
# id = "company-internal-token"
# regex = '''COMP-[0-9a-f]{32}'''
```

### GitHub Actions / CI/CD secrets

```bash
# Kiểm tra workflow files cho leaked secrets
# https://github.com/company/repo/blob/main/.github/workflows/
# Tìm: env vars, secrets references, hardcoded values

# Tìm qua search
org:CompanyName filename:.github/workflows
org:CompanyName "secrets." path:.github
```

### Source code analysis sau khi có access

```bash
# Tìm hardcoded credential trong code
grep -rn "password\s*=\s*['\"]" . --include="*.py" --include="*.php" --include="*.js"
grep -rn "api_key\|apikey\|api-key" . --include="*.js" --include="*.py"
grep -rn "secret\|token\|credential" . --include="*.env" --include="*.conf"

# Tìm internal URLs / IPs
grep -rn "http://192\.\|http://10\.\|http://172\." . --include="*.js"
grep -rn "internal\.\|dev\.\|staging\." . --include="*.js" --include="*.py"
```

---

## 10. Metadata & Document Recon

### ExifTool — Toàn diện

```bash
# Metadata ảnh (GPS, camera, timestamp)
exiftool image.jpg
exiftool -a -u -g1 image.jpg  # Tất cả metadata, grouped

# GPS coordinates → Google Maps
exiftool -GPSLatitude -GPSLongitude image.jpg
# Output: 37 deg 26' 2.00" N, 122 deg 1' 13.00" W
# → Dán vào Google Maps

# Batch extract metadata thành CSV
exiftool -csv -r /path/to/folder/ > metadata.csv

# Metadata PDF (author, software, company)
exiftool document.pdf | grep -i "author\|creator\|producer\|company\|subject"

# Office documents
exiftool report.docx | grep -i "author\|last\|company\|revision"
# Lộ: tên user, company name, internal path, revision count
```

### Metagoofil — Thu thập docs từ Google

```bash
# Download và extract metadata từ docs public
metagoofil -d example.com -t pdf,doc,docx,xls,xlsx,ppt,pptx -l 100 -o docs/ -f metadata.html

# Thông tin thường lộ:
# - Username (author name → map với email format)
# - Software versions (→ check exploits)
# - Internal directory paths (C:\Users\john\...)
# - Company name, department
# - Network paths (\\server\share\...)
```

### FOCA (Windows GUI)

```bash
# https://github.com/ElevenPaths/FOCA
# Tự động:
# 1. Google / Bing cho docs của domain
# 2. Download tất cả
# 3. Extract metadata
# 4. Visualize: users, servers, software, emails, paths
# Kết quả thường: internal hostnames, usernames → AD username guessing
```

---

## 11. Username & Social Media OSINT

### Sherlock & Maigret

```bash
# Sherlock — 400+ sites
sherlock username --timeout 5 -o sherlock.txt
sherlock username --csv -o sherlock.csv
sherlock username --print-found  # chỉ in kết quả found

# Maigret — phân tích sâu hơn, report đẹp
maigret username
maigret username --html report.html --pdf report.pdf
maigret username --tags social,dating,tech  # filter theo category
```

### Twitter/X OSINT

```bash
# Advanced search operators
# from:username → tweet từ user
# to:username → tweet reply đến user
# @username → mention
# since:2020-01-01 until:2023-12-31
# near:"Ho Chi Minh City" within:15mi
# filter:media → chỉ tweet có ảnh/video
# filter:links → chỉ tweet có link
# -filter:retweets → bỏ retweet

# Ví dụ combination
from:username since:2023-01-01 filter:media

# Tools
# https://tweetdeck.twitter.com  ← Multi-column monitoring
# https://tinfoleak.com           ← Phân tích account
# https://twint (CLI scraper, không cần API)
pip3 install twint
twint -u username -o tweets.csv --csv
twint -u username --email --phone  # Tìm contact info
twint -s "example.com" -o search.csv --csv
```

### LinkedIn OSINT

```bash
# Google dork cho LinkedIn
site:linkedin.com/in "Company Name" "Current"
site:linkedin.com/in "example.com" "Senior Engineer"

# linkedin2username
python3 linkedin2username.py -u YOUR_EMAIL -c "Company Name" -d example.com -s 5

# CrossLinked (generate username permutations)
pip3 install crosslinked
crosslinked -f '{first}.{last}@example.com' "Company Name"
crosslinked -f '{f}{last}@example.com' "Company Name" -j  # output JSON

# PhoneBook.cz → tìm email từ domain
# https://phonebook.cz → search domain → emails
```

---

## 12. People & Identity OSINT

### Sock Puppet (OPSEC cơ bản khi recon)

```bash
# Khi cần nhìn profile private / không muốn lộ danh tính:
# 1. Tạo tài khoản fake (sock puppet) hoàn toàn tách biệt
# 2. Dùng VPN khác hoàn toàn với identity thật
# 3. Browser riêng (Firefox container / Chromium fresh profile)
# 4. Không bao giờ dùng sock puppet từ thiết bị thật
# 5. Build backstory cho account (post lịch sử, followers...)
# → Học thêm: "The Art of The Sock" — Michael Bazzell
```

### Phone Number OSINT

```bash
# PhoneInfoga
docker run --rm -it sundowndev/phoneinfoga scan -n "+84912345678"
docker run --rm -it -p 5000:5000 sundowndev/phoneinfoga serve

# Truecaller API (reverse lookup)
# https://www.truecaller.com/search/vn/+84912345678

# Carrier lookup
curl "https://api.telnyx.com/v2/number_lookup/+84912345678" -H "Authorization: Bearer KEY"
```

### Email → Identity Pivot

```bash
# Epieos (tốt nhất 2024)
# https://epieos.com → nhập email → xem: Google account, Calendar, Maps reviews...

# GHunt (Google account OSINT)
pip3 install ghunt
ghunt login  # auth bằng cookie
ghunt email someone@gmail.com

# Holehe (registered services)
holehe email@example.com --only-used

# Mosint (automated email OSINT)
go install github.com/alpkeskin/mosint@latest
mosint email@example.com
```

---

## 13. Network / IP / ASN Recon

### ASN Discovery — Pivot quan trọng

```bash
# Tìm ASN của công ty
curl "https://api.bgpview.io/search?query_term=Company+Name" | jq '.data.asns[]'
curl "https://ipinfo.io/org/AS12345" | head -5

# Tìm tất cả IP range của ASN
curl "https://api.bgpview.io/asn/AS12345/prefixes" | jq '.data.ipv4_prefixes[].prefix'

# amass intel từ ASN
amass intel -asn 12345 -o asn_domains.txt

# Shodan từ ASN
shodan search "asn:AS12345" --fields ip_str,port,org | head -50
```

### Reverse IP & Hosting

```bash
# Tìm tất cả domain cùng IP
curl "https://api.hackertarget.com/reverseiplookup/?q=10.10.10.10"

# Robtex
curl "https://freeapi.robtex.com/ipquery/10.10.10.10" | jq '.pas[].o'  # passive DNS

# ViewDNS
# https://viewdns.info/reverseip/?host=10.10.10.10&t=1
```

### IP History — Tìm origin IP sau CDN

```bash
# SecurityTrails
curl "https://api.securitytrails.com/v1/domain/example.com/dns/a" \
  -H "apikey: KEY" | jq '.records[].values[].ip'

# Shodan historical
shodan host 10.10.10.10 --history

# Nếu target dùng Cloudflare → tìm IP thật:
# 1. crt.sh → subdomain không có trong Cloudflare (mail.example.com, smtp.example.com)
# 2. Shodan: ssl:"example.com" → filter bỏ Cloudflare/Fastly IPs
# 3. MX record → thường trỏ thẳng đến IP thật
# 4. Email headers từ phishing/newsletter → Received: from IP
# 5. Lịch sử DNS (trước khi migrate sang CDN)
```

---

## 14. Wayback Machine & URL Recon

### Waybackurls + gau

```bash
# Waybackurls
go install github.com/tomnomnom/waybackurls@latest
waybackurls example.com | tee wayback.txt

# gau (nhiều nguồn hơn: Wayback + OTX + CommonCrawl)
go install github.com/lc/gau/v2/cmd/gau@latest
gau example.com | tee gau.txt
gau --subs example.com | tee gau_subs.txt

# Kết hợp
cat wayback.txt gau.txt | sort -u > all_urls.txt
```

### Phân tích URL — Tìm endpoint quý giá

```bash
# Tìm URL có parameter (potential injection points)
cat all_urls.txt | grep "?.*=" | sort -u > params.txt

# Tìm file extension cụ thể
cat all_urls.txt | grep "\.php$\|\.asp$\|\.aspx$\|\.jsp$" | sort -u
cat all_urls.txt | grep "\.json$\|\.xml$\|\.yaml$\|\.yml$" | sort -u
cat all_urls.txt | grep "\.sql$\|\.bak$\|\.backup$\|\.old$" | sort -u
cat all_urls.txt | grep "\.js$" | sort -u > js_files.txt

# Tìm API endpoints
cat all_urls.txt | grep "/api/\|/v1/\|/v2/\|/graphql\|/rest/" | sort -u

# Tìm admin / sensitive paths
cat all_urls.txt | grep -i "admin\|login\|dashboard\|config\|backup\|upload" | sort -u

# LinkFinder — extract endpoint từ JS files
pip3 install linkfinder
while read url; do
  python3 linkfinder.py -i "$url" -o cli 2>/dev/null
done < js_files.txt | sort -u > js_endpoints.txt

# SecretFinder — tìm secrets trong JS
python3 SecretFinder.py -i https://example.com/app.js -o cli
```

---

## 15. Breach Data & Credential Recon

### Have I Been Pwned

```bash
# Check email
curl "https://haveibeenpwned.com/api/v3/breachedaccount/email@example.com" \
  -H "hibp-api-key: YOUR_KEY" | jq .

# Check domain (tất cả email bị breach)
curl "https://haveibeenpwned.com/api/v3/breachedaccount/email@example.com?truncateResponse=false" \
  -H "hibp-api-key: YOUR_KEY"
```

### DeHashed

```bash
# Search by domain
curl 'https://api.dehashed.com/search?query=domain:example.com&size=100' \
  -u "email:api_key" -H 'Accept: application/json' | jq .

# Search by email
curl 'https://api.dehashed.com/search?query=email:john@example.com' \
  -u "email:api_key" -H 'Accept: application/json'

# Search by username
curl 'https://api.dehashed.com/search?query=username:johndoe' \
  -u "email:api_key" -H 'Accept: application/json'
```

### Breach Parse Workflow

```bash
# Nếu có local breach database
# ripgrep (nhanh hơn grep rất nhiều)
rg "example.com" /path/to/breach/ -o | head -100
rg "@example\.com" breach_data/ | cut -d: -f2 | sort -u > company_creds.txt

# Phân loại credential tìm được
cat company_creds.txt | grep -oP "(?<=:).+" | sort | uniq -c | sort -rn | head -20
# → Tìm password phổ biến nhất → dùng cho password spray

# Password spray với Kerbrute (Active Directory)
kerbrute passwordspray -d DOMAIN.LOCAL --dc 10.10.10.10 users.txt "Company2024!"
```

---

## 16. Image & Geolocation OSINT

### Reverse Image Search

```bash
# Thứ tự ưu tiên:
# 1. Yandex → tốt nhất cho face matching
# 2. Google Images → rộng nhất
# 3. TinEye → track ảnh theo thời gian
# 4. Bing Visual Search
# 5. PimEyes → face search chuyên biệt (trả phí)

# Tip: Upload ảnh trực tiếp vs dán URL cho kết quả khác nhau
```

### Geolocation từ ảnh

```bash
# Bước 1: ExifTool GPS
exiftool -n -GPSLatitude -GPSLongitude image.jpg
# → Dán vào: https://maps.google.com?q=LAT,LON

# Bước 2 (không có GPS): Phân tích visual
# - Biển số xe (format theo quốc gia/vùng)
# - Biển đường, biển hiệu cửa hàng (Google Maps search)
# - Kiến trúc đặc trưng (nhận diện style)
# - Thực vật (loại cây → khí hậu → khu vực)
# - Bóng đổ + góc mặt trời → thời gian + vĩ độ
# - Google Street View / Maps xác nhận

# Bước 3: Công cụ hỗ trợ
# https://www.geospy.ai  ← AI geolocation từ ảnh
# https://sunlocation.com  ← tính vị trí từ góc mặt trời
# https://www.suncalc.org  ← verify hướng mặt trời
# https://shadow.suncalc.org  ← phân tích bóng
```

### Video OSINT

```bash
# Keyframe extraction
ffmpeg -i video.mp4 -vf fps=1 frames/frame_%04d.jpg
# → Reverse search từng frame

# Metadata video
exiftool video.mp4 | grep -i "gps\|location\|date\|create"

# YouTube metadata
yt-dlp --dump-json "https://youtube.com/watch?v=ID" | jq '{title, upload_date, uploader, location}'
```

---

## 17. Cloud & Infrastructure OSINT

### AWS S3 Bucket Enumeration

```bash
# Gobuster S3
gobuster s3 -w /usr/share/seclists/Discovery/S3/bucket-names.txt

# lazys3
ruby lazys3.rb CompanyName

# s3scanner
pip3 install s3scanner
s3scanner scan --bucket-file buckets.txt
s3scanner scan --bucket company-name

# Generate bucket name guesses
echo -e "company\ncompany-dev\ncompany-prod\ncompany-staging\ncompany-backup\ncompany-data\ncompany-assets\ncompany-static" | s3scanner scan --bucket-file -

# Verify public bucket
aws s3 ls s3://bucket-name --no-sign-request
curl https://bucket-name.s3.amazonaws.com/
```

### Azure Blob Storage

```bash
# MicroBurst (Azure OSINT)
Import-Module MicroBurst.psm1
Invoke-EnumerateAzureBlobs -Base "companyname"

# Manual check
curl https://companyname.blob.core.windows.net/?comp=list
curl https://companyname.blob.core.windows.net/public?restype=container&comp=list
```

### Google Cloud Storage

```bash
curl https://storage.googleapis.com/company-bucket/
curl https://company-bucket.storage.googleapis.com/
```

### Cloud IP Range Mapping

```bash
# AWS IP ranges
curl https://ip-ranges.amazonaws.com/ip-ranges.json | jq '.prefixes[] | select(.region=="ap-southeast-1") | .ip_prefix'

# Azure IP ranges
# https://www.microsoft.com/en-us/download/details.aspx?id=56519

# GCP IP ranges
curl https://www.gstatic.com/ipranges/cloud.json | jq '.prefixes[].ipv4Prefix'
```

---

## 18. Automated OSINT Frameworks

### SpiderFoot

```bash
# Docker (dễ nhất)
docker pull smicallef/spiderfoot
docker run -p 5001:5001 smicallef/spiderfoot

# Pip
pip3 install spiderfoot
python3 sf.py -l 0.0.0.0:5001

# CLI scan
python3 sfcli.py -s example.com -m all -o csv > sf_output.csv
python3 sfcli.py -s example.com -t INTERNET_NAME,EMAILADDR,IP_ADDRESS

# Module hay dùng
# sfp_dns, sfp_shodan, sfp_censys, sfp_crt, sfp_github
# sfp_emailformat, sfp_haveibeenpwned, sfp_virustotal
# sfp_waybackmachine, sfp_hunter, sfp_linkedin
```

### Amass Intelligence

```bash
# Amass intel — tìm domains từ org name
amass intel -org "Company Name" -o amass_intel.txt
amass intel -asn 12345 -o amass_asn.txt
amass intel -cidr 10.10.10.0/24 -o amass_cidr.txt

# Amass với config (API keys)
amass enum -d example.com -config ~/.config/amass/config.ini -o amass.txt

# Amass database query
amass db -names -d example.com
amass db -show -d example.com
amass viz -d example.com -d3  # Visualize graph
```

### Recon-ng

```bash
recon-ng -w workspace_example

# Trong REPL:
[recon-ng][workspace_example] > marketplace install all
[recon-ng][workspace_example] > db insert domains
[recon-ng][workspace_example] > [domain] >> example.com

# Chạy module
[recon-ng][workspace_example] > modules load recon/domains-hosts/hackertarget
[recon-ng][workspace_example] > run

[recon-ng][workspace_example] > modules load recon/domains-contacts/whois_pocs
[recon-ng][workspace_example] > run

# Export
[recon-ng][workspace_example] > modules load reporting/html
[recon-ng][workspace_example] > options set FILENAME report.html
[recon-ng][workspace_example] > run
```

### Maltego

```bash
# Workflow chuẩn Maltego:
# 1. New graph → Add entity: Domain
# 2. Run transform: "To DNS Name [Using Search Engine]"
# 3. Run transform: "To IP Address [DNS]"
# 4. Run transform: "To Domains [Reverse DNS]" từ mỗi IP
# 5. Run transform: "To Email Address [PGP]"
# 6. Chạy Shodan transforms
# 7. Visualize toàn bộ attack surface

# Community Edition: free, 12 entities/graph
# Pro: unlimited
```

---

## 19. Pentest OSINT Workflow Chuẩn

### Pre-engagement (trước khi bắt đầu)

```bash
mkdir -p ~/pentest/CLIENT/{recon/{passive,active,screenshots},loot,notes,reports}
cd ~/pentest/CLIENT

# Ghi chép scope
cat > scope.txt << 'EOF'
Domain: example.com
IP Range: 10.10.10.0/24
Out of scope: test.example.com, *.partner.example.com
EOF
```

### Phase 1 — Passive OSINT (Ngày 1)

```bash
TARGET="example.com"
COMPANY="Company Name"

# 1. WHOIS & DNS
whois $TARGET > recon/passive/whois.txt
dig $TARGET ANY > recon/passive/dns.txt
for type in A AAAA MX NS TXT SOA; do dig $TARGET $type +short; done >> recon/passive/dns.txt

# 2. Certificate Transparency
curl -s "https://crt.sh/?q=%25.$TARGET&output=json" \
  | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u \
  > recon/passive/crt_subs.txt

# 3. Subdomain passive
subfinder -d $TARGET -all -silent -o recon/passive/subfinder.txt
amass enum -passive -d $TARGET -o recon/passive/amass.txt
assetfinder --subs-only $TARGET > recon/passive/assetfinder.txt

# 4. Email harvesting
theHarvester -d $TARGET -b google,bing,linkedin,certspotter,virustotal \
  -f recon/passive/emails.html > /dev/null 2>&1

# 5. Google Dorks (manual)
# site:*.$TARGET
# site:$TARGET filetype:pdf OR filetype:xls
# "@$TARGET" password site:pastebin.com

# 6. Shodan
shodan search "ssl:\"$TARGET\"" --fields ip_str,port,org > recon/passive/shodan.txt

# 7. GitHub search (manual)
# org:$COMPANY password
# "$TARGET" api_key

# 8. Wayback URLs
waybackurls $TARGET > recon/passive/wayback.txt
gau $TARGET --subs >> recon/passive/wayback.txt
cat recon/passive/wayback.txt | sort -u -o recon/passive/wayback.txt

# Tổng hợp subdomain
cat recon/passive/crt_subs.txt recon/passive/subfinder.txt \
    recon/passive/amass.txt recon/passive/assetfinder.txt \
  | sort -u > recon/passive/all_subs.txt

echo "[*] Passive recon done: $(wc -l < recon/passive/all_subs.txt) subdomains found"
```

### Phase 2 — Active Recon (Ngày 2)

```bash
# 1. Resolve subdomains
cat recon/passive/all_subs.txt | dnsx -silent -o recon/active/live_subs.txt
echo "[*] Live subs: $(wc -l < recon/active/live_subs.txt)"

# 2. HTTP probe
cat recon/active/live_subs.txt | httpx -silent -status-code -title -tech-detect \
  -o recon/active/httpx.txt

# 3. Screenshots
gowitness file -f recon/active/live_subs.txt -P recon/screenshots/ --delay 2

# 4. Port scan
nmap -iL recon/active/live_subs.txt -sC -sV -p 80,443,8080,8443,22,21,25,3306,5432 \
  -oA recon/active/nmap_web

# 5. Directory brute-force trên targets quan trọng
while read url; do
  domain=$(echo $url | cut -d: -f2 | tr -d '/')
  gobuster dir -u "$url" \
    -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
    -x php,html,txt,bak -t 40 -q \
    -o recon/active/gobuster_$domain.txt 2>/dev/null
done < <(grep " 200 " recon/active/httpx.txt | awk '{print $1}' | head -20)

# 6. Subdomain permutation
cat recon/active/live_subs.txt | dnsgen - | dnsx -silent >> recon/active/live_subs.txt
sort -u -o recon/active/live_subs.txt recon/active/live_subs.txt
```

### Phase 3 — Intelligence Analysis (Ngày 3)

```bash
# 1. Phân tích wayback URLs
cat recon/passive/wayback.txt | grep "?.*=" | sort -u > recon/active/params.txt
cat recon/passive/wayback.txt | grep "\.js$" | sort -u > recon/active/js_files.txt
cat recon/passive/wayback.txt | grep -i "api\|v1\|v2\|graphql" | sort -u > recon/active/api_endpoints.txt

# 2. JS endpoint extraction
while read url; do
  python3 linkfinder.py -i "$url" -o cli 2>/dev/null
done < recon/active/js_files.txt | sort -u > recon/active/js_endpoints.txt

# 3. Technology mapping từ httpx output
grep "WordPress\|Drupal\|Joomla\|Laravel" recon/active/httpx.txt > recon/active/cms_targets.txt
grep "Apache\|Nginx\|IIS" recon/active/httpx.txt > recon/active/servers.txt

# 4. Interesting targets
grep " 200 " recon/active/httpx.txt | grep -i "admin\|login\|dashboard\|portal" > recon/active/interesting.txt

# 5. Tổng hợp vào notes
echo "=== OSINT Summary ===" > notes/summary.md
echo "Total subdomains: $(wc -l < recon/passive/all_subs.txt)" >> notes/summary.md
echo "Live hosts: $(wc -l < recon/active/live_subs.txt)" >> notes/summary.md
echo "Parameters found: $(wc -l < recon/active/params.txt)" >> notes/summary.md
echo "API endpoints: $(wc -l < recon/active/api_endpoints.txt)" >> notes/summary.md
```

---

## 20. Bug Bounty OSINT Workflow

### NahamSec / Jhaddix Methodology

```bash
# "The Bug Hunter's Methodology" — Jason Haddix

# Bước 1: Subdomain → assetfinder + amass + subfinder + crt.sh
# Bước 2: HTTP probe → httpx
# Bước 3: Screenshots → gowitness / EyeWitness
# Bước 4: Content discovery → feroxbuster (recursive) hoặc gobuster
# Bước 5: JS analysis → LinkFinder + SecretFinder
# Bước 6: Parameter discovery → Arjun
# Bước 7: Vulnerability scan → nuclei

# Nuclei — template-based scanner
nuclei -l recon/active/live_subs.txt -t ~/nuclei-templates/cves/ -o nuclei_cves.txt
nuclei -l recon/active/live_subs.txt -t ~/nuclei-templates/exposures/ -o nuclei_exposed.txt
nuclei -l recon/active/live_subs.txt -t ~/nuclei-templates/misconfigurations/ -o nuclei_misconfig.txt
nuclei -l recon/active/live_subs.txt -t ~/nuclei-templates/technologies/ -o nuclei_tech.txt

# Arjun — parameter discovery
arjun -u https://example.com/api/endpoint -m GET
arjun -u https://example.com/api/endpoint -m POST

# FFUF — parameter fuzzing
ffuf -u "https://example.com/api?FUZZ=test" \
  -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
  -mc 200,301,302 -c
```

---

## 21. Advanced Tips & Tricks

### IppSec OSINT Habits (từ HTB walkthroughs)

```bash
# 1. Luôn check /robots.txt, /sitemap.xml
curl https://example.com/robots.txt
curl https://example.com/sitemap.xml

# 2. Check email header khi có sender email
# Received: from [IP] → internal IP, real mail server

# 3. Source code viewer → tìm comment HTML
curl -s https://example.com | grep -i "<!--\|//\s" | head -30

# 4. Version disclosure → search exploit
# "Apache/2.4.49" → CVE-2021-41773 (Path Traversal)
# Nikto luôn chạy trước khi gobuster

# 5. VHost enum — không bao giờ bỏ qua
gobuster vhost -u http://10.10.10.10 \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  --append-domain -t 50 --exclude-length 302
```

### Pivot Chain Examples

```bash
# Email → Identity chain
email@company.com
  → Hunter.io → email format ({first}.{last}@company.com)
  → LinkedIn → list of employees
  → Generate email list
  → Holehe → check registered services
  → GitHub search → find personal repos → secrets
  → Breach database → old passwords → reuse check

# Domain → Infrastructure chain
example.com
  → crt.sh → subdomains → internal.example.com
  → Shodan → IP 10.10.10.10 → open port 8080
  → Wayback → old version → exposed /admin path
  → S3 bucket: company-backup.s3.amazonaws.com → open listing

# IP → Network chain  
10.10.10.10
  → WHOIS → ASN 12345 → Company Name
  → BGP prefixes → 10.10.10.0/24, 10.20.0.0/16
  → Shodan scan → 150 hosts → GitLab on 10.10.10.50
  → GitLab public repos → internal credentials
```

### OPSEC khi OSINT

```bash
# Dùng VPN + Tor nếu cần ẩn danh hoàn toàn
# Kali VM riêng biệt, snapshot trước mỗi op
# Không dùng tài khoản cá nhân cho sock puppet research
# Xóa cookies / cache sau mỗi session
# Dùng Firefox Multi-Account Containers
# Kiểm tra IP leak: https://ipleak.net

# Một số site log IP visitor:
# - LinkedIn → log views
# - Hunter.io → log searches
# → Dùng VPN trước khi search
```

### Automation Script Cơ bản

```bash
#!/bin/bash
# quickrecon.sh — Quick recon script
TARGET=$1
OUTPUT_DIR="recon_$TARGET_$(date +%Y%m%d)"
mkdir -p $OUTPUT_DIR

echo "[*] Starting passive recon for $TARGET"

# Subdomain
subfinder -d $TARGET -silent -o $OUTPUT_DIR/subs.txt &
assetfinder --subs-only $TARGET >> $OUTPUT_DIR/subs.txt &
curl -s "https://crt.sh/?q=%25.$TARGET&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' >> $OUTPUT_DIR/subs.txt &
wait

sort -u -o $OUTPUT_DIR/subs.txt $OUTPUT_DIR/subs.txt
echo "[*] Found $(wc -l < $OUTPUT_DIR/subs.txt) unique subdomains"

# Resolve
cat $OUTPUT_DIR/subs.txt | dnsx -silent -o $OUTPUT_DIR/live.txt
echo "[*] $(wc -l < $OUTPUT_DIR/live.txt) live hosts"

# HTTP probe
cat $OUTPUT_DIR/live.txt | httpx -silent -status-code -title -o $OUTPUT_DIR/httpx.txt
echo "[*] HTTP probe done"

# Wayback
waybackurls $TARGET > $OUTPUT_DIR/wayback.txt &
gau $TARGET >> $OUTPUT_DIR/wayback.txt &
wait
sort -u -o $OUTPUT_DIR/wayback.txt $OUTPUT_DIR/wayback.txt
echo "[*] $(wc -l < $OUTPUT_DIR/wayback.txt) URLs from archives"

echo "[+] Done! Results in $OUTPUT_DIR/"
```

---

## 22. Tài nguyên & Cộng đồng học OSINT

### Khóa học & Sách

```
📚 Sách:
├── "Open Source Intelligence Techniques" — Michael Bazzell (cập nhật hàng năm, PHẢI đọc)
├── "The Art of Intrusion" — Kevin Mitnick
└── "Operator Handbook" — Joshua Picolet

🎓 Khóa học:
├── TCM Security "Practical Ethical Hacking" — tcm-sec.com
├── TCM Security "OSINT Fundamentals" — tcm-sec.com
├── SANS FOR578: Cyber Threat Intelligence
├── Bellingcat Online Investigation Workshops — bellingcat.com
└── TryHackMe "OSINT" learning path — tryhackme.com
```

### YouTube Channels

```
🎥 Must watch:
├── IppSec — https://youtube.com/@ippsec (HTB walkthroughs, recon section)
├── NahamSec — https://youtube.com/@NahamSec (Bug bounty recon)
├── TCM Security — https://youtube.com/@TCMSecurityAcademy
├── John Hammond — https://youtube.com/@_JohnHammond (CTF + OSINT)
└── David Bombal — https://youtube.com/@davidbombal
```

### CTF & Practice

```
🏋️ Thực hành:
├── https://app.hackthebox.com — OSINT challenges + machines có recon phase
├── https://tryhackme.com — OSINT rooms
├── https://ctftime.org — filter OSINT category
├── https://tracelabs.org — Missing persons OSINT CTF (thực tế nhất)
├── https://gralhix.com/list-of-osint-exercises/ — OSINT exercises
└── https://www.ozintexercises.com — Photo OSINT exercises
```

### Tools Repository

```
🔧 Tổng hợp tool:
├── https://osintframework.com — Mind map tất cả tool
├── https://github.com/jivoi/awesome-osint — Awesome OSINT list
├── https://github.com/ProjectDiscovery — httpx, nuclei, subfinder, dnsx...
├── https://github.com/tomnomnom — waybackurls, gf, httprobe, assetfinder
└── https://github.com/danielmiessler/SecLists — wordlists
```

### Communities

```
💬 Cộng đồng:
├── Discord: TCM Security, HackTheBox, NahamSec
├── Reddit: r/OSINT, r/netsec, r/bugbounty
├── Twitter/X: @NahamSec, @ippsec, @jhaddix, @TCMSecurity, @micahflee
└── Telegram: OSINT Vietnam, HackerViet
```

### Online OSINT Tools Bookmarks

```
🔖 Bookmark folder:
├── https://osintframework.com
├── https://crt.sh
├── https://shodan.io
├── https://censys.io
├── https://fofa.info
├── https://dnsdumpster.com
├── https://viewdns.info
├── https://securitytrails.com
├── https://www.virustotal.com
├── https://web.archive.org
├── https://hunter.io
├── https://haveibeenpwned.com
├── https://dehashed.com
├── https://epieos.com
├── https://www.exploit-db.com/google-hacking-database
└── https://builtwith.com
```

---

> **Lưu ý đạo đức & pháp lý:** Passive OSINT (Shodan, Google, crt.sh, Wayback) thường hợp pháp cho research. Active recon **bắt buộc phải có phép** — chỉ dùng trên target được ủy quyền. Không dùng breach data, credential leak cho mục đích bất hợp pháp. OSINT trên cá nhân: tôn trọng quyền riêng tư, không doxxing, không harassment. Các nền tảng HTB, THM, PentesterLab = môi trường an toàn để thực hành.