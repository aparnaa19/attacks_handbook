<sub>[Home](../README.md) › [Credential attacks](README.md)</sub>

# AD CS certificate abuse

> **In one line:** The attacker abuses a misconfigured **Active Directory Certificate Services** template to get a certificate **in a Domain Admin's name**, then uses it to log in as that admin.

| **MITRE ATT&CK** | [T1649 Steal or Forge Authentication Certificates](https://attack.mitre.org/techniques/T1649/) |
|---|---|
| **Tactics** | Credential Access, Privilege Escalation, Persistence |
| **Needs** | Any domain user + a vulnerable certificate template (ESC1) or web enrollment open to relay (ESC8) |
| **Common tools** | Certipy, Certify, Rubeus, PKINITtools |

## Background in 3 lines

- AD CS is Microsoft's internal certificate authority. Users and computers get certificates from **templates**.
- A certificate with the **Client Authentication** purpose can be used to **log in** with Kerberos (called **PKINIT**) instead of a password.
- Certificates stay valid for a long time (often a year) and **survive password resets**.

## How it works (ESC1, the classic)

1. The attacker, as a normal user, lists templates: `certipy find -vulnerable`.
2. They find a template where:
    - normal users can **enroll**,
    - it allows **Client Authentication**,
    - and the **requester can type any name** into the certificate (`ENROLLEE_SUPPLIES_SUBJECT`).
3. They request a certificate with the name `administrator@corp.local`.
4. The CA issues it. The attacker uses it to get a **TGT as Administrator** (`certipy auth` / `Rubeus asktgt /certificate:`).
5. They now have domain admin, and the certificate keeps working after the admin's password is changed.

## Other common paths

| Name | Problem |
|---|---|
| **ESC1** | Template lets the requester choose the subject name (above) |
| **ESC4** | User can **edit** a template and make it vulnerable |
| **ESC6** | CA flag `EDITF_ATTRIBUTESUBJECTALTNAME2` lets any request specify a name |
| **ESC8** | Web enrollment (`/certsrv`) accepts NTLM → [NTLM relay](ntlm-relay.md) a DC to get a DC certificate |

## What you see in logs

| Log source | What to look for |
|---|---|
| CA server **4886** | Certificate Services **received** a request (needs CA auditing enabled) |
| CA server **4887** | Certificate **issued**. Check the **Requester** vs the **Subject / SAN**: requester `bob` but subject `administrator` = ESC1 |
| CA server 4899 / 4900 | Template updated / CA security settings changed (ESC4 / ESC7) |
| DC **4768** | TGT request with **Certificate Issuer Name / Serial Number** fields filled = certificate (PKINIT) logon. Unusual for admin accounts that normally use passwords |
| Sysmon 1 | `certipy`, `Certify.exe`, `Rubeus asktgt /certificate` |
| LDAP | Queries enumerating `pKICertificateTemplate` objects |

## How to detect it

- 4887 where the **requester** and the **subject/SAN** are different accounts.
- Privileged accounts suddenly authenticating with **certificates** (4768 with certificate fields).
- Any change to certificate templates or CA settings.

**Example Splunk search: certificate logons for admins**

```spl
index=wineventlog EventCode=4768 Certificate_Issuer_Name=*
  Account_Name IN ("administrator", "*adm*", "*admin*")
| table _time, Account_Name, Client_Address, Certificate_Issuer_Name, Certificate_Serial_Number
```

## How to respond

1. **Revoke** the issued certificate(s) on the CA **and** publish the CRL. Resetting the password is **not** enough.
2. Investigate everything done with the impersonated account.
3. Fix the template (remove "supply in request", restrict enrollment, require manager approval).
4. **Fix:** audit AD CS regularly (Certipy / PSPKIAudit / Locksmith), enable CA auditing (4886/4887), disable web enrollment or enforce EPA + HTTPS, remove `EDITF_ATTRIBUTESUBJECTALTNAME2`.

## References

- [MITRE ATT&CK T1649](https://attack.mitre.org/techniques/T1649/)
- [Certipy (AD CS enumeration and abuse tool, with ESC explanations)](https://github.com/ly4k/Certipy)

---

| [← Golden and silver ticket](golden-silver-ticket.md) | [Home](../README.md) | [Malware →](../malware/README.md) |
|:---|:---:|---:|
