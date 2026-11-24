# Pass-the-hash

!!! abstract "In one line"
    The attacker logs in with a stolen **NTLM password hash** instead of the password. No cracking needed.

| | |
|---|---|
| **MITRE ATT&CK** | [T1550.002 Pass the Hash](https://attack.mitre.org/techniques/T1550/002/) |
| **Tactics** | Defense Evasion, Lateral Movement |
| **Needs** | An NTLM hash (from [credential dumping](credential-dumping.md)) and a target that accepts NTLM |
| **Common tools** | Mimikatz `sekurlsa::pth`, Impacket `psexec.py` / `wmiexec.py` with `-hashes`, NetExec `-H` |

## How it works

1. NTLM login proves identity by encrypting a challenge with the user's **NT hash**, not with the password (see [NTLM in 3 steps](index.md#ntlm-in-3-steps)).
2. So anyone holding the hash can answer the challenge.
3. The attacker dumps a hash on one machine, for example the **local Administrator** hash.
4. If other machines have the **same local admin password** (very common without LAPS), the same hash logs in everywhere.
5. The attacker runs commands on those machines with PsExec, WMI or SMB.

## What you see in logs

| Where | Log | What to look for |
|---|---|---|
| **Target** machine | 4624 | Logon type **3**, Authentication Package **NTLM**, from a workstation that does not normally connect there |
| Target | 4672 | Admin-level logon right after |
| Target | 7045 / 5140 | Service created / `ADMIN$` share accessed (PsExec-style execution) |
| **Source** machine (Mimikatz `pth`) | 4624 | Logon type **9** (NewCredentials) with Logon Process **`seclogo`** |
| Source | Sysmon 10 | Access to `lsass.exe` (the tool writes the hash into LSASS) |
| Domain controller | 4776 | NTLM validation for a domain account from an unusual workstation |

## How to detect it

- **One account doing NTLM type 3 logons to many hosts** in a short time.
- **Local administrator account** logging on over the network (should be rare in a well-managed domain).
- Admin accounts using **NTLM** where they normally use Kerberos.
- Logon type 9 + `seclogo` on workstations.

```spl title="Example Splunk search: one account, NTLM, many hosts"
index=wineventlog EventCode=4624 Logon_Type=3 Authentication_Package=NTLM
  NOT user="ANONYMOUS LOGON" NOT user="*$"
| stats dc(dest) as hosts, values(dest) by user, src_ip
| where hosts > 5
```

## How to respond

1. **Isolate the source host**. It is where the attacker works from.
2. **Reset the password** of the abused account. A new password gives a new hash, so the stolen one stops working.
3. Find where the hash was stolen ([credential dumping](credential-dumping.md)) and what was run on each target.
4. **Fix:** **LAPS** (unique local admin password per machine), block network logon for local accounts, put admins in the **Protected Users** group, reduce NTLM use, tiered admin model.

## Don't confuse with

- [Pass-the-ticket](pass-the-ticket.md): reuses a **Kerberos ticket**, not an NTLM hash.
- **Overpass-the-hash**: uses the hash to request a **Kerberos TGT**, then continues with Kerberos. Shows up as 4768 with RC4 encryption (`0x17`) for an account that normally uses AES.

## References

- [MITRE ATT&CK T1550.002](https://attack.mitre.org/techniques/T1550/002/)
