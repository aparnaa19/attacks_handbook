<sub>[Home](../README.md) › [Initial access](README.md)</sub>

# Lookalike domains (typosquatting)

> **In one line:** The attacker registers a domain that **looks almost exactly like yours or a partner's** (`rnicrosoft.com`, `acme-corp.co`) and uses it for phishing, BEC or fake websites.

| **MITRE ATT&CK** | [T1583.001 Acquire Infrastructure: Domains](https://attack.mitre.org/techniques/T1583/001/) |
|---|---|
| **Tactic** | Resource Development (happens **before** the attack) |
| **Used for** | [Phishing](phishing.md), [BEC](business-email-compromise.md), fake login pages, malware download sites |

## Types of lookalikes

| Type | Real | Fake |
|---|---|---|
| **Typo** | `acmebank.com` | `acmebnak.com`, `acmebank.co` |
| **Character swap** | `microsoft.com` | `rnicrosoft.com` (`rn` looks like `m`), `micr0soft.com` |
| **Homoglyph (IDN)** | `apple.com` | `аpple.com` (Cyrillic `а`), shown in punycode as `xn--pple-43d.com` |
| **Added word** | `acmebank.com` | `acmebank-secure.com`, `acmebank-login.com`, `support-acmebank.com` |
| **Different TLD** | `acmebank.com` | `acmebank.net`, `acmebank.xyz` |
| **Subdomain trick** | `acmebank.com` | `acmebank.com.login-verify.net` (real domain is `login-verify.net`) |

## How it works

1. The attacker registers a lookalike, often days or hours before the attack.
2. They set up email (SPF/DKIM/DMARC that **pass**, because they own the domain) and a website with a valid TLS certificate.
3. They send email that appears to come from a colleague, vendor or brand, or host a fake login page.
4. Users and some filters trust it because it **looks** right and passes authentication checks.

## What you see in logs

| Log source | What to look for |
|---|---|
| Email gateway | Sender domains with a small **edit distance** from your own or your vendors' domains |
| Email gateway | Sender domain **registered recently** (WHOIS creation date within 30 days) |
| Proxy / DNS | Users visiting newly registered domains containing your brand name |
| Certificate transparency | New TLS certificates issued for domains containing your brand (monitor `crt.sh`) |

## How to detect it

- Brand-monitoring: watch certificate transparency logs and new domain registrations for your brand and key vendors (tools like dnstwist generate likely lookalikes).
- Email rules: flag external senders whose domain is "close to" internal or vendor domains; tag external emails.
- Block or warn on **newly registered domains** at the proxy/DNS layer.

## How to respond

1. Block the domain at the email gateway, proxy and DNS.
2. Search for **all emails** from it and all visits to it; follow the [phishing](phishing.md) / [BEC](business-email-compromise.md) response.
3. Request a **takedown** from the registrar/host (abuse contact), or through a brand-protection service.
4. **Fix:** proactively register common typos of your own domain, DMARC `reject` on your domain (stops exact spoofing, not lookalikes), user training on checking the real domain.

## References

- [MITRE ATT&CK T1583.001](https://attack.mitre.org/techniques/T1583/001/)
- [dnstwist (lookalike domain generator)](https://github.com/elceef/dnstwist)

---

| [← Business email compromise (BEC)](business-email-compromise.md) | [Home](../README.md) | [Social engineering →](social-engineering.md) |
|:---|:---:|---:|
