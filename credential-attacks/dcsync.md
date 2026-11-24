<sub>[Home](../README.md) › [Credential attacks](README.md)</sub>

# DCSync

> **In one line:** The attacker **pretends to be a domain controller** and asks a real DC to send over everyone's password hashes using normal AD replication.

| **MITRE ATT&CK** | [T1003.006 DCSync](https://attack.mitre.org/techniques/T1003/006/) |
|---|---|
| **Tactic** | Credential Access |
| **Needs** | An account with **"Replicating Directory Changes"** and **"Replicating Directory Changes All"** rights (Domain Admins, Enterprise Admins, DC computer accounts, or any account given these rights) |
| **Common tools** | Mimikatz `lsadump::dcsync /user:krbtgt`, Impacket `secretsdump.py` |

## How it works

1. Domain controllers copy AD data to each other through a replication protocol (DRSUAPI, function `GetNCChanges`).
2. The attacker, using a privileged account, sends that same replication request **from their own machine**.
3. The DC checks the account has replication rights and answers with the requested password hashes.
4. The attacker usually asks for **krbtgt** (to forge [golden tickets](golden-silver-ticket.md)) and Domain Admin accounts.

No malware runs on the DC. It all happens over the network.

## What you see in logs

| Log source | What to look for |
|---|---|
| DC Security log **4662** | Operation on the domain object with these property GUIDs:<br>`1131f6aa-9c07-11d1-f79f-00c04fc2dcd2` (Replicating Directory Changes)<br>`1131f6ad-9c07-11d1-f79f-00c04fc2dcd2` (Replicating Directory Changes All)<br>`89e95b76-444d-4c62-991a-0facbeda640c` (Replicating Directory Changes In Filtered Set) |
| 4662 `SubjectUserName` | **Not** a DC computer account (DC accounts end with `$`) |
| Network (Zeek `dce_rpc.log`) | `DRSGetNCChanges` calls from an IP that is **not a DC** |
| Defender for Identity | "Suspected DCSync attack (replication of directory services)" |
| 5136 | Someone **added replication rights** to an account (a way to set this up as persistence) |

4662 requires "Audit Directory Service Access" enabled and auditing set on the domain object.

## How to detect it

- Replication requests where the source is not on your list of DC IPs.
- 4662 with the replication GUIDs where the account is a user, not a `DC$` account.

**Example Splunk search**

```spl
index=wineventlog EventCode=4662
  (Properties="*1131f6aa-9c07-11d1-f79f-00c04fc2dcd2*" OR Properties="*1131f6ad-9c07-11d1-f79f-00c04fc2dcd2*")
  NOT SubjectUserName="*$"
| stats count by SubjectUserName, host
```

## How to respond

This is a **domain compromise**.

1. Disable the account that did the replication and isolate the source machine.
2. **Reset krbtgt twice** (wait for replication between resets), then all Domain Admin, service and other privileged passwords.
3. Find **how** the attacker got a privileged account (work backwards: [credential dumping](credential-dumping.md), [Kerberoasting](kerberoasting.md)...).
4. Check who has replication rights on the domain object and remove anyone who shouldn't.

## References

- [MITRE ATT&CK T1003.006](https://attack.mitre.org/techniques/T1003/006/)

---

| [← Credential dumping](credential-dumping.md) | [Home](../README.md) | [Pass-the-hash →](pass-the-hash.md) |
|:---|:---:|---:|
