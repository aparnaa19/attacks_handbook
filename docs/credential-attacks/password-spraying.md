# Password spraying

!!! abstract "In one line"
    The attacker tries **one common password against many accounts**, slowly, so no single account gets locked out.

| | |
|---|---|
| **MITRE ATT&CK** | [T1110.003 Password Spraying](https://attack.mitre.org/techniques/T1110/003/) |
| **Tactic** | Credential Access |
| **Common targets** | Microsoft 365 / Entra ID, Outlook Web Access, VPN, ADFS, Citrix, Active Directory via Kerberos or SMB |
| **Common tools** | Kerbrute, NetExec, MSOLSpray, Spray, custom scripts behind rotating proxies |

## How it works

1. The attacker collects a list of usernames: from LinkedIn (`first.last@company.com`), a previous breach, or by enumerating AD.
2. They choose a password many people use: `Winter2026!`, `Company@123`, `Welcome1`.
3. They try that **one** password against **every** account.
4. They wait (for example 30–60 minutes) so the lockout counter resets, then try the next password.
5. With hundreds of users, one or two usually have that password.

**Why it works:** lockout policies count failures **per account**. Spraying stays at 1–2 failures per account, under the threshold.

## What you see in logs

| Log source | What to look for |
|---|---|
| Windows Security | Many **4625** with **many different `TargetUserName`** from one `IpAddress`, sub-status `0xC000006A` (bad password) |
| Domain controller (Kerberos) | Many **4771** with failure code `0x18` across many users (Kerbrute does this; it does **not** create 4625) |
| Domain controller (Kerberos username enumeration) | **4768** with result code `0x6` (user does not exist) for many names |
| Entra ID sign-in logs | Error **50126** (invalid username or password) across many users from the same IP or ASN; **50053** (account locked by smart lockout) |
| Entra ID | Sign-ins using **legacy authentication** (IMAP, POP, SMTP AUTH): they cannot do MFA, so attackers prefer them |
| VPN logs | Failed logins across many usernames from one IP |

## How to detect it

- **Many distinct users, one source:** more than 10 different usernames failing from the same IP in an hour.
- **Same password time pattern:** failures for all accounts appear in "waves" every 30–60 minutes.
- **A success inside the wave:** one account succeeds from the spraying IP. That account is compromised.

```spl title="Example Splunk search"
index=wineventlog EventCode=4625
| bin _time span=1h
| stats dc(user) as distinct_users, count by _time, src_ip
| where distinct_users > 10
```

```spl title="Entra ID sign-ins (field names depend on your add-on)"
index=azure sourcetype=azure:aad:signin status.errorCode=50126
| stats dc(userPrincipalName) as users, count by ipAddress
| where users > 10
```

## How to respond

1. **Find any successful logon** from the spraying IP(s). Those accounts are compromised: reset password, revoke sessions, check what they did.
2. **Block the IP / ASN** at the identity provider or VPN.
3. Check for **MFA bypass**: did the attacker log in through a legacy protocol that skipped MFA?
4. **Fix:** MFA on everything, block legacy authentication, ban common passwords (Entra Password Protection / a banned-password list), use smart lockout.

## Don't confuse with

| Attack | Pattern |
|---|---|
| [Brute force](brute-force.md) | Many passwords → **one** account. Locks accounts quickly |
| **Password spraying** | **One** password → many accounts. Few failures per account |
| [Credential stuffing](credential-stuffing.md) | Different **real** passwords per account, taken from leaks |

## References

- [MITRE ATT&CK T1110.003](https://attack.mitre.org/techniques/T1110/003/)
- [Microsoft: Password spray investigation playbook](https://learn.microsoft.com/en-us/security/operations/incident-response-playbook-password-spray)
