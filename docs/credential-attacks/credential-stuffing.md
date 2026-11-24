# Credential stuffing

!!! abstract "In one line"
    The attacker takes **username + password pairs leaked from other websites** and tries them on your login page, betting that people reuse passwords.

| | |
|---|---|
| **MITRE ATT&CK** | [T1110.004 Credential Stuffing](https://attack.mitre.org/techniques/T1110/004/) |
| **Tactic** | Credential Access |
| **Common targets** | Customer-facing web apps (banking, retail, streaming), SSO portals, VPN |
| **Common tools** | OpenBullet, SentryMBA, custom bots, residential proxy networks |

## How it works

1. A website gets breached and its users' emails and passwords are published or sold ("combo lists").
2. The attacker loads millions of these pairs into a bot.
3. The bot tries each pair **once** on your login page, from thousands of different IPs (residential proxies) so no single IP looks noisy.
4. Users who reused the same password on your site are logged in by the attacker.
5. The attacker takes over the account: changes email, steals stored payment data, redeems points, or sells the account.

## What you see in logs

| Log source | What to look for |
|---|---|
| Web app / auth logs | Spike in login attempts and **failure ratio** (often 95%+ failures) |
| Web app | High rate of **"user does not exist"** failures: combo lists contain emails that never registered on your site |
| Web server / WAF | Many IPs, but the **same User-Agent**, same TLS fingerprint (JA3/JA4), same request pattern |
| Web app | Successful logins followed quickly by **email change, password change, MFA change or payout change** |
| Identity provider | Many users with one failure each, from many IPs; logins from hosting providers or proxy ASNs |

## How to detect it

- Login attempts per minute far above normal.
- Failure rate jumps for the whole site, not one account.
- Many accounts each tried **once**, spread across many IPs.
- Successful logins from IPs that also produced many failures for other users.

```spl title="Example Splunk search: IPs failing across many accounts"
index=webapp action=login result=failure
| stats dc(user) as users, count by src_ip, http_user_agent
| where users > 5
| sort - users
```

## How to respond

1. **Identify accounts that logged in successfully** from the attack sources. Force a password reset and kill their sessions.
2. Check those accounts for **changes** (email, phone, payout, shipping address) and roll them back.
3. Add **rate limiting, CAPTCHA or bot protection** on the login page.
4. **Fix:** MFA, check new passwords against known-breached lists (e.g. Have I Been Pwned's Pwned Passwords), alert users on new-device logins.

## Don't confuse with

| Attack | Difference |
|---|---|
| [Brute force](brute-force.md) | Guesses many passwords for one account |
| [Password spraying](password-spraying.md) | Same guessed password for every account |
| **Credential stuffing** | **Real** passwords from leaks, **one** try per account, usually **many IPs** |

## References

- [MITRE ATT&CK T1110.004](https://attack.mitre.org/techniques/T1110/004/)
- [OWASP: Credential Stuffing Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Credential_Stuffing_Prevention_Cheat_Sheet.html)
