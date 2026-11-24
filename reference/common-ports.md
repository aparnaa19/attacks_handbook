<sub>[Home](../README.md) › Quick reference</sub>

# Common ports

| Port | Protocol | Service | Attack notes |
|---|---|---|---|
| 20, 21 | TCP | FTP | Clear-text credentials, anonymous login |
| 22 | TCP | SSH | [Brute force](../credential-attacks/brute-force.md) target, tunneling |
| 23 | TCP | Telnet | Clear text. Botnets (Mirai) brute force it on IoT devices |
| 25, 465, 587 | TCP | SMTP | Email sending. Outbound 25 from a workstation = possible spam bot |
| 53 | UDP / TCP | DNS | [DNS tunneling](../network-attacks/dns-tunneling.md), DNS amplification DDoS |
| 67, 68 | UDP | DHCP | Rogue DHCP server (MITM) |
| 80 | TCP | HTTP | Web attacks, clear-text C2 |
| 88 | TCP / UDP | Kerberos | [Kerberoasting](../credential-attacks/kerberoasting.md), ticket attacks |
| 110, 995 | TCP | POP3 / POP3S | Mail access |
| 123 | UDP | NTP | NTP amplification DDoS |
| 135 | TCP | MS-RPC | WMI, DCOM lateral movement |
| 137, 138, 139 | UDP / TCP | NetBIOS | [NBT-NS poisoning](../network-attacks/llmnr-nbtns-poisoning.md), old SMB |
| 143, 993 | TCP | IMAP / IMAPS | Mail access, password spraying against legacy auth |
| 161, 162 | UDP | SNMP | Default community strings (`public`), information leak |
| 389, 636 | TCP | LDAP / LDAPS | AD enumeration |
| 443 | TCP | HTTPS | Most C2 hides here |
| 445 | TCP | SMB | [Lateral movement](../post-exploitation/lateral-movement.md), [worms](../malware/worms.md) (WannaCry / EternalBlue) |
| 1433 | TCP | Microsoft SQL | Brute force, `xp_cmdshell` |
| 3306 | TCP | MySQL | |
| 3389 | TCP | RDP | Brute force, lateral movement. Should never be internet-facing |
| 4444 | TCP | — | Metasploit default listener port |
| 5355 | UDP | LLMNR | [LLMNR poisoning](../network-attacks/llmnr-nbtns-poisoning.md) |
| 5432 | TCP | PostgreSQL | |
| 5985, 5986 | TCP | WinRM (HTTP / HTTPS) | PowerShell remoting, lateral movement |
| 6379 | TCP | Redis | Often exposed with no password |
| 8080, 8443 | TCP | Alt HTTP / HTTPS | Admin panels, proxies |
| 9200 | TCP | Elasticsearch | Exposed databases, data leaks |
| 27017 | TCP | MongoDB | Exposed databases, data leaks |

> [!TIP]
> **Ports are a hint, not proof**
>
> Attackers run services on any port. C2 on 443 looks like normal HTTPS; SSH on 443 bypasses firewalls. Always check **which process** made the connection (Sysmon 3 / EDR).

---

| [← Log sources](log-sources.md) | [Home](../README.md) | [Initial access →](../initial-access/README.md) |
|:---|:---:|---:|
