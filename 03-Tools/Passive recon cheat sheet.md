# 🕵️ Passive Reconnaissance Cheat Sheet

> Tổng hợp công cụ & kỹ thuật được sử dụng bởi **IppSec**, **TJNull**, **0xdf** và cộng đồng hacker uy tín.

---

## 📌 Mục lục

1. [WHOIS](https://claude.ai/chat/15a1d2d3-a923-47f5-acf1-d7766136f7a4#1-whois)
2. [RDAP](https://claude.ai/chat/15a1d2d3-a923-47f5-acf1-d7766136f7a4#2-rdap)
3. [nslookup](https://claude.ai/chat/15a1d2d3-a923-47f5-acf1-d7766136f7a4#3-nslookup)
4. [dig](https://claude.ai/chat/15a1d2d3-a923-47f5-acf1-d7766136f7a4#4-dig)
5. [DNSDumpster](https://claude.ai/chat/15a1d2d3-a923-47f5-acf1-d7766136f7a4#5-dnsdumpster)
6. [Shodan.io](https://claude.ai/chat/15a1d2d3-a923-47f5-acf1-d7766136f7a4#6-shodanio)
7. [Công cụ bổ sung](https://claude.ai/chat/15a1d2d3-a923-47f5-acf1-d7766136f7a4#7-c%C3%B4ng-c%E1%BB%A5-b%E1%BB%95-sung)
8. [Workflow tổng quát](https://claude.ai/chat/15a1d2d3-a923-47f5-acf1-d7766136f7a4#8-workflow-t%E1%BB%95ng-qu%C3%A1t)

---

## 1. WHOIS

> Truy vấn thông tin đăng ký domain / IP (registrar, owner, dates, name servers).

### Cú pháp cơ bản

```bash
whois <domain>
whois <IP>
whois -h whois.arin.net <IP>        # Query ARIN (Mỹ)
whois -h whois.ripe.net <IP>        # Query RIPE (Châu Âu)
whois -h whois.apnic.net <IP>       # Query APNIC (Châu Á)
```

### Ví dụ thực tế

```bash
whois hackthebox.eu
whois 10.10.10.1
whois -h whois.verisign-grs.com google.com
```

### Thông tin cần chú ý

|Field|Ý nghĩa|
|---|---|
|`Registrar`|Nhà đăng ký domain|
|`Creation Date`|Ngày tạo — domain mới thường đáng nghi|
|`Name Server`|NS records → pivot sang infrastructure|
|`Admin/Tech Email`|Pivot sang OSINT (người thật, tổ chức)|
|`Registrant Org`|Tên công ty / tổ chức sở hữu|

### Tips từ IppSec / Community

- So sánh `Creation Date` vs `Expiry Date` → domain sắp hết hạn có thể bị hijack
- Email trong WHOIS → tra Google, LinkedIn, HaveIBeenPwned
- Dùng `jwhois` trên Linux để query tự động đúng server

---

## 2. RDAP

> Thế hệ kế tiếp của WHOIS — JSON format, chuẩn hóa, hỗ trợ tốt hơn.

### Web API (không cần cài đặt)

```
https://rdap.verisign.com/com/v1/domain/example.com
https://rdap.arin.net/registry/ip/8.8.8.8
https://rdap.db.ripe.net/ip/1.1.1.1
```

### CLI với `curl`

```bash
curl -s https://rdap.verisign.com/com/v1/domain/hackthebox.eu | python3 -m json.tool
curl -s https://rdap.arin.net/registry/ip/10.10.10.1 | jq .
```

### Tool `rdap` (Python)

```bash
pip install rdap
rdap hackthebox.eu
rdap --json 8.8.8.8
```

### Ưu điểm so với WHOIS

- Output JSON → dễ parse với `jq`
- Thông tin chuẩn hóa, ít bị redact hơn
- Hỗ trợ HTTPS, có authentication
- Lookup ASN, network blocks đầy đủ hơn

---

## 3. nslookup

> DNS lookup tương tác — có trên mọi OS (Windows, Linux, macOS).

### Cú pháp cơ bản

```bash
nslookup <domain>
nslookup <domain> <dns_server>      # Dùng DNS server cụ thể
nslookup -type=<record> <domain>    # Query record type cụ thể
```

### Query các record type

```bash
nslookup -type=A example.com        # IPv4 address
nslookup -type=AAAA example.com     # IPv6 address
nslookup -type=MX example.com       # Mail servers
nslookup -type=NS example.com       # Name servers
nslookup -type=TXT example.com      # TXT (SPF, DKIM, verify tokens)
nslookup -type=CNAME www.example.com
nslookup -type=SOA example.com      # Start of Authority
nslookup -type=ANY example.com      # Tất cả records
```

### Reverse lookup

```bash
nslookup 8.8.8.8                    # PTR record
nslookup -type=PTR 8.8.8.8
```

### Chế độ tương tác

```bash
nslookup
> server 8.8.8.8        # Đổi DNS server
> set type=MX
> example.com
> exit
```

### Tips

- Dùng `8.8.8.8` hoặc `1.1.1.1` khi muốn bypass DNS nội bộ
- TXT records thường chứa thông tin nhạy cảm (SPF leak internal hosts)
- MX records → xác định email provider → pivot sang phishing recon

---

## 4. dig

> Công cụ DNS mạnh nhất — được IppSec sử dụng rộng rãi trong các write-up.

### Cú pháp cơ bản

```bash
dig <domain>
dig <domain> <record_type>
dig @<dns_server> <domain> <record_type>
```

### Query phổ biến

```bash
dig example.com                     # A record (mặc định)
dig example.com A
dig example.com AAAA
dig example.com MX
dig example.com NS
dig example.com TXT
dig example.com SOA
dig example.com CNAME
dig example.com ANY                 # Tất cả (bị block nhiều nơi)
dig example.com CAA                 # Certificate Authority Authorization
```

### Dùng DNS server cụ thể

```bash
dig @8.8.8.8 example.com
dig @1.1.1.1 example.com MX
dig @ns1.example.com example.com    # Query trực tiếp authoritative NS
```

### Reverse lookup

```bash
dig -x 8.8.8.8
dig -x 1.1.1.1 @8.8.8.8
```

### Zone Transfer (AXFR) — 🔥 Critical

```bash
dig axfr <domain> @<nameserver>
dig axfr example.com @ns1.example.com
dig axfr @nsztm1.digi.ninja zonetransfer.me   # Lab test
```

### Output gọn

```bash
dig +short example.com              # Chỉ hiện IP
dig +short example.com MX
dig +noall +answer example.com      # Chỉ phần answer
dig +nocmd +noall +answer example.com TXT
```

### Subdomain brute / bulk

```bash
# Query nhiều domain cùng lúc
dig +short example.com google.com hackthebox.eu

# Từ file
for sub in $(cat subdomains.txt); do dig +short $sub.example.com; done
```

### Trace DNS resolution

```bash
dig +trace example.com             # Trace từ root đến authoritative
dig +trace +additional example.com
```

### Tips từ IppSec

- Luôn thử **AXFR** trên lab HTB/CTF — misconfigured DNS thường leak toàn bộ zone
- `dig +short` nhanh hơn khi scripting
- TXT records thường có **API keys**, **Google site verification**, **internal hostnames**
- Dùng `dig SOA` để biết primary NS trước khi AXFR

---

## 5. DNSDumpster

> Web-based DNS recon — không cần cài đặt, visualize DNS infrastructure.

### Web Interface

```
https://dnsdumpster.com
```

### API (unofficial)

```bash
curl -s "https://api.hackertarget.com/hostsearch/?q=example.com"
curl -s "https://api.hackertarget.com/dnslookup/?q=example.com"
curl -s "https://api.hackertarget.com/reversedns/?q=1.1.1.1"
curl -s "https://api.hackertarget.com/zonetransfer/?q=zonetransfer.me"
```

### Python wrapper

```bash
pip install dnsdumpster
python3 -c "from dnsdumpster.DNSDumpsterAPI import DNSDumpsterAPI; r = DNSDumpsterAPI().search('example.com'); print(r)"
```

### Thông tin DNSDumpster trả về

|Loại|Mô tả|
|---|---|
|**DNS Records**|A, MX, NS, TXT records|
|**Host Records**|Subdomain enumeration từ crawl + brute|
|**MX Records**|Mail servers + IP|
|**TXT Records**|SPF, DKIM, verification strings|
|**Map**|Visual network map có thể export PNG|

### Tips

- Export **network map** → trình bày trong report rất chuyên nghiệp
- Kết hợp với Shodan để enrich IP information
- So sánh kết quả với `dig ANY` → tìm records bị ẩn

---

## 6. Shodan.io

> Search engine cho IoT, servers, exposed services — "Google cho hackers".

### Web Interface

```
https://shodan.io
```

### CLI Setup

```bash
pip install shodan
shodan init <API_KEY>
shodan info                         # Kiểm tra credits
```

### Search queries cơ bản

```bash
shodan search "apache"
shodan search "nginx port:8080"
shodan search hostname:example.com
shodan search org:"Target Company"
shodan host <IP>                    # Chi tiết 1 IP
shodan count "port:22 country:VN"  # Đếm kết quả
```

### Shodan Dorks — 🔥 Quan trọng

```
# Tìm theo tổ chức
org:"Company Name"
org:"Company Name" port:22

# Tìm theo domain
hostname:example.com
hostname:.example.com               # Tất cả subdomains

# Theo ASN
asn:AS15169                         # Google ASN

# Theo city/country
city:"Ho Chi Minh City" port:80
country:VN port:22

# Tìm thiết bị cụ thể
product:"Apache httpd"
product:"nginx" version:"1.14"

# Tìm CVE / vuln
vuln:CVE-2021-44228                 # Log4Shell
vuln:CVE-2019-0708                  # BlueKeep

# Certificates (TLS recon)
ssl:"example.com"
ssl.cert.subject.cn:"*.example.com"
ssl.cert.issuer.cn:"Let's Encrypt"

# Tìm config files exposed
http.title:"phpMyAdmin"
http.title:"Kibana"
http.title:"Grafana"
http.title:"Jenkins"

# Webcams
has_screenshot:true port:554

# Default credentials panels
http.html:"default password"
http.title:"Router Login"
```

### Shodan CLI commands

```bash
shodan search --fields ip_str,port,org "apache" > output.txt
shodan host 1.1.1.1
shodan host 1.1.1.1 --history       # Lịch sử các lần scan
shodan download results "apache port:8080"
shodan parse --fields ip_str,port results.json.gz
shodan alert create --expires 30 "My Network" 192.168.1.0/24
shodan monitor list                 # Monitoring alerts
```

### Shodan API (Python)

```python
import shodan

api = shodan.Shodan('YOUR_API_KEY')

# Search
results = api.search('hostname:example.com')
for r in results['matches']:
    print(r['ip_str'], r.get('port'), r.get('org'))

# Host lookup
host = api.host('8.8.8.8')
print(host['org'], host['country_name'])
for item in host['data']:
    print(f"Port: {item['port']} | Banner: {item['data'][:50]}")
```

### Tips từ IppSec / Community

- `ssl:"example.com"` → tìm **tất cả IP** dùng cert của domain đó (bypass CDN)
- Kết hợp `org:` + `port:` → mapping attack surface nhanh
- `http.favicon.hash:<hash>` → tìm server dùng cùng favicon (fingerprint)
- Free account: 1 page kết quả. Dùng **Shodan CLI** với API key để lấy nhiều hơn

---

## 7. Công cụ bổ sung

### theHarvester — Email & subdomain harvesting

```bash
theHarvester -d example.com -b all
theHarvester -d example.com -b google,linkedin,shodan -l 500
theHarvester -d example.com -b dnsdumpster,crtsh
```

### crt.sh — Certificate Transparency Logs

```bash
# Web
https://crt.sh/?q=%.example.com

# API
curl -s "https://crt.sh/?q=%.example.com&output=json" | jq '.[].name_value' | sort -u
curl -s "https://crt.sh/?q=example.com&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u
```

### Amass — Subdomain enumeration

```bash
amass enum -passive -d example.com
amass enum -passive -d example.com -o output.txt
amass intel -org "Target Company"
amass intel -ip -src -brute -min-for-recursive 2 -d example.com
```

### Subfinder

```bash
subfinder -d example.com
subfinder -d example.com -o subdomains.txt
subfinder -d example.com -silent | httpx -title -status-code
```

### ASN Lookup

```bash
# Tìm ASN của IP
curl -s https://api.hackertarget.com/aslookup/?q=8.8.8.8

# Tìm IP ranges của org
curl -s https://api.hackertarget.com/aslookup/?q=AS15169

# whois ASN
whois -h whois.radb.net AS15169
```

### BGP / IP Intelligence

```
https://bgp.he.net/            # Hurricane Electric BGP Toolkit
https://ipinfo.io/8.8.8.8      # IP info + ASN
https://mxtoolbox.com          # MX, SPF, DKIM checker
https://dnschecker.org         # DNS propagation checker
https://viewdns.info           # Reverse IP, WHOIS, DNS
```

---

## 8. Workflow Tổng Quát

```
TARGET: example.com
│
├── 1. WHOIS / RDAP
│   ├── Registrar, dates, name servers
│   └── Admin email → OSINT pivot
│
├── 2. DNS Enumeration
│   ├── dig NS example.com        → Tìm authoritative NS
│   ├── dig AXFR @ns1.example.com → Zone transfer (nếu được phép)
│   ├── dig TXT example.com       → SPF, internal hosts
│   └── dig MX example.com        → Email infrastructure
│
├── 3. Subdomain Discovery
│   ├── crt.sh                    → Certificate transparency
│   ├── DNSDumpster               → Passive crawl + brute
│   ├── subfinder / amass         → Multi-source aggregation
│   └── theHarvester              → Google dorks + LinkedIn
│
├── 4. IP / Service Discovery
│   ├── Shodan: hostname:example.com
│   ├── Shodan: ssl:"example.com" → Bypass CDN
│   ├── Shodan: org:"Company"    → Toàn bộ attack surface
│   └── BGP / ipinfo.io          → ASN, IP ranges
│
└── 5. Correlation & Pivot
    ├── IP → Reverse DNS → Thêm hosts
    ├── Email → HaveIBeenPwned → Breach data
    ├── Cert SAN → Thêm subdomains
    └── TXT records → Internal hostnames, API keys
```

---

## 🔗 Tài nguyên tham khảo

|Resource|Link|
|---|---|
|IppSec YouTube|https://www.youtube.com/@ippsec|
|0xdf Writeups|https://0xdf.gitlab.io|
|TJNull HTB List|https://www.netsecfocus.com/oscp/2021/05/06/The_Journey_to_Try_Harder.html|
|HackerTarget API|https://hackertarget.com/ip-tools|
|Shodan Dorks|https://github.com/jakejarvis/awesome-shodan-queries|
|DNSDumpster|https://dnsdumpster.com|
|crt.sh|https://crt.sh|
|BGP.he.net|https://bgp.he.net|

---

> **⚠️ Disclaimer:** Chỉ sử dụng các kỹ thuật này trên hệ thống bạn được phép kiểm tra (CTF, Bug Bounty, Pentest có hợp đồng). Unauthorized recon có thể vi phạm pháp luật.