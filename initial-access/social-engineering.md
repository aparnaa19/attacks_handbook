<sub>[Home](../README.md) › [Initial access](README.md)</sub>

# Social engineering

> **In one line:** Manipulating **people** instead of computers: phone calls, SMS, fake IT support and help desk tricks to get passwords, MFA codes or remote access.

| **MITRE ATT&CK** | [T1566.004 Spearphishing Voice](https://attack.mitre.org/techniques/T1566/004/), [T1219 Remote Access Tools](https://attack.mitre.org/techniques/T1219/), [T1656 Impersonation](https://attack.mitre.org/techniques/T1656/) |
|---|---|
| **Known use** | Scattered Spider calling help desks to reset MFA (MGM Resorts, 2023) |

## Types

| Type | What happens |
|---|---|
| **Vishing** (voice phishing) | Caller pretends to be IT, the bank, or a manager and asks for a password or MFA code |
| **Smishing** (SMS phishing) | Text message with a link: "Your account is locked, verify here" |
| **Help desk social engineering** | Attacker calls the help desk **pretending to be an employee** and asks for a password or MFA reset |
| **Callback phishing** | Email says "You were charged $499, call to cancel". The "support agent" guides the victim to install a remote tool |
| **Fake IT support** | Spam floods the inbox, then a "helpdesk" call on Teams offers to fix it and asks the user to start Quick Assist / AnyDesk |
| **Pretexting** | A made-up but believable story (auditor, new vendor, new employee) to get information |
| **Baiting** | Infected USB drives left where employees will find them |
| **Tailgating / piggybacking** | Walking into a secure building behind an employee |

## How it works (help desk example)

1. The attacker gathers details about an employee from LinkedIn and leaked data (employee ID, manager's name).
2. They call the help desk: "I got a new phone, I can't log in, please reset my MFA."
3. The help desk resets MFA. The attacker registers **their own** phone.
4. They log in with full access, often to an IT admin account.

## What you see in logs

| Log source | What to look for |
|---|---|
| Help desk tickets | MFA or password reset requests, especially for admins, right before suspicious activity |
| Identity logs | **New MFA method registered**, then sign-in from new device / location / IP |
| EDR / Sysmon 1 | **Remote access software** installed or run: AnyDesk, ScreenConnect, TeamViewer, Quick Assist (`quickassist.exe`) |
| Teams / chat logs | External accounts named "IT Support" / "Help Desk" messaging employees |
| Email | Spam flood to one user followed by an external Teams call |

## How to detect it

- MFA reset + new device + sign-in from a new location within a short window.
- Remote access tools that are **not** your company's approved tool.
- External Teams chats from accounts with "support" / "helpdesk" in the name.

## How to respond

1. Contact the **real** user through a known channel and confirm whether they made the request.
2. If not: revoke sessions, reset password, remove the attacker's MFA method, check what was accessed.
3. If remote tooling was installed: isolate the host and treat it as hands-on-keyboard compromise.
4. **Fix:** help desk identity verification (video call, manager approval, callback to number on file), block unapproved remote tools, restrict external Teams access, user awareness training.

## References

- [MITRE ATT&CK T1566.004](https://attack.mitre.org/techniques/T1566/004/)
- [CISA advisory on Scattered Spider](https://www.cisa.gov/news-events/cybersecurity-advisories/aa23-320a)

---

| [← Business email compromise](business-email-compromise.md) | [Home](../README.md) | [Drive-by compromise →](drive-by-compromise.md) |
|:---|:---:|---:|
