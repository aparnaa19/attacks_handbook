# I see X, what is it?

Find what you see in the alert or log, then open the page for the likely attack.

## Logons and accounts

| What you see | Likely attack |
|---|---|
| Many 4625 for **one** user from one IP | [Brute force](../credential-attacks/brute-force.md) |
| 4625 for **many** users, few attempts each, same IP | [Password spraying](../credential-attacks/password-spraying.md) |
| Many users, one attempt each, many IPs, some succeed | [Credential stuffing](../credential-attacks/credential-stuffing.md) |
| Many MFA push requests for one user, then an approval | [MFA fatigue](../credential-attacks/mfa-fatigue.md) |
| Sign-in from a new country right after a normal sign-in ("impossible travel") with MFA satisfied | [Session token theft](../cloud-identity/session-token-theft.md) |
| 4624 logon type 3 with NTLM from a workstation to many hosts | [Pass-the-hash](../credential-attacks/pass-the-hash.md) |
| Many 4769 with encryption type `0x17` from one user | [Kerberoasting](../credential-attacks/kerberoasting.md) |
| 4768 with pre-auth type 0 | [AS-REP roasting](../credential-attacks/as-rep-roasting.md) |
| 4662 with replication GUIDs from a non-DC account | [DCSync](../credential-attacks/dcsync.md) |
| 4769 with no matching 4768, odd ticket lifetime | [Golden ticket](../credential-attacks/golden-silver-ticket.md) |
| New user created (4720) or added to admins (4728/4732) unexpectedly | [Persistence](../post-exploitation/persistence.md) / [Privilege escalation](../post-exploitation/privilege-escalation.md) |
| Inbox rule forwarding mail to an external address | [Business email compromise](../initial-access/business-email-compromise.md) |
| New OAuth app with mail / file permissions consented by a user | [OAuth consent phishing](../cloud-identity/oauth-consent-phishing.md) |

## Processes

| What you see | Likely attack |
|---|---|
| `winword.exe` / `excel.exe` / `outlook.exe` spawning `powershell`, `cmd`, `mshta`, `wscript` | [Phishing](../initial-access/phishing.md) with malicious attachment |
| `powershell -enc`, `-nop -w hidden`, `IEX (New-Object Net.WebClient)` | [PowerShell abuse](../malware/powershell-abuse.md) |
| `certutil -urlcache`, `mshta http://`, `regsvr32 /i:http`, `rundll32` with odd DLL | [LOLBins](../malware/lolbins.md) |
| Process accessing `lsass.exe` (Sysmon 10), `procdump lsass`, `comsvcs.dll MiniDump` | [Credential dumping](../credential-attacks/credential-dumping.md) |
| `vssadmin delete shadows`, `wbadmin delete catalog`, mass file renames | [Ransomware](../malware/ransomware.md) |
| `w3wp.exe` / `httpd` / `java` spawning `cmd`, `whoami`, `sh` | [Web shell](../web-attacks/file-upload-web-shell.md) / [Command injection](../web-attacks/command-injection.md) / [Exploited app](../initial-access/exploit-public-facing-app.md) |
| `whoami`, `net user /domain`, `net group "domain admins"`, `nltest /dclist` in a burst | Discovery after compromise. See [Attack lifecycle](../basics/attack-lifecycle.md) |
| `wevtutil cl`, event 1102, Defender 5001 | [Defense evasion](../post-exploitation/defense-evasion.md) |
| `rclone`, `MEGAsync`, `7z a -p` on a server | [Data exfiltration](../post-exploitation/data-exfiltration.md) |
| Event 7045 service with random name or `\\ADMIN$` path | [Lateral movement](../post-exploitation/lateral-movement.md) (PsExec-style) |

## Network

| What you see | Likely attack |
|---|---|
| One IP connecting to many ports or many hosts | [Reconnaissance and scanning](../network-attacks/reconnaissance-scanning.md) |
| Connections to the same external IP at regular intervals | [C2 beaconing](../network-attacks/c2-beaconing.md) |
| Very long, random-looking subdomains; many TXT queries to one domain | [DNS tunneling](../network-attacks/dns-tunneling.md) |
| Huge traffic spike, service down | [DoS / DDoS](../network-attacks/dos-ddos.md) |
| One MAC address claiming the gateway's IP | [ARP spoofing](../network-attacks/mitm-arp-spoofing.md) |
| LLMNR / NBT-NS responses from a workstation | [LLMNR poisoning](../network-attacks/llmnr-nbtns-poisoning.md) |
| Large outbound upload to cloud storage at night | [Data exfiltration](../post-exploitation/data-exfiltration.md) |
| One host scanning port 445 across the subnet | [Worm](../malware/worms.md) |

## Web server logs

| What you see in the URL / body | Likely attack |
|---|---|
| `' OR 1=1--`, `UNION SELECT`, `SLEEP(5)`, `sqlmap` User-Agent | [SQL injection](../web-attacks/sql-injection.md) |
| `<script>`, `onerror=`, `javascript:`, `%3Cscript%3E` | [XSS](../web-attacks/xss.md) |
| `;id`, `|whoami`, `$(curl ...)`, `` `uname -a` `` | [Command injection](../web-attacks/command-injection.md) |
| `../../etc/passwd`, `%2e%2e%2f`, `..\..\windows\win.ini` | [Path traversal](../web-attacks/path-traversal.md) |
| `url=http://169.254.169.254/`, `url=http://localhost` | [SSRF](../web-attacks/ssrf.md) |
| Sequential IDs: `/invoice?id=1001`, `1002`, `1003`... by one user | [IDOR](../web-attacks/idor.md) |
| POST of `.php` / `.aspx` / `.jsp` to an upload folder, then GET with `?cmd=` | [Web shell](../web-attacks/file-upload-web-shell.md) |
| Thousands of 404s from one IP | Directory brute forcing ([scanning](../network-attacks/reconnaissance-scanning.md)) |
| Many POSTs to `/login` from one IP | [Brute force](../credential-attacks/brute-force.md) |

## Email

| What you see | Likely attack |
|---|---|
| Sender display name is the CEO but address is external; urgent payment request | [Business email compromise](../initial-access/business-email-compromise.md) |
| Link to a login page on a look-alike domain | [Phishing](../initial-access/phishing.md) / [Session token theft](../cloud-identity/session-token-theft.md) |
| Attachment `.html`, `.iso`, `.zip` with `.lnk`, `.one`, macro Office file | [Phishing](../initial-access/phishing.md) |
| SPF / DKIM / DMARC fail | Spoofed sender ([Phishing](../initial-access/phishing.md)) |
