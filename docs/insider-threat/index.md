# Insider threat

!!! abstract "In one line"
    Harm caused by someone with **legitimate access**: an employee, contractor or partner who misuses it on purpose, by accident, or because their account was taken over.

| | |
|---|---|
| **MITRE ATT&CK** | No single technique. Insiders use [T1078 Valid Accounts](https://attack.mitre.org/techniques/T1078/), then exfiltration techniques such as [T1052.001 USB](https://attack.mitre.org/techniques/T1052/001/) and [T1567 Web Service](https://attack.mitre.org/techniques/T1567/) |
| **Why it's hard** | The person is **allowed** to log in and access the data. Only the **pattern** is wrong |

## Three types

| Type | Example |
|---|---|
| **Malicious** | Employee copies the customer database before leaving for a competitor; admin sabotages systems after being fired; fraud |
| **Negligent / accidental** | Emails a spreadsheet of customer data to the wrong person; uploads work files to a personal cloud "to work from home"; falls for phishing |
| **Compromised** | An attacker controls the account. Looks like an insider until you notice the IP, device or behaviour doesn't match the real person |

## Technical indicators

| Indicator | Log source |
|---|---|
| **Mass download** of files, far more than the person's normal | SharePoint / OneDrive audit (`FileDownloaded`, `FileSyncDownloadedFull`), file server 5145 / 4663 |
| Access to data **outside their role** (HR files by an engineer) | File / database audit logs |
| **Uploads to personal cloud or webmail** (personal Gmail, Dropbox, WeTransfer) | Proxy, CASB, DLP |
| **USB** storage use, especially large copies | 6416, EDR device control, DLP |
| **Forwarding** work email to a personal address | M365 audit (inbox rules), email gateway |
| **Large print jobs** | Print server logs |
| Activity at **unusual hours** or after **resignation notice** | All sources + HR context |
| Attempts to **disable monitoring** or access security tools | [Defense evasion](../post-exploitation/defense-evasion.md) signs |
| **Privilege abuse** by admins: creating accounts, granting themselves access | AD audit (4720, 4728, 4732) |

**UEBA** (User and Entity Behaviour Analytics) tools build a baseline for each user and score deviations, which is the main way to catch insiders.

## How to detect it

- Compare each user to **their own baseline** and to **peers in the same role**.
- Combine technical signals with **HR context** (resignation, termination, performance issues), shared through an agreed process with HR and legal.
- High-risk moment: the weeks **before and after** someone resigns.

```spl title="Example Splunk search: unusual download volume (M365)"
index=o365 Operation IN ("FileDownloaded", "FileSyncDownloadedFull")
| bin _time span=1d
| stats count as files by _time, UserId
| eventstats avg(files) as avg_files, stdev(files) as sd_files by UserId
| where files > avg_files + (3 * sd_files) AND files > 100
```

## How to respond

Insider cases are handled differently from external attacks:

1. **Involve HR and legal early.** They decide the next steps; the SOC provides evidence.
2. **Keep it confidential.** Need-to-know only; don't alert the person.
3. **Preserve evidence carefully** (chain of custody). It may be used in legal action.
4. **Contain** as directed: restrict access, disable the account at the agreed moment (often timed with an HR meeting).
5. For **negligent** cases: fix the process and train; for **compromised** accounts: handle as an external compromise.
6. **Fix:** least privilege, offboarding process that removes access immediately, DLP, block personal cloud uploads and USB storage where possible, logging of sensitive data access.

## References

- [CISA: Insider Threat Mitigation](https://www.cisa.gov/topics/physical-security/insider-threat-mitigation)
