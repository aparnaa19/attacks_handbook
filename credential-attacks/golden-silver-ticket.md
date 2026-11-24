<sub>[Home](../README.md) › [Credential attacks](README.md)</sub>

# Golden and silver ticket

> **In one line:** With a stolen Kerberos key, the attacker **forges their own tickets**: a **golden ticket** (forged TGT) gives access to the whole domain; a **silver ticket** (forged service ticket) gives access to one service.

| | Golden ticket | Silver ticket |
|---|---|---|
| **MITRE ATT&CK** | [T1558.001](https://attack.mitre.org/techniques/T1558/001/) | [T1558.002](https://attack.mitre.org/techniques/T1558/002/) |
| **Key needed** | **krbtgt** account hash | Hash of a **service or computer account** |
| **What it forges** | TGT (step 2 of Kerberos) | Service ticket (step 4 of Kerberos) |
| **Access** | Any user, any group, any service in the domain | Only that one service on that one server |
| **Talks to the DC?** | Yes, to get service tickets | **No**. Goes straight to the service |
| **Tools** | Mimikatz `kerberos::golden`, Rubeus, Impacket `ticketer.py` | Same tools |

## How it works

**Golden ticket:**

1. The attacker already has domain admin and steals the **krbtgt** hash ([DCSync](dcsync.md) or NTDS.dit).
2. Every TGT in the domain is signed with that hash, so the attacker can **create a TGT** for any name, with any group memberships (e.g. Domain Admins), valid for as long as they choose (Mimikatz defaults to 10 years).
3. It keeps working even if the real admin passwords are reset, because only the **krbtgt** key matters.

**Silver ticket:**

1. The attacker steals the hash of a service account or a computer account (e.g. `FILESRV01$`).
2. They forge a service ticket for that service (e.g. CIFS on FILESRV01).
3. The service accepts it. The DC never sees anything, so DC logs show nothing.

## What you see in logs

| Log source | Golden ticket | Silver ticket |
|---|---|---|
| DC 4769 | Service ticket requests **with no earlier 4768** for that user/session (the TGT never came from the DC) | **Nothing** (DC not involved) |
| Target 4624 | Logon with an account name that **doesn't exist**, or odd domain field values | 4624 on the server with **no matching 4769** on any DC |
| Ticket details | Lifetime longer than domain policy (default max 10 hours) | Same |
| Defender for Identity | "Suspected Golden Ticket usage" alerts | |

## How to detect it

- Correlate DC 4768/4769 with server 4624: Kerberos logons that the DC never issued a ticket for.
- Tickets with lifetimes outside policy, RC4 encryption where AES is enforced.
- Golden tickets mean the attacker **already** owned the domain. Hunt for the [DCSync](dcsync.md) or NTDS theft that came first.

## How to respond

**Golden ticket:**

1. Treat as **full domain compromise**.
2. **Reset the krbtgt password twice**, waiting for AD replication between resets. The DC accepts both the current and the previous krbtgt key, so one reset is not enough.
3. Reset all privileged accounts, find and close every persistence the attacker added.

**Silver ticket:**

1. **Reset the password** of the abused service account, or the computer account (`Reset-ComputerMachinePassword`).
2. Find how its hash was stolen.

## References

- [MITRE ATT&CK T1558.001](https://attack.mitre.org/techniques/T1558/001/)
- [MITRE ATT&CK T1558.002](https://attack.mitre.org/techniques/T1558/002/)

---

| [← AS-REP roasting](as-rep-roasting.md) | [Home](../README.md) | [AD CS certificate abuse →](ad-cs-abuse.md) |
|:---|:---:|---:|
