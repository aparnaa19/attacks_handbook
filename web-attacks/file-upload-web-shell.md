<sub>[Home](../README.md) › [Web attacks](README.md)</sub>

# File upload attacks and web shells

> **In one line:** The attacker **uploads a script** (`.php`, `.aspx`, `.jsp`) to the web server and then opens it in the browser, giving them a **web shell**: a web page that runs any command on the server.

| **MITRE ATT&CK** | [T1505.003 Server Software Component: Web Shell](https://attack.mitre.org/techniques/T1505/003/) (Persistence) |
|---|---|
| **CWE** | CWE-434 Unrestricted Upload of File with Dangerous Type |
| **Known web shells** | China Chopper (one line of code), ASPXSpy, WSO, Godzilla, Behinder (encrypted traffic) |

Web shells also arrive through other bugs: after [exploiting a public-facing app](../initial-access/exploit-public-facing-app.md), attackers almost always drop one for persistence.

## How it works

1. A site lets users upload a profile picture.
2. The attacker uploads `shell.php` containing:

    ```php
    <?php system($_GET['cmd']); ?>
    ```

3. The site saves it to `/uploads/shell.php` and doesn't stop `.php` files from running.
4. The attacker opens:

    ```text
    https://site.com/uploads/shell.php?cmd=whoami
    ```

5. The server runs `whoami` and shows the output. The attacker now has a remote shell over HTTP.

## Upload filter bypasses

| Trick | Example |
|---|---|
| Double extension | `shell.php.jpg`, `shell.jpg.php` |
| Alternative extensions | `.phtml`, `.php5`, `.phar`, `.asp`, `.ashx`, `.cer` |
| Case change | `shell.PhP` |
| Fake Content-Type | Send `Content-Type: image/jpeg` for a PHP file |
| Magic bytes | Start the file with `GIF89a` so it "looks like" a GIF |
| Config upload | Upload `.htaccess` / `web.config` that makes `.jpg` files run as code |

## What you see in logs

| Log source | What to look for |
|---|---|
| Web access log | **POST to an upload endpoint**, followed by **GET/POST to a new script file** in an uploads or images folder |
| Web access log | Requests to a script that **only one or two IPs** ever access, often with `cmd=`, `c=`, `exec=` or encrypted POST bodies |
| **EDR / Sysmon 1** | `w3wp.exe` (IIS), `httpd`, `php-fpm`, `java`, `tomcat` spawning `cmd.exe`, `powershell.exe`, `sh`, `whoami`, `net`, `ipconfig` |
| **Sysmon 11** | The web server process **creating** `.aspx`, `.php`, `.jsp` files in web directories |
| File integrity monitoring | New or modified script files in the web root |

## How to detect it

- Web server process creating script files or spawning shells. **High-fidelity on Windows IIS** (`w3wp.exe` → `cmd.exe`).
- Rare URLs: script files with very few distinct visitors.

**Example Splunk search: IIS writing script files**

```spl
index=sysmon EventCode=11 Image="*\\w3wp.exe"
  (TargetFilename="*.aspx" OR TargetFilename="*.ashx" OR TargetFilename="*.asmx" OR TargetFilename="*.jsp" OR TargetFilename="*.php")
| table _time, host, TargetFilename
```

## How to respond

1. **Isolate** the server and **preserve** the web shell file and logs (evidence).
2. Find **every** web shell: hunt for script files created around the same time and anywhere in web roots.
3. Reconstruct what the attacker ran (process logs) and whether they moved deeper into the network.
4. Find the **entry point** (vulnerable upload form? exploited CVE?) and fix it, or they'll come back.
5. Rotate credentials stored on the server (DB passwords in config files).
6. **Fix:** allowlist file types by checking content, rename uploads to random names, store uploads **outside the web root** or on separate storage, remove execute permission from upload folders.

## References

- [MITRE ATT&CK T1505.003](https://attack.mitre.org/techniques/T1505/003/)
- [OWASP: File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)

---

| [← IDOR](idor.md) | [Home](../README.md) | [Network attacks →](../network-attacks/README.md) |
|:---|:---:|---:|
