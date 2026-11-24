# SOC Attack Handbook

A theory handbook for SOC analysts. Every attack page answers the same questions:

1. **What is it?** One line, then the attacker's steps in order.
2. **What does it look like in logs?** The event IDs, log lines and network signs.
3. **How do I detect it?** The pattern to look for, with an example Splunk search.
4. **How do I respond?** What to check first, how to contain it, how to fix the cause.
5. **What is it often confused with?** Similar attacks and how to tell them apart.

## Stuck on an alert?

Start at **[I see X, what is it?](reference/symptom-lookup.md)**. It maps what you see in a log or alert to the attack page that explains it.

## Attack categories

| Category | What's in it |
|---|---|
| [Initial access](initial-access/index.md) | How attackers get in the first time: phishing, BEC, social engineering, drive-by, exploited servers, supply chain |
| [Credential attacks](credential-attacks/index.md) | Guessing, stealing and reusing passwords, hashes and Kerberos tickets |
| [Malware](malware/index.md) | Ransomware, RATs, worms, infostealers, fileless malware, PowerShell and LOLBin abuse |
| [Web attacks](web-attacks/index.md) | SQLi, XSS, command injection, path traversal, SSRF, CSRF, IDOR, web shells |
| [Network attacks](network-attacks/index.md) | Scanning, DoS, MITM, LLMNR poisoning, DNS tunneling, C2 beaconing |
| [Post-exploitation](post-exploitation/index.md) | What happens after the attacker is in: privilege escalation, persistence, lateral movement, evasion, exfiltration |
| [Cloud and identity](cloud-identity/index.md) | Session token theft, OAuth consent phishing, cloud misconfiguration |
| [Insider threat](insider-threat/index.md) | Employees or contractors misusing their access |

## Quick reference

- [Windows event IDs](reference/windows-event-ids.md): the IDs you will see every day, plus logon types and failure codes
- [Sysmon event IDs](reference/sysmon-event-ids.md)
- [Linux logs](reference/linux-logs.md): which file has what
- [Log sources](reference/log-sources.md): which log answers which question
- [Common ports](reference/common-ports.md)

## Basics

- [Attack lifecycle](basics/attack-lifecycle.md): the Cyber Kill Chain and MITRE ATT&CK tactics, so you know where an attack fits
- [Key SOC terms](basics/key-terms.md): IOC, IOA, TTP, true/false positive and more

!!! note "About the Splunk searches"
    The example searches use common field names (`src_ip`, `user`, `dest`). Your environment may name them differently depending on the add-on that parses the logs, for example `IpAddress` or `Account_Name` for raw Windows events. Treat them as patterns to adapt, not copy-paste rules.
