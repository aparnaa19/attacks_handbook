<sub>[Home](../README.md) › [Initial access](README.md)</sub>

# Phishing

> **In one line:** A fake message tricks the user into **clicking a link, opening an attachment or entering their password**.

| **MITRE ATT&CK** | [T1566 Phishing](https://attack.mitre.org/techniques/T1566/): .001 attachment, .002 link, .003 via service (Teams, LinkedIn), .004 voice |
|---|---|
| **Tactic** | Initial Access |
| **Most common entry point** | Phishing is one of the top ways attacks start, year after year |

## Types

| Type | Meaning |
|---|---|
| **Mass phishing** | Same generic email to thousands of people ("Your parcel is waiting") |
| **Spear phishing** | Targeted at a specific person, using their name, role, colleagues |
| **Whaling** | Spear phishing aimed at executives |
| **Clone phishing** | Copy of a real email the user received, with the link/attachment swapped |
| **Credential phishing** | Link to a fake login page (Microsoft 365, bank) to steal the password |
| **QR code phishing (quishing)** | Link hidden in a QR code image so email filters can't read it |
| **HTML smuggling** | `.html` attachment that builds the malware file inside the browser, bypassing gateway scanning |
| **Callback phishing** | Email with no link, just a phone number to call ("you were charged $499"). See [Social engineering](social-engineering.md) |

## How it works (attachment example)

1. The user receives an email "Invoice #4471 overdue" with a `.zip` file.
2. Inside is an `.iso` or `.lnk` file (Office macros from the internet are blocked by default since 2022, so attackers moved to containers, shortcuts, OneNote files and HTML).
3. The user opens it. The `.lnk` runs `powershell` or `mshta`, which downloads a loader.
4. The loader connects to the attacker's server ([C2](../network-attacks/c2-beaconing.md)) and the intrusion begins.

**Credential phishing example:** link → fake Microsoft login page on `micros0ft-login[.]com` → user types password → attacker logs in. Modern kits also steal the MFA session, see [Session token theft](../cloud-identity/session-token-theft.md).

## Email headers to check

| Header | What it tells you |
|---|---|
| `From` | Who it **claims** to be from (easy to fake) |
| `Return-Path` / envelope sender | Where bounces go. Mismatch with `From` is a red flag |
| `Reply-To` | Where replies go. Different from `From` = classic phishing/BEC trick |
| `Received` | The servers the email passed through (read bottom to top). First one = real origin |
| `Authentication-Results` | **SPF**, **DKIM**, **DMARC** results |

| Check | What it verifies |
|---|---|
| **SPF** | The sending server's IP is allowed to send for that domain |
| **DKIM** | The email was signed by the domain and not changed in transit |
| **DMARC** | The `From` domain aligns with SPF/DKIM, and what to do if it fails (none / quarantine / reject) |

A lookalike domain (`paypa1.com`) can **pass** SPF, DKIM and DMARC, because the attacker owns it. Passing does not mean safe.

## What you see in logs

| Log source | What to look for |
|---|---|
| Email gateway / M365 | Sender, subject, attachment names and hashes, URLs, verdict, delivered or blocked |
| URL click logs (Safe Links, proxy) | Who clicked, when, and whether the page was blocked |
| Sysmon 1 / EDR | `winword.exe`, `excel.exe`, `outlook.exe`, `explorer.exe` (opening `.lnk`) spawning `powershell`, `cmd`, `mshta`, `wscript`, `rundll32` |
| Sysmon 11 / 15 | Files written to `Downloads` or `Temp` with Mark-of-the-Web |
| Proxy / DNS | Connection to a **newly registered domain** right after the email |
| Identity logs | Sign-in for the user from a new IP / country shortly after the click |

## How to detect it

- Office or mail client spawning script interpreters.
- Attachments of risky types: `.iso`, `.img`, `.lnk`, `.one`, `.hta`, `.js`, `.vbs`, `.html`, password-protected `.zip`.
- Sender domains registered in the last 30 days, or lookalikes of your own domain.
- User reports (the "Report phishing" button).

**Example Splunk search: Office spawning a shell**

```spl
index=sysmon EventCode=1
  ParentImage IN ("*\\winword.exe", "*\\excel.exe", "*\\powerpnt.exe", "*\\outlook.exe", "*\\onenote.exe")
  Image IN ("*\\powershell.exe", "*\\cmd.exe", "*\\mshta.exe", "*\\wscript.exe", "*\\cscript.exe", "*\\rundll32.exe")
| table _time, host, user, ParentImage, Image, CommandLine
```

## How to respond

1. **Analyse the email:** headers, sender, URL (check in a sandbox or URLScan), attachment hash (VirusTotal).
2. **Scope:** message trace for everyone who received the same email (same sender, subject or URL).
3. **Purge** it from all mailboxes. Block the sender, domain, URL and hash.
4. **Find who interacted:** clicked (proxy/click logs), opened the attachment (EDR), entered credentials (sign-in logs).
5. Clicked + entered password → reset password, revoke sessions, check MFA methods. Ran the attachment → isolate the host and investigate.

## References

- [MITRE ATT&CK T1566](https://attack.mitre.org/techniques/T1566/)

---

| [← Initial access](README.md) | [Home](../README.md) | [QR code phishing (quishing) →](qr-code-phishing.md) |
|:---|:---:|---:|
