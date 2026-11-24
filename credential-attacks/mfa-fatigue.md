<sub>[Home](../README.md) › [Credential attacks](README.md)</sub>

# MFA fatigue (push bombing)

> **In one line:** The attacker already has the password and **sends MFA push notifications over and over** until the user taps "Approve" to make them stop.

| **MITRE ATT&CK** | [T1621 Multi-Factor Authentication Request Generation](https://attack.mitre.org/techniques/T1621/) |
|---|---|
| **Tactic** | Credential Access |
| **Common targets** | Microsoft Authenticator, Okta Verify, Duo push |
| **Known use** | Lapsus$ (2022), including the Uber breach |

## How it works

1. The attacker gets a valid password (phishing, infostealer, [credential stuffing](credential-stuffing.md)).
2. They log in. The user's phone gets an MFA push.
3. The user denies it. The attacker logs in again. And again, often late at night.
4. Sometimes the attacker **messages or calls the user pretending to be IT**: "Please approve the prompt to stop the alerts."
5. The user approves. The attacker is in.
6. Often the attacker then **registers their own MFA device** on the account so they don't need the user again.

## What you see in logs

| Log source | What to look for |
|---|---|
| Entra ID sign-in logs | Many sign-ins with error **500121** (MFA failed / user declined), then a success |
| Okta System Log | Many `system.push.send_factor_verify_push`, then `user.mfa.okta_verify.deny_push`, then a success |
| Duo | Many pushes with result "denied" or "timeout" for one user |
| Identity logs | **New MFA method registered** right after the approval |
| User reports | "Report fraud" on the push, or user tells the helpdesk about strange prompts |

## How to detect it

- More than ~5 MFA prompts for one user in 10–15 minutes.
- **Denials followed by an approval**, especially from a new IP, country or device.
- MFA approval at an unusual hour.
- A new authenticator device added shortly after a suspicious approval.

**Example Splunk search (Okta)**

```spl
index=okta eventType=system.push.send_factor_verify_push
| bin _time span=15m
| stats count by _time, actor.alternateId
| where count > 5
```

## How to respond

1. **Revoke all sessions** for the user and **reset the password**. The password is definitely known to the attacker.
2. **Remove MFA methods** the user didn't register.
3. Review what the attacker did after logging in: mailbox rules, file downloads, new apps, other sign-ins.
4. Talk to the user: did someone contact them?
5. **Fix:** enable **number matching** (user must type a number shown on the login screen), show location in the prompt, move high-value users to **phishing-resistant MFA** (FIDO2 security keys, passkeys).

## Don't confuse with

- [Session token theft (AiTM)](../cloud-identity/session-token-theft.md): the user completes MFA themselves on a fake page. No push spam.

## References

- [MITRE ATT&CK T1621](https://attack.mitre.org/techniques/T1621/)

---

| [← Credential stuffing](credential-stuffing.md) | [Home](../README.md) | [Credential dumping →](credential-dumping.md) |
|:---|:---:|---:|
