# WAF-Bypass-Techniques-Null-Byte-Encoding-Manipulation
A deep dive into advanced WAF bypass techniques using Null Byte injection, encoding mismatches, and parser inconsistencies — based on real-world payloads.

Why This Repository?
Web Application Firewalls (WAFs) are designed to block malicious requests. But sometimes, tiny inconsistencies between how a WAF parses data and how a browser interprets it can lead to complete bypasses.

This repository analyzes a real-world payload that earned €1000 in a bug bounty program — and explains exactly why it worked.

🎯 What You'll Learn
How Null Byte (%00) manipulation fools WAF parsers

Why WAFs and browsers handle encoding differently

How parser inconsistencies can hide malicious HTML/JavaScript

Why manual testing beats automation in finding logic flaws

How to build your own WAF bypass payloads

🔍 The Payload
text
1%3CWOQGK3%3ETCXTG[!%2B!]%27%22%3C%00!--%00%3E%3C%001 mg/Src/On%00Error=(conf%00irm) (1)%3E%3C/WOQGK3%3E
Decoded version:

text
1<WOQGK3>TCXTG[!+!]'"<�!--�><�1 mg/Src/On�Error=(conf�irm) (1)></WOQGK3>
This payload bypassed a major WAF and triggered an XSS vulnerability. Here's why.

1️⃣ Null Byte (%00) Manipulation
🛡️ What the WAF Sees
Many WAFs use C/C++ libraries or legacy regex engines where Null Byte (\0 or %00) is treated as a string terminator. When the WAF encounters %00, it thinks:

"End of string. Stop processing."

🌐 What the Browser Sees
Modern browsers (Chrome, Firefox, etc.) ignore Null Bytes or treat them as whitespace. They continue parsing.

Example from the payload:
html
On%00Error
WAF sees: On + [END] → No match for onerror

Browser sees: OnError (after stripping %00) → JavaScript triggers

Another example:
html
conf%00irm
WAF sees: conf + [END] → No match for confirm

Browser sees: confirm → Function executes

2️⃣ Encoding & Normalization Inconsistencies
WAFs typically decode requests before applying rules. But how they decode matters.

The payload contains:
text
%27%22%3C%00!--%00%3E
Decoded: '"<�!--�>

The problem:
If the WAF doesn't sanitize Null Bytes after decoding, the malicious signature is broken:

Expected signature	What WAF sees
<script>	<�script�> → No match
onerror	on�error → No match
confirm	conf�irm → No match
But the browser strips the Null Bytes and sees the real attack.

3️⃣ HTML Parser Inconsistencies
The payload contains:
html
%3C%001 mg/Src/On%00Error
Decoded: <�1 mg/Src/On�Error

What the WAF expects:
After <, a valid tag name like img, script, svg, etc.

What the WAF sees:
<�1 mg → Invalid tag name → "Not HTML, ignore"

What the browser does:
Browsers have high fault tolerance. They try to "fix" broken HTML:

Strip Null Bytes

Remove invalid characters

Parse remaining structure

Result: The browser sees:

html
<img src onerror=...
XSS triggered.

4️⃣ Why This Payload Earned €1000
Reason	Explanation
Logic flaw, not signature bypass	The WAF had a processing logic error, not just a missing rule
Null byte mishandling	WAF failed to sanitize %00 before analysis
Parser mismatch	WAF and browser parsed the same input differently
Manual discovery	Automation tools couldn't find this; required human creativity
"This proves the target skipped or mishandled Null Byte sanitization during preprocessing."

5️⃣ How WAFs Should Handle This (The Right Way)
A secure WAF preprocessing pipeline should:

Step	Action
1	Decode all encodings (URL, HTML, Unicode, etc.)
2	Strip Null Bytes (%00, \0)
3	Normalize input (remove junk, normalize whitespace)
4	Apply rules to cleaned input
If any step is missing or flawed, bypasses happen.

6️⃣ Building Your Own WAF Bypass Payloads
Techniques to experiment with:
Technique	Example
Null Byte injection	on%00error
Mixed encoding	&#x6f;&#x6e;&#x65;&#x72;&#x72;&#x6f;&#x72; (HTML entities)
Unexpected tags	<�1mg src>
Broken signatures	conf%00irm, aler%00t
Comment injection	<!–-><script>–->
Methodology:
Identify what the WAF blocks (e.g., onerror, alert, <script>)

Try to break the signature without breaking browser execution

Test encoding variations (URL, HTML, Unicode, UTF-7, etc.)

Study parser differences (WAF vs. browser)

📂 Repository Structure
File/Folder	Description
README.md	This guide
payloads/	Collection of WAF bypass payloads
techniques/	Detailed breakdown of each bypass method
browser-tests/	How different browsers parse malformed input
waf-behavior/	Notes on popular WAFs (Cloudflare, ModSecurity, AWS WAF, etc.)
cheatsheet.md	Quick reference for bypass techniques
📚 Resources
OWASP: WAF Evasion Techniques

PortSwigger: Bypassing WAFs

PayloadsAllTheThings: WAF Bypass

Null Byte Injection Explained

👨‍💻 Contributing
Found a new bypass technique? Want to add payloads or test results? PRs welcome!
