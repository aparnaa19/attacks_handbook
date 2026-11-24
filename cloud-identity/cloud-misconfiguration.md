<sub>[Home](../README.md) › [Cloud and identity](README.md)</sub>

# Cloud misconfiguration and key abuse

> **In one line:** Many cloud breaches need no exploit: data is **left public**, **access keys are leaked**, or **permissions are far too wide**. Attackers scan for these constantly.

| **MITRE ATT&CK** | [T1530 Data from Cloud Storage](https://attack.mitre.org/techniques/T1530/), [T1552.001 Credentials In Files](https://attack.mitre.org/techniques/T1552/001/), [T1078.004 Valid Accounts: Cloud Accounts](https://attack.mitre.org/techniques/T1078/004/), [T1562.008 Disable or Modify Cloud Logs](https://attack.mitre.org/techniques/T1562/008/), [T1496 Resource Hijacking](https://attack.mitre.org/techniques/T1496/) |
|---|---|
| **Typical outcomes** | Data leak, crypto-mining bills, ransom of cloud data |

## Common mistakes

| Mistake | What happens |
|---|---|
| **Public storage** (S3 bucket, Azure Blob container, GCS bucket) | Anyone on the internet can list and download files |
| **Access keys in code** | Keys pushed to a public GitHub repo; bots find them within minutes |
| **Over-permissive IAM** | A role with `"Action": "*", "Resource": "*"` given to an app or user |
| **Open security groups** | SSH (22), RDP (3389) or databases open to `0.0.0.0/0` |
| **Exposed databases** | Elasticsearch, MongoDB, Redis on the internet with no password |
| **No MFA on root / admin** | One phished password = full account |
| **Logging off** | CloudTrail not enabled in all regions, so nothing is recorded |

## How a leaked-key attack works (AWS example)

1. Attacker finds an access key in a public repo.
2. **"Who am I?"** `aws sts get-caller-identity`.
3. **Discovery:** `ListBuckets`, `DescribeInstances`, `ListUsers`, `GetAccountAuthorizationDetails`.
4. **Persistence:** `CreateUser`, `CreateAccessKey`, `AttachUserPolicy` (AdministratorAccess), `CreateLoginProfile`.
5. **Evasion:** `StopLogging` / `DeleteTrail` on CloudTrail, `DeleteDetector` (GuardDuty).
6. **Impact:** `GetObject` on thousands of files (data theft), `RunInstances` with large GPU instances in **regions you never use** (crypto-mining), or deleting data and leaving a ransom note.

## What you see in logs

| Log source | What to look for |
|---|---|
| **CloudTrail** | `GetCallerIdentity` from a new IP or user-agent; burst of `List*` / `Describe*` calls |
| CloudTrail | `CreateUser`, `CreateAccessKey`, `AttachUserPolicy`, `PutUserPolicy`, `CreateLoginProfile` |
| CloudTrail | `StopLogging`, `DeleteTrail`, `UpdateTrail`, `DeleteDetector`, `PutBucketPolicy` / `PutBucketAcl` / `PutPublicAccessBlock` (making things public) |
| CloudTrail | `RunInstances` in unused regions or of unusual instance types |
| CloudTrail | `ConsoleLogin` without MFA, or root account use |
| S3 data events / server access logs | Large numbers of `GetObject` from one identity or IP |
| GuardDuty | Findings like credential use from unusual locations, crypto-mining, CloudTrail disabled |
| Azure Activity Log / Entra | Equivalent operations: new role assignments, storage made public, new app secrets |

## How to detect it

**Example Splunk search (AWS CloudTrail): high-risk API calls**

```spl
index=aws sourcetype=aws:cloudtrail
  eventName IN ("CreateUser", "CreateAccessKey", "AttachUserPolicy", "StopLogging", "DeleteTrail", "PutBucketPolicy", "PutBucketAcl")
| table _time, userIdentity.arn, sourceIPAddress, awsRegion, eventName, errorCode
```

- API activity from an identity at an IP / location it has never used.
- Activity in regions your company doesn't operate in.
- Any change to logging.

## How to respond

1. **Deactivate the leaked key** (don't just delete: deactivating preserves evidence) and rotate it.
2. Remove everything the attacker created: users, keys, roles, policies, instances, Lambda functions.
3. Review CloudTrail for **all** actions by the key and anything it created, across **all regions**.
4. Determine data accessed (S3 data events), make public resources private again.
5. Re-enable logging and detection services.
6. **Fix:** Cloud Security Posture Management (CSPM), account-level **Block Public Access**, secret scanning (e.g. GitHub push protection), short-lived credentials (roles / SSO instead of long-term keys), least privilege, MFA on root, organisation policies (SCPs) that block disabling CloudTrail.

## References

- [MITRE ATT&CK T1530](https://attack.mitre.org/techniques/T1530/)
- [MITRE ATT&CK Cloud Matrix](https://attack.mitre.org/matrices/enterprise/cloud/)

---

| [← OAuth consent phishing](oauth-consent-phishing.md) | [Home](../README.md) | [Insider threat →](../insider-threat/README.md) |
|:---|:---:|---:|
