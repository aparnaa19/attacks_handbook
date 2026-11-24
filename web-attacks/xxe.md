<sub>[Home](../README.md) › [Web attacks](README.md)</sub>

# XML external entity (XXE)

> **In one line:** The attacker sends XML that tells the server's XML parser to **load an "external entity"** (a local file or a URL), so the server reads its own files or makes requests for the attacker.

| **OWASP** | A02:2025 Security Misconfiguration · CWE-611 |
|---|---|
| **MITRE ATT&CK** | [T1190 Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/) |
| **Where XML hides** | SOAP APIs, SAML, file uploads (`.docx`, `.xlsx`, `.svg` are XML inside), RSS, old REST APIs accepting `application/xml` |

## How it works

XML lets a document define **entities** (shortcuts). An entity can point to a file or URL. If the parser processes them, the attacker sends:

```xml
<?xml version="1.0"?>
<!DOCTYPE order [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<order><item>&xxe;</item></order>
```

The parser replaces `&xxe;` with the **contents of `/etc/passwd`**, and the app returns it in the response.

| Variant | What it does |
|---|---|
| **File read** | `SYSTEM "file:///etc/passwd"`, `file:///c:/windows/win.ini` |
| **SSRF** | `SYSTEM "http://169.254.169.254/latest/meta-data/"` → see [SSRF](ssrf.md) |
| **Blind / out-of-band** | Data sent to the attacker's server via a nested entity, when the response doesn't show it |
| **Billion laughs (DoS)** | Entities that expand into each other exponentially and exhaust memory |

## What you see in logs

| Log source | What to look for |
|---|---|
| WAF / app logs (request bodies) | `<!DOCTYPE`, `<!ENTITY`, `SYSTEM "`, `file://`, `php://`, `expect://` in XML bodies |
| Web access log | POSTs with `Content-Type: application/xml` / `text/xml` to endpoints that normally take JSON |
| Network / proxy | The **server** making outbound HTTP or DNS requests to unknown domains (out-of-band XXE) |
| Web access log | Uploads of `.svg`, `.docx`, `.xlsx` followed by errors or large responses |

Access logs don't contain POST bodies, so XXE is usually only visible in **WAF / application logs**.

## How to respond

1. Check responses (if logged) for file contents or metadata output.
2. If files with secrets were exposed (`.env`, config, SSH keys): **rotate** them.
3. If cloud metadata was reached: follow the [SSRF](ssrf.md) response.
4. **Fix:** **disable DTDs and external entities** in the XML parser (the real fix); prefer JSON; keep XML libraries updated.

## References

- [OWASP: XML External Entity Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)

---

| [← Server-side request forgery (SSRF)](ssrf.md) | [Home](../README.md) | [Insecure deserialization →](insecure-deserialization.md) |
|:---|:---:|---:|
