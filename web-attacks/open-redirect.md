<sub>[Home](../README.md) › [Web attacks](README.md)</sub>

# Open redirect

> **In one line:** A trusted website has a redirect feature that sends users to **any URL in a parameter**, so attackers use **the trusted domain's link** to bounce victims to a phishing site.

| **CWE** | CWE-601 URL Redirection to Untrusted Site |
|---|---|
| **MITRE ATT&CK** | Used in [T1566.002 Spearphishing Link](https://attack.mitre.org/techniques/T1566/002/) |
| **Why attackers like it** | The link in the email starts with a domain users and filters **trust** |

## How it works

Many sites redirect after login or when tracking clicks:

```text
https://trusted-bank.example/login?next=/account
https://news.example/click?url=https://partner.example/offer
```

If the site doesn't check the destination, the attacker sends:

```text
https://trusted-bank.example/login?next=https://trusted-bank-secure.example/login
```

1. The user sees `trusted-bank.example` at the start of the link and clicks.
2. The real site immediately redirects them to the attacker's lookalike login page.
3. Email filters that check the first domain may allow it, because the domain has a good reputation.

**Abused services:** search-engine redirect URLs, ad-tracking links, marketing email click trackers, and legitimate "safe link" rewriters are frequently chained in phishing.

## Common parameters and tricks

| Parameter names | Bypass tricks |
|---|---|
| `url`, `next`, `redirect`, `redirect_uri`, `return`, `returnUrl`, `goto`, `dest`, `continue`, `r`, `u` | `//evil.example` (protocol-relative), `https:evil.example`, `trusted.example.evil.example`, `trusted.example@evil.example`, URL-encoded `%2F%2Fevil.example` |

`trusted.example@evil.example` goes to **evil.example**: everything before `@` is treated as a username.

## What you see in logs

| Log source | What to look for |
|---|---|
| Your web server logs | Redirect endpoints with **external domains** in `next=` / `url=` / `redirect=` parameters, often from email referrers |
| Email gateway | Links to trusted domains whose query string contains **another full URL** |
| Proxy | User visits trusted site → **302** → newly registered domain within the same second |
| OAuth logs | `redirect_uri` values not matching the registered app URL (can leak authorization codes) |

## How to respond

1. **Phishing side:** treat as phishing. Find the final landing domain, block it, follow the [phishing](../initial-access/phishing.md) response.
2. **Your own site is the redirector:** report it to the app owner to fix; until then, block the pattern at the WAF.
3. **Fix:** only allow **relative paths** or an **allowlist** of destinations; show a warning page for external redirects; strict exact-match `redirect_uri` for OAuth.

## References

- [OWASP: Unvalidated Redirects and Forwards Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html)

---

| [← Insecure direct object reference (IDOR)](idor.md) | [Home](../README.md) | [File upload attacks and web shells →](file-upload-web-shell.md) |
|:---|:---:|---:|
