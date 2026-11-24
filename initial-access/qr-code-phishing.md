<sub>[Home](../README.md) › [Initial access](README.md)</sub>

# QR code phishing (quishing)

> **In one line:** The phishing link is hidden **inside a QR code image**, so email filters can't read it, and the victim scans it with their **phone**, which has no corporate protection.

| **MITRE ATT&CK** | [T1566.002 Spearphishing Link](https://attack.mitre.org/techniques/T1566/002/) |
|---|---|
| **Tactic** | Initial Access |
| **Usual goal** | Steal Microsoft 365 / Google credentials and MFA session ([AiTM](../cloud-identity/session-token-theft.md)) |
| **Common lures** | "MFA re-enrollment required", "Shared document", "Payroll / benefits update", fake parking meter stickers |

## How it works

1. The email contains **no clickable link**, only an image (or a PDF / Word attachment) with a QR code: "Scan to re-activate your MFA".
2. The email gateway sees an image, not a URL, so it often passes.
3. The user scans it with their **personal phone**.
4. The phone opens a fake login page (often an AiTM proxy). The phone has no EDR, no web proxy, no Safe Links.
5. The user signs in. The attacker gets the password and session token.

**Variations:** QR code built from **ASCII / HTML table cells** instead of an image (harder for scanners), QR codes inside PDFs, physical QR stickers on parking meters or posters.

## What you see in logs

| Log source | What to look for |
|---|---|
| Email gateway | Emails with **one image and very little text**, urgent MFA/HR wording, sender outside the org |
| Email gateway (if it decodes QR) | Decoded URL pointing to a new / lookalike domain |
| Entra ID / IdP sign-in logs | Sign-in from a **mobile device / mobile IP** (carrier network) just after the email, followed by session use from a **different IP** |
| Entra ID Protection | Risky sign-in: anomalous token, unfamiliar properties |
| **Not** in proxy logs | The click happened on the phone, outside your network |

## How to detect it

- Use an email security product that **decodes QR codes** and scans the URL.
- Correlate: user received a QR email → sign-in from a mobile carrier IP → session reused elsewhere.
- User reports. Train users that **IT never asks you to scan a QR code to log in**.

## How to respond

1. Find **all recipients** of the email (same sender / subject / image hash) and purge it.
2. Decode the QR code (any QR reader, or CyberChef "Parse QR Code") and block the domain.
3. For users who scanned and signed in: **revoke sessions**, reset password, remove attacker MFA methods. See [Session token theft](../cloud-identity/session-token-theft.md).
4. **Fix:** QR-aware email filtering, phishing-resistant MFA, Conditional Access requiring compliant devices for sign-in.

## References

- [MITRE ATT&CK T1566.002](https://attack.mitre.org/techniques/T1566/002/)

---

| [← Phishing](phishing.md) | [Home](../README.md) | [HTML smuggling →](html-smuggling.md) |
|:---|:---:|---:|
