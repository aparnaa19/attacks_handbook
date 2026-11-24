<sub>[Home](../README.md) › [Cloud and identity](README.md)</sub>

# OAuth consent phishing

> **In one line:** The attacker tricks a user into clicking **"Accept"** on a real Microsoft/Google permissions screen, giving the **attacker's app** access to the user's mail and files. The access keeps working **even after a password reset or MFA**.

| **MITRE ATT&CK** | [T1528 Steal Application Access Token](https://attack.mitre.org/techniques/T1528/), [T1550.001 Application Access Token](https://attack.mitre.org/techniques/T1550/001/) |
|---|---|
| **Tactics** | Credential Access, Persistence |
| **Also called** | Illicit consent grant |

## How it works

1. The attacker registers an app in their own cloud tenant with a trustworthy name and logo: "Secure Document Viewer", "Adobe Sign Plugin".
2. They send the user a link. It goes to the **real** Microsoft login domain, so URL checks look fine.
3. The user signs in (password + MFA, all real) and sees a consent screen: *"Secure Document Viewer wants to: Read your mail, Read and write your files, Maintain access to data you have given it access to."*
4. The user clicks **Accept**.
5. The app receives **access and refresh tokens**. The attacker reads mail and files through the API, from anywhere.
6. Resetting the user's password **does not** remove the app's access. The consent must be revoked.

**Dangerous permissions to watch for:** `Mail.Read`, `Mail.ReadWrite`, `Mail.Send`, `Files.ReadWrite.All`, `Contacts.Read`, `MailboxSettings.ReadWrite`, `offline_access` (long-lived refresh token).

**Related: device code phishing.** The attacker sends a code and the link `microsoft.com/devicelogin`; the user enters the code and signs in, which gives the **attacker's device** a token. Also uses the real Microsoft site.

## What you see in logs

| Log source | What to look for |
|---|---|
| Entra ID audit log | **"Consent to application"**, **"Add delegated permission grant"**, **"Add service principal"** for an app with an unfamiliar name or publisher |
| Entra ID audit log | Consent with high-risk scopes (mail, files, `offline_access`) by a normal user |
| M365 Unified Audit Log | `MailItemsAccessed` / file access by that **App ID** from unfamiliar IPs |
| Entra ID sign-in logs | Service principal sign-ins; for device code phishing, sign-ins using **device code flow** that the user didn't initiate |

## How to detect it

- Alert on every user consent to an app that is **not from a verified publisher** or requests mail/file scopes.
- Regularly review **Enterprise applications** for apps with few users and high permissions.

## How to respond

1. **Revoke the consent / remove the app's permissions**, and disable or delete the service principal in your tenant.
2. **Revoke the user's sessions and tokens.**
3. Review what the app accessed (audit logs by App ID): which mailboxes, which files.
4. Search for **other users** who consented to the same app.
5. Report the app to Microsoft / Google.
6. **Fix:** restrict user consent (allow only verified publishers and low-risk permissions, or require **admin consent** for everything), admin consent workflow, block device code flow where not needed (Conditional Access).

## References

- [MITRE ATT&CK T1528](https://attack.mitre.org/techniques/T1528/)
- [Microsoft: Detect and remediate illicit consent grants](https://learn.microsoft.com/en-us/defender-office-365/detect-and-remediate-illicit-consent-grants)

---

| [← Session token theft (AiTM)](session-token-theft.md) | [Home](../README.md) | [Cloud misconfiguration →](cloud-misconfiguration.md) |
|:---|:---:|---:|
