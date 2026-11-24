<sub>[Home](../README.md) › [Cloud and identity](README.md)</sub>

# Golden SAML

> **In one line:** The attacker steals the **token-signing key** of the company's identity provider (e.g. AD FS) and **forges SAML logins** as any user to any connected cloud app, bypassing passwords and MFA.

| **MITRE ATT&CK** | [T1606.002 Forge Web Credentials: SAML Tokens](https://attack.mitre.org/techniques/T1606/002/), [T1552.004 Private Keys](https://attack.mitre.org/techniques/T1552/004/) |
|---|---|
| **Tactics** | Credential Access, Persistence |
| **Needs** | Admin on the identity provider server (e.g. AD FS) to steal its signing certificate |
| **Known use** | SolarWinds / Nobelium (2020) |
| **Tools** | AADInternals, ADFSDump, ADFSpoof, shimit |

## Background

With **SAML single sign-on**, the company's identity provider (IdP, e.g. AD FS) checks the user's password and MFA, then gives the user a **signed SAML token**. Cloud apps (Microsoft 365, AWS, Salesforce...) trust **any token signed with the IdP's key**. They never see the password.

## How it works

1. The attacker compromises the AD FS server (or gets Domain Admin).
2. They export the **token-signing certificate and private key** (from the AD FS configuration database, decrypted with the key stored in AD).
3. On their own machine, they **create a SAML token** saying "this is the CEO, MFA completed", and sign it with the stolen key.
4. They present it to Microsoft 365 / AWS. The app verifies the signature: valid. They're in as the CEO.
5. This works **for any user**, **without MFA**, and keeps working after password resets, until the signing certificate is replaced.

It is the SAML version of the [golden ticket](../credential-attacks/golden-silver-ticket.md).

## What you see in logs

| Log source | What to look for |
|---|---|
| AD FS server | Export of the signing certificate; access to the AD FS DKM key in AD (directory service access events); tools like `ADFSDump` / AADInternals; **Sysmon / EDR** on the AD FS server |
| AD FS logs (event **1200 / 1202**, with auditing on) | **Missing**: a forged token was never issued by AD FS, so there's **no matching issuance** |
| Cloud sign-in logs (Entra ID, AWS CloudTrail `AssumeRoleWithSAML`) | Federated sign-ins that have **no corresponding AD FS 1200 event**; sign-ins from unusual IPs claiming MFA |
| Entra ID audit | Changes to **federation settings** or a new trusted federated domain (a related persistence trick) |

## How to detect it

- **Correlate** every cloud federated sign-in with an AD FS token issuance. A sign-in with no matching issuance = forged token.
- Treat AD FS servers as **Tier 0** (same protection and monitoring as domain controllers).
- Alert on federation configuration changes.

## How to respond

1. Treat as **identity-provider compromise**: the attacker can be anyone.
2. **Rotate the token-signing certificate twice** (so the old key is no longer trusted), and update every app that trusts it.
3. Revoke sessions for affected users; review cloud activity for all federated sign-ins in the period.
4. Investigate how the AD FS server / Domain Admin was compromised.
5. **Fix:** protect AD FS like a DC, store signing keys in an HSM, or move to cloud-native authentication (e.g. Entra ID) instead of on-prem federation.

## References

- [MITRE ATT&CK T1606.002](https://attack.mitre.org/techniques/T1606/002/)
- [CISA: Detecting post-compromise threat activity in Microsoft cloud environments (AA21-008A)](https://www.cisa.gov/news-events/cybersecurity-advisories/aa21-008a)

---

| [← OAuth consent phishing](oauth-consent-phishing.md) | [Home](../README.md) | [Cloud misconfiguration and key abuse →](cloud-misconfiguration.md) |
|:---|:---:|---:|
