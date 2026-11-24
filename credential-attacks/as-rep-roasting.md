<sub>[Home](../README.md) › [Credential attacks](README.md)</sub>

# AS-REP roasting

> **In one line:** For accounts with **Kerberos pre-authentication disabled**, anyone can ask the DC for a TGT response and **crack the user's password** from it offline.

| **MITRE ATT&CK** | [T1558.004 AS-REP Roasting](https://attack.mitre.org/techniques/T1558/004/) |
|---|---|
| **Tactic** | Credential Access |
| **Needs** | A list of usernames. A valid account helps (to find vulnerable users via LDAP), but is not required |
| **Common tools** | Rubeus `asreproast`, Impacket `GetNPUsers.py`, Hashcat (mode 18200) |

## How it works

1. Normally, to get a TGT, the user must first prove they know the password (**pre-authentication**).
2. Some accounts have the setting **"Do not require Kerberos preauthentication"** (often for old apps).
3. For those accounts, the DC sends back an **AS-REP** without any proof, and part of it is **encrypted with the user's password hash**.
4. The attacker saves it and cracks it offline.

## What you see in logs

| Log source | What to look for |
|---|---|
| DC **4768** | **Pre-Authentication Type: 0** (no pre-auth) |
| DC 4768 | Ticket Encryption Type **`0x17`** (RC4) |
| DC 4738 | Account changed with **"Don't Require Preauth – Enabled"** (an attacker may set this on purpose to roast an account later) |
| LDAP / EDR | Query for accounts with `userAccountControl` flag `DONT_REQ_PREAUTH` (4194304) |

## How to detect it

- Any 4768 with pre-auth type 0. In most domains this should be **zero or near zero**: list the few legitimate accounts and alert on everything else.

**Example Splunk search**

```spl
index=wineventlog EventCode=4768 Pre_Authentication_Type=0
| stats count by Account_Name, Client_Address
```

## How to respond

1. **Enable pre-authentication** on the account.
2. **Reset its password** (assume it is being cracked).
3. Investigate the source IP of the request.
4. **Fix:** audit regularly. PowerShell: `Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true}`.

## Don't confuse with

- [Kerberoasting](kerberoasting.md): targets **service** accounts with SPNs (4769). AS-REP roasting targets accounts with **pre-auth disabled** (4768).

## References

- [MITRE ATT&CK T1558.004](https://attack.mitre.org/techniques/T1558/004/)

---

| [← Kerberoasting](kerberoasting.md) | [Home](../README.md) | [Golden and silver ticket →](golden-silver-ticket.md) |
|:---|:---:|---:|
