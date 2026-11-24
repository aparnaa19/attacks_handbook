<sub>[Home](../README.md) › [Web attacks](README.md)</sub>

# Server-side request forgery (SSRF)

> **In one line:** The attacker makes the **server send a request to a URL they choose**, usually an internal system the attacker can't reach directly, like the cloud metadata service.

| **MITRE ATT&CK** | [T1190 Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/), [T1552.005 Cloud Instance Metadata API](https://attack.mitre.org/techniques/T1552/005/) |
|---|---|
| **OWASP** | A01:2025 Broken Access Control (SSRF was its own category in 2021) · CWE-918 |
| **Real example** | Capital One (2019): SSRF on a misconfigured WAF was used to fetch AWS credentials from the metadata service, then data from S3 |

## How it works

A feature lets users give a URL that the server fetches: "import image from URL", webhooks, PDF generators, link previews.

```text
https://app.com/fetch?url=https://example.com/cat.png
```

The attacker changes it to:

```text
https://app.com/fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

`169.254.169.254` is the **cloud metadata service**. It is only reachable from inside the cloud server, and it hands out the server's **temporary cloud credentials**. The server fetches it and returns the credentials to the attacker.

## Common SSRF targets

| Target | Why |
|---|---|
| `http://169.254.169.254/` | AWS / Azure / GCP metadata: cloud credentials, instance info |
| `http://localhost:8080/admin`, `http://127.0.0.1/` | Admin panels only reachable locally |
| `http://10.x.x.x`, `192.168.x.x` | Internal services, internal port scanning |
| `file:///etc/passwd` | Local files (if the library supports `file://`) |

**Bypasses** for blocklists: `http://2130706433/` (decimal for 127.0.0.1), `http://0x7f000001/`, `http://[::1]/`, a domain that resolves to an internal IP, or a redirect from an attacker URL to an internal one.

## What you see in logs

| Log source | What to look for |
|---|---|
| Web access log | Parameters like `url=`, `uri=`, `dest=`, `redirect=`, `image=`, `webhook=` containing `169.254.169.254`, `localhost`, `127.0.0.1`, private IPs, `file://` |
| Server network / VPC flow logs | The **application** connecting to `169.254.169.254` or to internal hosts it never talks to |
| AWS CloudTrail | API calls made with the **instance role's credentials from an IP outside AWS** |
| AWS GuardDuty | `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.OutsideAWS` |

## How to detect it

- Internal IPs or metadata addresses in user-supplied URL parameters.
- Cloud credentials belonging to a server being used from somewhere else.

## How to respond

1. Check whether the metadata endpoint or internal systems actually **returned data** (status, response size).
2. If cloud credentials may have leaked: **revoke the role's active sessions**, and review CloudTrail for what those credentials did (S3 reads, new users, new keys).
3. Block the request pattern at the WAF; fix the feature.
4. **Fix:** allowlist the domains the server may fetch, block internal and metadata IPs at the network level, **IMDSv2** on AWS (requires a token that a simple SSRF can't obtain), least privilege for the instance role.

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [MITRE ATT&CK T1552.005](https://attack.mitre.org/techniques/T1552/005/)

---

| [← Path traversal and file inclusion (LFI / RFI)](path-traversal.md) | [Home](../README.md) | [XML external entity (XXE) →](xxe.md) |
|:---|:---:|---:|
