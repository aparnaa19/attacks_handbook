<sub>[Home](../README.md) › [Web attacks](README.md)</sub>

# Insecure direct object reference (IDOR)

> **In one line:** The attacker changes an ID in a request (`invoice=1001` → `1002`) and gets **someone else's data**, because the server never checks that the object belongs to them.

| **OWASP** | A01:2025 Broken Access Control · CWE-639 |
|---|---|
| **Also called** | BOLA (Broken Object Level Authorization), #1 in the OWASP API Security Top 10 |

## How it works

1. User Bob logs in and views his invoice:

    ```text
    GET /api/invoices/1001
    ```

2. The server checks that Bob is **logged in**, but not that invoice 1001 is **Bob's**.
3. Bob (or a script) requests `/api/invoices/1002`, `1003`, `1004`... and downloads every customer's invoice.

The ID can be anywhere: URL path, query string, POST body, JSON field, hidden form field, cookie. Using random IDs (GUIDs) makes guessing harder but is **not** a fix; the ownership check is.

## Typical IDOR targets

| Request | Impact |
|---|---|
| `GET /api/users/57/profile` | Read other users' personal data |
| `PUT /api/users/57/email` | Change another user's email → take over the account |
| `GET /download?doc_id=8812` | Download other customers' documents |
| `DELETE /api/orders/4471` | Delete other users' orders |
| `"role": "admin"` in a profile update body | Mass assignment, a related bug: user sets fields they shouldn't |

## What you see in logs

| Log source | What to look for |
|---|---|
| Web / API logs | **One session or user accessing many sequential IDs** in a short time |
| Web / API logs | Many 200s for objects, mixed with 403/404 as the script hits missing or protected IDs |
| App logs | User ID in the session ≠ owner of the object accessed (if the app logs both) |
| Data volume | Unusually large total download by one account |

## How to detect it

**Example Splunk search: one user enumerating object IDs**

```spl
index=api uri_path="/api/invoices/*"
| rex field=uri_path "/api/invoices/(?<invoice_id>\d+)"
| bin _time span=10m
| stats dc(invoice_id) as ids, count by _time, user, src_ip
| where ids > 50
```

## How to respond

1. Confirm the bug by checking whether responses for other users' objects returned data (200 + normal size).
2. Determine **which records** were accessed. Personal data exposure may have legal reporting requirements.
3. Block / disable the account or token doing the enumeration; rate-limit the endpoint.
4. **Fix:** server-side **authorization check on every object access** ("does this object belong to this user?"), deny by default, log object-level access.

## References

- [OWASP: Insecure Direct Object Reference Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html)
- [OWASP API Security Top 10: API1 Broken Object Level Authorization](https://owasp.org/API-Security/)

---

| [← CSRF](csrf.md) | [Home](../README.md) | [File upload and web shells →](file-upload-web-shell.md) |
|:---|:---:|---:|
