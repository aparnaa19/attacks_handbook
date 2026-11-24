<sub>[Home](../README.md)</sub>

# Web attacks

Attacks against **web applications and APIs**. The attacker sends specially crafted input in a URL, form, header or API body, and the application handles it unsafely.

| Attack | Root cause | OWASP Top 10:2025 | CWE |
|---|---|---|---|
| [SQL injection](sql-injection.md) | Input becomes part of a database query | A05 Injection | CWE-89 |
| [Cross-site scripting (XSS)](xss.md) | Input becomes part of a web page's HTML/JavaScript | A05 Injection | CWE-79 |
| [Command injection](command-injection.md) | Input becomes part of an OS command | A05 Injection | CWE-78 |
| [Path traversal and LFI](path-traversal.md) | Input becomes part of a file path | A01 Broken Access Control | CWE-22 |
| [SSRF](ssrf.md) | Server fetches a URL chosen by the attacker | A01 Broken Access Control | CWE-918 |
| [CSRF](csrf.md) | Server trusts any request that carries the user's cookie | | CWE-352 |
| [IDOR](idor.md) | Server doesn't check if the object belongs to the user | A01 Broken Access Control | CWE-639 |
| [File upload and web shells](file-upload-web-shell.md) | Server accepts and runs uploaded code | | CWE-434 |
| [XXE](xxe.md) | XML parser loads files or URLs named in the input | A02 Security Misconfiguration | CWE-611 |
| [Insecure deserialization](insecure-deserialization.md) | Server rebuilds attacker-supplied objects and runs code | A08 Software or Data Integrity Failures | CWE-502 |
| [Open redirect](open-redirect.md) | Trusted site redirects to any URL in a parameter | | CWE-601 |

## OWASP Top 10:2025

| # | Category |
|---|---|
| A01 | Broken Access Control (now includes SSRF) |
| A02 | Security Misconfiguration |
| A03 | Software Supply Chain Failures |
| A04 | Cryptographic Failures |
| A05 | Injection |
| A06 | Insecure Design |
| A07 | Authentication Failures |
| A08 | Software or Data Integrity Failures |
| A09 | Security Logging and Alerting Failures |
| A10 | Mishandling of Exceptional Conditions |

## HTTP basics you need

```http
POST /login.php?lang=en HTTP/1.1          ← method, path, query string
Host: shop.example.com
User-Agent: Mozilla/5.0 ...               ← client software (sqlmap, curl, python-requests = suspicious)
Cookie: session=abc123                    ← who the user is
Content-Type: application/x-www-form-urlencoded

username=bob&password=secret              ← body (NOT in normal access logs)
```

> [!WARNING]
> **Access logs don't show POST bodies**
>
> Apache, Nginx and IIS access logs record the **URL and query string**, not the request body. Attacks sent in POST bodies or JSON only show up in **WAF logs, application logs or full packet capture**.

## URL encoding cheat sheet

Attackers encode characters to slip past filters. Decode before judging a request.

| Encoded | Character | Encoded | Character |
|---|---|---|---|
| `%27` | `'` | `%3C` | `<` |
| `%22` | `"` | `%3E` | `>` |
| `%20` or `+` | space | `%2F` | `/` |
| `%2E` | `.` | `%5C` | `\` |
| `%3B` | `;` | `%7C` | `|` |
| `%26` | `&` | `%3D` | `=` |
| `%23` | `#` | `%25` | `%` (so `%252e` = double-encoded `.`) |
| `%0a` | newline | `%00` | null byte |

Tool: **CyberChef** → `URL Decode`.

## General triage for a web attack alert

1. **Decode** the request and identify the attack type.
2. **Did it work?** Look at the **status code**, **response size** (compare with normal) and **response time**. A WAF "blocked" = it didn't reach the app.
3. **Scanner or targeted?** Scanners (sqlmap, Nikto, Burp, Nuclei) send hundreds of payloads fast with tell-tale User-Agents. A few precise requests are more worrying.
4. **What happened next on the server?** New processes, new files, outbound connections.

---

| [← Process injection and DLL side-loading](../malware/process-injection.md) | [Home](../README.md) | [SQL injection (SQLi) →](sql-injection.md) |
|:---|:---:|---:|
