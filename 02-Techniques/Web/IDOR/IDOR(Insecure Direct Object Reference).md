

# I/What is an IDOR?

**-Definition of IDOR**

- **IDOR** stands for **Insecure Direct Object Reference**.
- It is categorized as an **access control vulnerability**.

**-How the Vulnerability Occurs**

+IDOR can occur when:

- A web server accepts **user-supplied input**.
- That input is used to directly retrieve or reference objects.

+Examples of such objects include:

- Files
- Data
- Documents

**-Root Cause**

+The vulnerability is caused by:

- Placing **excessive trust** in input provided by users.
- Failing to perform proper **server-side validation**.

**-Missing Security Check**

+The server does not verify whether:

- The requested object actually belongs to the user making the request.
- The user has permission to access that object.

**-Security Impact**

+As a result:

- Users may be able to access resources they are not authorized to view or retrieve.
- The application’s access control mechanism is effectively bypassed.

# II/An IDOR Example

**-Initial Scenario**

- A user signs up for an online service.
- The user wants to update or view their profile information.

**-Profile Access URL Structure**

+The application provides access through a URL such as:

http://online-service.thm/profile?user_id=1305

- The user_id parameter identifies which user's profile data is requested.
- In this case, the logged-in user can view their own profile information.

**-Parameter Manipulation**

+The user modifies the URL by changing:

user_id=1305

to:

user_id=1000

Resulting URL:

http://online-service.thm/profile?user_id=1000

**-Vulnerability Discovery**

+After changing the parameter:

- The application displays another user's profile information.
- This confirms unauthorized access is possible.

This behavior reveals an **IDOR vulnerability**.

**-Why This Happens**

+The vulnerability exists because:

- The server trusts the user_id value supplied in the request.
- There is no validation to ensure the requested profile belongs to the authenticated user.
- Access control checks are missing or improperly implemented.

**-Expected Secure Behavior**

+A properly secured application should:

- Verify that the logged-in user is authorized to access the requested profile.
- Reject unauthorized requests.
- Prevent direct access to resources owned by other users.

**-Security Implications**

+This flaw can allow attackers to:

- Access private user information
- Enumerate user accounts by modifying IDs
- Potentially gather sensitive or confidential data

# III/Finding IDORs in Encoded IDs

![Lesson 5 IDOR(Insecure Direct Object Reference)](<Module%206%20Web%20Application%20Vulnerabilities%20I/Attachments/Lesson%205%20IDOR\(Insecure%20Direct%20Object%20Reference\).png>)

**-Purpose of Encoding**

+Web developers often encode data before passing it between pages through:

- **POST data**
- **Query strings**
- **Cookies**

+The main goal is to ensure:

- Data is transmitted in a format the receiving web server can correctly interpret.
- Binary or structured data is safely converted into text-based form for web communication.

**-What Encoding Does**

+Encoding transforms:

- **Binary data**

into:

- **ASCII string representation**

+Commonly used characters include:

- Lowercase letters: **a-z**
- Uppercase letters: **A-Z**
- Numbers: **0-9**
- Padding character: **=**

This conversion makes the data suitable for transmission through HTTP requests.

**-Most Common Web Encoding Technique**

The content identifies **Base64 encoding** as:

- The most commonly used encoding technique on the web
- Generally easy to recognize due to its distinctive character patterns and use of = padding

**-Decoding and Manipulating Encoded Data**

Encoded values can often be analyzed through the following process:

**Step 1: Decode the String**

Use a decoding tool such as:

[Base64 Decode](https://www.base64decode.org/?utm_source=chatgpt.com)

to convert the encoded string back into readable data.

**Step 2: Modify the Data**

After decoding:

- Inspect the contents
- Edit relevant values or identifiers

**Step 3: Re-encode the Modified Data**

Use an encoding tool such as:

[Base64 Encode](https://www.base64encode.org/?utm_source=chatgpt.com)

to convert the modified data back into Base64 format.

**Step 4: Resubmit the Request**

Send the altered encoded value back to the web application and observe whether:

- The server accepts it
- The response changes
- Unauthorized behavior occurs

**-Security Testing Relevance**

This technique is useful for identifying whether:

- The application improperly trusts encoded client-side input
- Validation mechanisms are weak or absent
- Hidden identifiers can be manipulated

Encoding does **not** provide security by itself; it only changes data representation.

# IV/Finding IDORs in Hashed IDs

**-What Are Hashed IDs**

Hashed IDs are identifiers transformed using a **hashing algorithm**.

Their purpose is to:

- Obscure the original value
- Make identifiers less directly readable than plain integers

Compared to encoded IDs:

- They are more difficult to manipulate
- They cannot simply be decoded back to their original value

**-Why They Are More Complicated**

Unlike encoding:

- Hashing is designed as a **one-way transformation**
- There is no direct reversal process

This makes hashed IDs harder to inspect or alter.

**-Predictable Hashing Patterns**

Hashed IDs may still be vulnerable if the application generates them using predictable input values.

A common example:

- The application hashes a simple integer ID directly

Example:

Original ID:

123

MD5 hash:

202cb962ac59075b964b07152d234b70

This means:

- If IDs are sequential
- And the hashing algorithm is known

Attackers may generate hashes for likely values and test them.

**-Example Hashing Algorithm**

The content references **MD5** as the example hashing algorithm.

In this case:

- Input: 123
- Output: 202cb962ac59075b964b07152d234b70

This demonstrates how a numeric identifier can be converted into a fixed-length hash string.

**-Using Hash Lookup Services**

Discovered hashes can be checked against online lookup databases.

One example is:

[CrackStation](https://crackstation.net/?utm_source=chatgpt.com)

This service:

- Maintains databases containing billions of known hash-to-value mappings
- Can identify original values if the hash is already stored

**-Purpose of Hash Lookup Testing**

Checking hashes can help determine whether:

- The hash corresponds to a simple predictable value
- The application uses weak or guessable identifier generation
- The obscured identifier can still be practically resolved

**-Security Implication**

Hashing alone does not guarantee security if:

- Input values are predictable
- Unsalted hashes are used
- Sequential identifiers are directly hashed

In such cases, hashed IDs may still be exploitable.

# V/Finding IDORs in Unpredictable IDs

**-The Challenge of Unpredictable IDs**

+Some applications use identifiers that:

- Do not follow sequential numbering
- Are not obviously encoded
- Cannot be reversed through decoding
- Do not reveal predictable hashing patterns

Because of this:

- Traditional analysis methods may fail to expose the identifier structure

**-Alternative Detection Method**

When identifier patterns cannot be determined, a practical approach is to:

**Create two separate user accounts**

This allows controlled testing between known resources.

**-Testing Procedure**

The process involves:

**Step 1: Create Two Accounts**

- Account A
- Account B

Each account will have:

- Its own resources
- Its own object identifiers

**Step 2: Access One Account Normally**

Log in to one account and observe the identifier used in requests.

**Step 3: Replace the Identifier**

Modify the request by swapping the current account’s ID with the ID belonging to the second account.

**Step 4: Observe the Response**

Check whether the application allows access to the second account’s content.

**-Indicators of a Valid IDOR**

An IDOR vulnerability exists if:

**Scenario A**

While logged into Account A:

- You can access Account B’s data by replacing the identifier

**Scenario B**

Without authentication:

- You can still access another user’s content

Both situations indicate missing access control enforcement.

**-Why This Works**

This method tests whether the application validates:

- User ownership of the requested resource
- Session-to-resource authorization mapping
- Access permissions before returning data

If these checks are absent, IDOR is present.

**- Security Implication**

A successful test demonstrates that:

- The server trusts the provided identifier without verification
- Authorization controls are missing or insufficient
- Users can access resources they do not own

This can expose:

- Private account data
- Sensitive user content
- Restricted resources

# VI/Where are IDORs located

**-Vulnerable Endpoints May Be Hidden**

+The content emphasizes that vulnerable endpoints are not always directly visible to users.

+They may exist in:

- Background browser requests
- Application scripts
- Hidden API calls
- Undocumented request parameters

+This means vulnerability discovery often requires deeper inspection beyond visible page navigation.

**-AJAX Requests as a Source**

+A vulnerable endpoint may be loaded through an **AJAX request**.

+This means:

- The browser fetches data asynchronously
- The request happens in the background
- The endpoint may not appear in the address bar

+Attackers can inspect these requests using browser developer tools to identify potentially vulnerable object references.

**-JavaScript File References**

+Endpoints may also be revealed inside **JavaScript files**.

+JavaScript can contain:

- API endpoint definitions
- Request parameters
- Internal route references
- Development-related functionality

+Reviewing these files may expose hidden access paths.

**-Unreferenced Parameters**

+Applications may contain parameters that are:

- No longer actively used
- Left over from development
- Accidentally deployed to production

+These undocumented parameters can still be processed by the server.

**-Parameter Mining**

+The content introduces **parameter mining** as an attack technique.

+Parameter mining involves:

- Discovering hidden request parameters
- Testing undocumented input fields
- Identifying server-supported parameters not exposed in the interface

+This can reveal unexpected functionality.

**-Example Scenario**

+A normal request might be:

/user/details

+This endpoint displays the authenticated user’s information using session-based authentication.

**-Discovery of Hidden Parameter**

+Through parameter mining, an attacker discovers:

user_id

+This transforms the endpoint into:

/user/details?user_id=123

**-Why This Creates Risk**

+If the server accepts this parameter without validating ownership:

- Users may request arbitrary user records
- Session-based protections may be bypassed
- Unauthorized data exposure becomes possible

+This creates an IDOR vulnerability.

**-Development-to-Production Risk**

+The example demonstrates how:

- Development functionality may remain enabled
- Legacy parameters may persist unnoticed
- Security validation may be incomplete

+These oversights can introduce exploitable vulnerabilities.