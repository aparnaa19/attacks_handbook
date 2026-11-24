<sub>[Home](../README.md) › [Web attacks](README.md)</sub>

# Cross-site request forgery (CSRF)

> **In one line:** A malicious website makes the victim's browser **send a request to another site where the victim is logged in**, so the action runs with the victim's session (change email, transfer money).

| **CWE** | CWE-352 |
|---|---|
| **Needs** | Victim is logged in to the target site with a cookie; the action has no CSRF token; parameters are predictable |

## How it works

1. The victim is logged in to `bank.example`. Their browser holds the session cookie.
2. The victim visits the attacker's page, which contains a hidden auto-submitting form:

    ```html
    <form action="https://bank.example/transfer" method="POST" id="f">
      <input type="hidden" name="to" value="attacker-account">
      <input type="hidden" name="amount" value="5000">
    </form>
    <script>document.getElementById('f').submit()</script>
    ```

3. The browser sends the request to `bank.example` and **automatically attaches the session cookie**.
4. The bank sees a valid session and performs the transfer.

The attacker never sees the cookie or the response. They only trigger the action.

## CSRF vs XSS

| | CSRF | [XSS](xss.md) |
|---|---|---|
| Where the attacker's code runs | On the **attacker's** site | On the **target** site |
| What the attacker can do | Trigger actions blindly | Read pages, steal data, trigger actions |
| Main fix | CSRF tokens, SameSite cookies | Output encoding, CSP |

## What you see in logs

| Log source | What to look for |
|---|---|
| Web / app logs | State-changing requests (POST to `/transfer`, `/change-email`, `/settings`) with a **`Referer` / `Origin` from an external domain** |
| App logs | Requests missing the CSRF token or with an invalid one (rejected attempts) |
| User reports | "I didn't change my email / make this transfer" |

## How to respond

1. Identify affected users and actions; reverse unauthorised changes (email, password, payout).
2. Find the attacker page (Referer) and block / report it.
3. **Fix:** anti-CSRF tokens on every state-changing request, `SameSite=Lax` or `Strict` cookies (modern browsers default to Lax), check `Origin` header, re-authentication for sensitive actions, never use GET for state changes.

## References

- [OWASP: Cross Site Request Forgery](https://owasp.org/www-community/attacks/csrf)
- [OWASP: CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)

---

| [← Insecure deserialization](insecure-deserialization.md) | [Home](../README.md) | [Insecure direct object reference (IDOR) →](idor.md) |
|:---|:---:|---:|
