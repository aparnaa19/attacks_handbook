<sub>[Home](../README.md)</sub>

# Cloud and identity

In cloud and SaaS environments (Microsoft 365, Google Workspace, AWS, Azure), **identity is the perimeter**: if the attacker has a valid session or key, they don't need malware or network access.

| Attack | What is stolen or abused | MITRE |
|---|---|---|
| [Session token theft (AiTM)](session-token-theft.md) | Logged-in session cookies, which skip MFA | T1557, T1539, T1550.004 |
| [OAuth consent phishing](oauth-consent-phishing.md) | User grants a malicious app access to their mail/files | T1528 |
| [Cloud misconfiguration](cloud-misconfiguration.md) | Public storage, leaked access keys, over-permissive roles | T1530, T1552.001, T1078.004 |

Related pages: [MFA fatigue](../credential-attacks/mfa-fatigue.md), [Password spraying](../credential-attacks/password-spraying.md), [BEC](../initial-access/business-email-compromise.md), [SSRF to cloud metadata](../web-attacks/ssrf.md), [Infostealers](../malware/infostealers.md) (cookie theft).

## Key cloud log sources

| Platform | Log | What it records |
|---|---|---|
| Microsoft Entra ID | **Sign-in logs** | Every login: user, IP, location, app, MFA result, error code |
| Microsoft Entra ID | **Audit logs** | Directory changes: users, groups, app consents, MFA methods, roles |
| Microsoft 365 | **Unified Audit Log** | Mailbox rules, file access in SharePoint/OneDrive, Teams, admin actions |
| AWS | **CloudTrail** | Every API call: who, what, from which IP |
| AWS | **GuardDuty** | Managed threat detection findings |
| Azure | **Activity Log** | Changes to Azure resources |
| Google Workspace | **Admin / Login audit** | Logins, admin actions, Drive sharing |
| Okta | **System Log** | Logins, MFA, admin changes |

## Useful Entra ID sign-in error codes

| Code | Meaning |
|---|---|
| `0` | Success |
| `50126` | Invalid username or password |
| `50053` | Account locked (smart lockout) or sign-in from a malicious IP blocked |
| `50057` | Account disabled |
| `50074` | Strong authentication (MFA) required |
| `500121` | MFA failed / user didn't complete or denied it |
| `53003` | Blocked by Conditional Access |

---

| [← Data exfiltration](../post-exploitation/data-exfiltration.md) | [Home](../README.md) | [Session token theft (AiTM) →](session-token-theft.md) |
|:---|:---:|---:|
