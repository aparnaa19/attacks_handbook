# Kerberoasting

!!! abstract "In one line"
    Any domain user requests **service tickets** for service accounts, takes them offline, and **cracks the service account's password** from the ticket.

| | |
|---|---|
| **MITRE ATT&CK** | [T1558.003 Kerberoasting](https://attack.mitre.org/techniques/T1558/003/) |
| **Tactic** | Credential Access |
| **Needs** | Any valid domain account (no admin rights needed) |
| **Common tools** | Rubeus `kerberoast`, Impacket `GetUserSPNs.py`, Hashcat (mode 13100) |

## How it works

1. Service accounts (e.g. `svc_sql`) have a **Service Principal Name (SPN)** so Kerberos knows which account runs a service.
2. Any domain user can ask the DC for a service ticket for any SPN. This is normal Kerberos.
3. The service ticket is **encrypted with the service account's password hash**.
4. The attacker requests tickets for all SPN accounts, saves them, and **cracks them offline** with Hashcat.
5. Service accounts often have **old, weak passwords that never expire** and are frequently admins. Cracking one can give domain admin.

Tickets encrypted with **RC4** (`0x17`) crack much faster than AES, so attackers request RC4 specifically.

## What you see in logs

| Log source | What to look for |
|---|---|
| DC **4769** | Ticket Encryption Type **`0x17`** (RC4) |
| DC 4769 | Service Name is a **user account** (not ending in `$`, not `krbtgt`) |
| DC 4769 | **Many** service tickets for **different** services from **one account** in a few minutes |
| LDAP logs / EDR | Query for `servicePrincipalName=*` (listing all SPN accounts) |
| Sysmon 1 / 4688 | `Rubeus.exe kerberoast`, `GetUserSPNs`, PowerShell `Invoke-Kerberoast` |

## How to detect it

- One user requesting tickets for many SPNs in a short time.
- RC4 tickets in an environment that normally uses AES.
- **Honeypot SPN:** create a fake service account with an SPN that nothing uses. Any 4769 for it is malicious.

```spl title="Example Splunk search"
index=wineventlog EventCode=4769 Ticket_Encryption_Type=0x17
  NOT Service_Name="*$" NOT Service_Name="krbtgt"
| bin _time span=10m
| stats dc(Service_Name) as services, values(Service_Name) by _time, Account_Name, Client_Address
| where services > 3
```

## How to respond

1. Identify the **requesting account and host**. That account is likely compromised; investigate it.
2. **Reset the passwords** of every service account whose ticket was requested. Assume they will be cracked.
3. Check whether those service accounts were used anywhere unusual after the event.
4. **Fix:** use **gMSA** (Group Managed Service Accounts, with automatic long random passwords), otherwise 25+ character passwords; enforce AES; remove unnecessary SPNs; least privilege for service accounts.

## Don't confuse with

- [AS-REP roasting](as-rep-roasting.md): cracks **user** accounts that have pre-auth disabled (event 4768, not 4769).

## References

- [MITRE ATT&CK T1558.003](https://attack.mitre.org/techniques/T1558/003/)
