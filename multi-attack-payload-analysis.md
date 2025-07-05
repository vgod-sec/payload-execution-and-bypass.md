
title: "Payload Execution Analysis for Multiple Attacks"
description: "Guide to analyzing how payloads execute for XSS, SSRF, SQLi, IDOR, LFI, RCE, and more through source inspection and behavior analysis."
extended_description: "A practical walkthrough for bug bounty hunters and penetration testers to analyze how payloads execute across different attack types including XSS, SSRF, Path Traversal, SQL Injection, IDOR, LFI/RFI, RCE, Open Redirect, and Host Header Injection—by inspecting response behavior, source code reflection, timing differences, and out-of-band signals using tools like Burp Collaborator and browser DevTools."
---

# 🧠 How to Analyze Payload Execution for Different Attack Types (Bug Bounty/Recon)

## ✅ 1. XSS (Cross-Site Scripting)
### 🔍 Where to Look:
- Right click → View Page Source
- Chrome DevTools → Elements tab
- Reflected input in HTML, JS, attributes

### 🔍 What to Check:
- Is payload reflected? Escaped? Encoded?
- Is it inside a tag, script, attribute?
- Is it inside JS string or HTML comment?

### ✅ Signs of Execution:
- `<script>alert(1)</script>` runs
- DOM behavior changes
- Browser alert, Collaborator callback

---

## ✅ 2. SSRF (Server-Side Request Forgery)
### 🔍 Where to Look:
- Use **Burp Collaborator**, **interact.sh**
- Analyze server-side request logs (if blind)
- Look for image fetchers, PDF generators, webhooks

### 🔍 What to Check:
- Does the server make outbound HTTP requests?
- DNS or HTTP ping to your server?
- Response delay (e.g. from `http://localhost`)

### ✅ Signs of Execution:
- You receive DNS/HTTP hit on your listener
- You get internal IP disclosure (`169.254.x.x`)
- Time delay in response for `http://127.0.0.1:80`

---

## ✅ 3. Path Traversal
### 🔍 Where to Look:
- Response body shows file contents
- Source reveals path like `/etc/passwd`, `.bash_history`
- Parameter like `?file=about.txt` can be fuzzed

### 🔍 What to Check:
- Can you use `../` or `%2e%2e/`?
- Are you getting readable files?
- Is there a filter or extension like `.php`?

### ✅ Signs of Execution:
- File content returned (passwd, logs, config)
- 200 OK with sensitive content
- Source shows partial file or broken UI

---

## ✅ 4. SQL Injection
### 🔍 Where to Look:
- Look for reflected error messages in response
- Page breaking or returning SQL syntax errors
- Check responses to `'`, `1 OR 1=1`, etc.

### 🔍 What to Check:
- Is payload modifying SQL logic?
- Are you getting database errors?
- Can you use time delay or boolean logic?

### ✅ Signs of Execution:
- SQL error: `You have an error in your SQL syntax`
- Page shows unusual results (dumped tables, rows)
- Response delay when using `SLEEP(5)`

---

## ✅ 5. IDOR (Insecure Direct Object Reference)
### 🔍 Where to Look:
- Authenticated user ID or resource ID in URLs/requests
- Look at API responses and status codes
- Switch your ID to another (`user_id=2`)

### 🔍 What to Check:
- Are you getting other users' data?
- Can you modify IDs in requests or JSON?
- Is there any broken access control?

### ✅ Signs of Execution:
- You access other users' data
- Response includes another user’s email/username
- No 403/401 even after modifying IDs

---

## ✅ 6. LFI/RFI (Local/Remote File Inclusion)
### 🔍 Where to Look:
- Parameters like `?page=home`, `?file=report`
- Response shows source code or errors
- Use payloads like:
  - `../../etc/passwd`
  - `php://filter/convert.base64-encode/resource=index.php`

### 🔍 What to Check:
- Is the file being included and rendered?
- Are there PHP errors or source code leaks?
- Is your input used in `include()`, `require()`?

### ✅ Signs of Execution:
- Source code or sensitive file is displayed
- Base64 dump of PHP file is visible
- 500 errors on invalid file name (debug trace)

---

## ✅ 7. RCE (Remote Code Execution)
### 🔍 Where to Look:
- Input fields passed to system commands (e.g., ping, curl)
- File upload fields with interpretable content
- Use Burp Collaborator to confirm blind RCE

### 🔍 What to Check:
- Can you inject shell commands? (`;id`, `|whoami`)
- Is output reflected back? or delays?
- Does uploading a script get executed?

### ✅ Signs of Execution:
- Shell output (e.g., `uid=0(root)`)
- Collaborator hits from injected command
- Uploaded file runs on access (`.php`, `.jsp`)

---

## ✅ 8. Open Redirect
### 🔍 Where to Look:
- Parameters like `?redirect=https://target.com`
- After login or logout redirects

### 🔍 What to Check:
- Can you change the URL?
- Does it redirect to attacker domain?

### ✅ Signs of Execution:
- Browser redirects to your payload
- HTTP 302 response with `Location: attacker.com`

---

## ✅ 9. Host Header Injection
### 🔍 Where to Look:
- Analyze requests with `Host` header
- Test password reset, download links, or absolute URLs

### 🔍 What to Check:
- Change Host header to `evil.com`
- Does email or link use your Host?
- Do redirects point to your domain?

### ✅ Signs of Execution:
- Email sent with attacker-controlled domain
- Server uses injected Host in response

---

## 🧪 Tools to Observe Execution:

| Tool               | Purpose                                   |
|--------------------|-------------------------------------------|
| **Burp Collaborator** | SSRF, RCE, Blind XSS, etc.               |
| **interact.sh**       | Modern OAST (free) for testing callbacks |
| **DevTools (F12)**    | XSS, DOM injection, JS debugging         |
| **View Page Source**  | Reflected HTML/JS analysis               |
| **Burp Suite Logger** | Analyze request/response flows           |

---

## 👑 Summary Table

| Attack Type | What to Observe                    | How to Detect Execution                      |
|-------------|-------------------------------------|----------------------------------------------|
| XSS         | Page source / DOM                  | Alert, JS exec, input reflected in script    |
| SSRF        | Collaborator / timing              | DNS/HTTP hit, internal IP, delay             |
| Path Trav   | Response body                      | File contents, 200 OK with `/etc/passwd`     |
| SQLi        | Response / error / delay           | SQL error, dump, SLEEP-based delay           |
| IDOR        | API responses                      | Data from other users, 200 instead of 403    |
| LFI/RFI     | Source code, logs, file fetch      | File dump, base64 output, include errors     |
| RCE         | Response, collaborator, output     | Command output, server hit, uploaded file    |
| Redirect    | Response headers                   | Redirects to attacker                        |
| Host Inject | Email/links/headers                | Host replaced in output                      |

---

> 💡 **Always correlate the behavior of your payload with how the application reacts in HTML, headers, responses, time delays, or out-of-band (OAST) interactions.** This is how you confirm execution and impact.
