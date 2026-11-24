<sub>[Home](../README.md) › [Initial access](README.md)</sub>

# Business email compromise (BEC)

> **In one line:** The attacker **pretends to be (or takes over) a trusted email account**, an executive, a vendor or a lawyer, and asks someone to **send money or sensitive data**.

| **MITRE ATT&CK** | [T1534 Internal Spearphishing](https://attack.mitre.org/techniques/T1534/), [T1114.003 Email Forwarding Rule](https://attack.mitre.org/techniques/T1114/003/), [T1564.008 Email Hiding Rules](https://attack.mitre.org/techniques/T1564/008/), [T1657 Financial Theft](https://attack.mitre.org/techniques/T1657/) |
|---|---|
| **Usually contains** | **No malware and no link**. Only text, so email filters often miss it |

## Common types

| Type | Example |
|---|---|
| **CEO fraud** | "This is the CEO. I need you to wire $48,000 to this account today. Keep it confidential." |
| **Vendor / invoice fraud** | A real supplier's mailbox is hacked; it replies in an existing thread: "Our bank details have changed." |
| **Payroll diversion** | "Hi HR, please update my direct deposit to this new account." |
| **Gift card scam** | "I need 10 Apple gift cards for clients, send me the codes." |
| **Data theft** | "Send me all employees' W-2 forms." |

## How it works

1. **Two ways in:** a **lookalike domain** (`acme-corp.co` instead of `acmecorp.com`) / spoofed display name, **or** a **real mailbox taken over** with stolen credentials.
2. In a taken-over mailbox, the attacker reads email quietly to learn who pays whom and when.
3. They create **inbox rules** that hide replies (move emails with "invoice", "payment", "bank" to an obscure folder like *RSS Feeds*, or delete them) so the real owner doesn't notice.
4. At the right moment they send the payment request, often replying inside a real thread.
5. Finance pays. The money is moved out quickly.

## What you see in logs

| Log source | What to look for |
|---|---|
| Email headers | `Reply-To` different from `From`; display name of an executive with an external address |
| Microsoft 365 Unified Audit Log | `New-InboxRule` / `Set-InboxRule` with forward, redirect, delete or move-to-folder actions, keywords like *invoice, payment, wire* |
| M365 audit | `Set-Mailbox` with `ForwardingSmtpAddress` to an external address |
| M365 audit | `MailItemsAccessed` in large volume from an unusual IP |
| Entra ID sign-in logs | Sign-in from a new country / hosting provider IP before the rules appear |

## How to detect it

- Inbox rules that **forward externally** or **move/delete** messages with finance keywords.
- Mailbox access from new locations followed by rule creation.
- Emails with executive names from external domains; domains that look like yours or your vendors'.

**Example Splunk search (O365 add-on)**

```spl
index=o365 Operation IN ("New-InboxRule", "Set-InboxRule", "Set-Mailbox")
| table _time, UserId, ClientIP, Operation, Parameters
```

## How to respond

1. **Money first:** if a payment was sent, finance must contact the bank **immediately** to try to recall it. Speed matters (hours).
2. **Contain the mailbox:** reset password, revoke sessions, remove malicious inbox rules and forwarding, check MFA methods.
3. **Scope:** what did the attacker read, send and to whom? Warn recipients of any fraudulent emails sent from the account.
4. If a **vendor** was compromised, tell the vendor.
5. **Fix:** verify any bank-detail change by **calling a known phone number** (never the one in the email), MFA, block external auto-forwarding, tag external emails, DMARC on your domain.

## References

- [MITRE ATT&CK T1114.003](https://attack.mitre.org/techniques/T1114/003/)
- [FBI IC3: Business Email Compromise](https://www.ic3.gov/)

---

| [← HTML smuggling](html-smuggling.md) | [Home](../README.md) | [Lookalike domains (typosquatting) →](lookalike-domains.md) |
|:---|:---:|---:|
