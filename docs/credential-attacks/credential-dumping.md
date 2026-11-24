# Credential dumping

!!! abstract "In one line"
    The attacker, with admin rights on a machine, **pulls passwords and hashes out of Windows** (memory, registry or the AD database) to use on other machines.

| | |
|---|---|
| **MITRE ATT&CK** | [T1003 OS Credential Dumping](https://attack.mitre.org/techniques/T1003/) |
| **Tactic** | Credential Access |
| **Needs** | Local admin or SYSTEM on the machine |
| **Common tools** | Mimikatz, ProcDump, `comsvcs.dll`, Impacket `secretsdump.py`, NanoDump, Task Manager "Create dump file" |

## Where the credentials are

| Location | Sub-technique | What it holds | Typical command |
|---|---|---|---|
| **LSASS memory** | T1003.001 | Hashes (and sometimes passwords) of everyone **logged on** to this machine | `mimikatz sekurlsa::logonpasswords`, `procdump -ma lsass.exe`, `rundll32 comsvcs.dll, MiniDump <pid> out.dmp full` |
| **SAM** registry hive | T1003.002 | Hashes of **local** accounts (e.g. local Administrator) | `reg save HKLM\SAM sam.save` + `reg save HKLM\SYSTEM sys.save` |
| **NTDS.dit** on a DC | T1003.003 | Hashes of **every domain account** | `ntdsutil "ac i ntds" "ifm" "create full C:\temp"`, or copy from a Volume Shadow Copy |
| **LSA secrets** | T1003.004 | Service account passwords, saved auto-logon passwords | `reg save HKLM\SECURITY` |
| **Cached domain creds** | T1003.005 | Hashes of recent domain logons (for offline logon) | `mimikatz lsadump::cache` |
| **DCSync** | T1003.006 | Every domain hash, over the network | See [DCSync](dcsync.md) |

## How it works (LSASS example)

1. The attacker gets admin on a workstation where an IT admin recently logged in.
2. They run a tool that **opens the `lsass.exe` process** and reads its memory, or writes it to a `.dmp` file.
3. They extract the admin's NTLM hash (or Kerberos tickets).
4. They use it for [pass-the-hash](pass-the-hash.md) to move to servers.

## What you see in logs

| Log source | What to look for |
|---|---|
| **Sysmon 10** (process access) | `TargetImage = lsass.exe` from an unusual `SourceImage` (rundll32, powershell, procdump, unknown exe). `GrantedAccess` such as `0x1010`, `0x1410`, `0x1FFFFF` |
| Sysmon 1 / 4688 | Command lines with `sekurlsa`, `lsass`, `MiniDump`, `comsvcs.dll`, `procdump -ma`, `reg save HKLM\SAM`, `ntdsutil`, `vssadmin create shadow` |
| Sysmon 11 | `.dmp` file created, especially `lsass*.dmp` |
| EDR / Defender | Alerts like "LSASS memory access", "Mimikatz" signatures |
| DC | `ntdsutil` or shadow copy creation on a domain controller |

## How to detect it

- **Any non-system process reading LSASS memory.** Legitimate readers (AV/EDR, `svchost`, `MsMpEng`) should be allow-listed.
- **Registry hive exports** of SAM / SYSTEM / SECURITY.
- **Shadow copy + copy of `ntds.dit`** on a DC.

```spl title="Example Splunk search (Sysmon)"
index=sysmon EventCode=10 TargetImage="*\\lsass.exe"
  NOT SourceImage IN ("*\\MsMpEng.exe", "*\\svchost.exe", "*\\wininit.exe", "*\\csrss.exe")
| stats count by host, SourceImage, GrantedAccess
```

## How to respond

1. **Isolate the host.**
2. Find **which accounts were logged on** to that host (4624 events, `quser`). Assume all their credentials are stolen: reset them, starting with admins and service accounts.
3. Hunt for **use of those accounts** on other hosts ([lateral movement](../post-exploitation/lateral-movement.md)).
4. If **NTDS.dit** was taken: the whole domain is compromised. Reset **krbtgt twice** and all privileged accounts (see [Golden ticket](golden-silver-ticket.md)).
5. **Fix:** Credential Guard, LSA protection (RunAsPPL), LAPS for unique local admin passwords, don't log on to workstations with domain admin accounts.

## Don't confuse with

- [Infostealers](../malware/infostealers.md) steal **browser** passwords and cookies, and don't need admin.
- [DCSync](dcsync.md) gets the same data as NTDS.dit **without touching the DC's disk**.

## References

- [MITRE ATT&CK T1003](https://attack.mitre.org/techniques/T1003/)
