<sub>[Home](../README.md) › Quick reference</sub>

# Log sources

Which log answers which question.

| Question | Log source |
|---|---|
| Who logged in, where, how? | Windows Security (4624/4625), Linux `auth.log`, Entra ID / Okta sign-in logs, VPN logs |
| What program ran, with what command line? | Sysmon 1, Windows 4688, EDR process telemetry |
| Which process talked to which IP? | Sysmon 3, EDR network telemetry |
| Which IPs talked to each other, how much data? | Firewall logs, NetFlow, Zeek `conn.log` |
| Which domains were looked up? | DNS server logs, Sysmon 22, Zeek `dns.log` |
| Which websites did users visit? | Web proxy logs (URL, user, bytes, category) |
| What did the web server receive? | Web server access logs (Apache, Nginx, IIS), WAF logs |
| What emails came in, with what attachments and links? | Email gateway logs, Microsoft 365 message trace |
| What happened in the cloud account? | AWS CloudTrail, Azure Activity Log, Microsoft 365 Unified Audit Log, GCP Audit Logs |
| What files were opened / changed on a server? | Windows 4663 (with auditing), Linux `auditd`, EDR file events |
| What changed in Active Directory? | Domain controller Security log (4720, 4728, 4738, 4662) |
| What did antivirus block? | Defender 1116/1117, EDR alerts |

## Network logs in detail

| Source | Gives you | Does **not** give you |
|---|---|---|
| **Firewall** | Source/destination IP and port, allow/deny, bytes | Which process, which URL |
| **NetFlow** | Who talked to whom, how long, how many bytes | Content |
| **Web proxy** | Full URL, user, User-Agent, category | Non-web traffic |
| **Zeek** | Parsed protocol logs: `conn`, `dns`, `http`, `ssl`, `files`, `smb` | Payload (unless you also keep PCAP) |
| **IDS (Snort / Suricata)** | Signature alerts on known-bad traffic | Anything without a signature |
| **PCAP** | Everything, full packets | Easy search at scale (very large) |

## Web server access log format

```text
203.0.113.5 - - [24/Nov/2026:10:15:32 +0000] "GET /login.php?id=1' OR 1=1-- HTTP/1.1" 200 5320 "-" "sqlmap/1.8"
   │                         │                          │                                   │    │      │
 client IP                 time                    method + URL                          status size  User-Agent
```

| Status | Meaning | Attack signal |
|---|---|---|
| 200 | OK | Attack request returned 200 = it may have worked |
| 301 / 302 | Redirect | |
| 400 | Bad request | Malformed attack payloads |
| 401 / 403 | Unauthorised / forbidden | Many = brute force or forced browsing |
| 404 | Not found | Many = directory scanning (Gobuster, Dirb) |
| 500 | Server error | Injection payloads breaking the app (SQLi often causes 500s) |

---

| [← Linux logs](linux-logs.md) | [Home](../README.md) | [Common ports →](common-ports.md) |
|:---|:---:|---:|
