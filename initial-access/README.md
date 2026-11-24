<sub>[Home](../README.md)</sub>

# Initial access

How attackers get their **first foothold** in the organisation. Stop it here and nothing else in this handbook happens.

| Attack | How they get in | MITRE |
|---|---|---|
| [Phishing](phishing.md) | Email with a malicious link or attachment | T1566 |
| [Business email compromise](business-email-compromise.md) | Fake or hijacked email account asks for money or data | T1534, T1657 |
| [Social engineering](social-engineering.md) | Phone calls, SMS, help desk tricks, fake IT support | T1566.004 |
| [Drive-by compromise](drive-by-compromise.md) | Visiting a malicious or hacked website, fake downloads | T1189 |
| [Exploiting public-facing apps](exploit-public-facing-app.md) | Hacking an internet-facing server, VPN or firewall | T1190 |
| [Supply chain attack](supply-chain.md) | Malicious code inside software or updates you trust | T1195 |

Other ways in, covered elsewhere:

- **Valid accounts** (T1078): logging in with stolen or guessed passwords. See [Credential attacks](../credential-attacks/README.md).
- **External remote services** (T1133): exposed RDP, VPN, Citrix with weak credentials. See [Brute force](../credential-attacks/brute-force.md).
- **Session token theft**: see [Cloud and identity](../cloud-identity/README.md).

## The first question for any initial access alert

**Did it execute or succeed?** An email that was delivered but never opened, or an exploit attempt against a patched server, is a very different incident from one that ran code. Check:

- Endpoint: did a child process start (Office → PowerShell, web server → cmd)?
- Network: did the host connect out to a new domain or IP right after?
- Identity: did someone log in with the user's account from somewhere new?

---

| [← Common ports](../reference/common-ports.md) | [Home](../README.md) | [Phishing →](phishing.md) |
|:---|:---:|---:|
