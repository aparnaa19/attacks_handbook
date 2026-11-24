# Windows event IDs

The Security log IDs a SOC analyst sees every day. Most need the right **audit policy** enabled to appear.

## Logons and accounts

| ID | Meaning | Used to detect |
|---|---|---|
| **4624** | Successful logon | Who logged in, from where, how (see logon types below) |
| **4625** | Failed logon | [Brute force](../credential-attacks/brute-force.md), [password spraying](../credential-attacks/password-spraying.md) |
| **4634 / 4647** | Logoff | Session length |
| **4648** | Logon with explicit credentials (`runas`, or a tool passing creds) | [Lateral movement](../post-exploitation/lateral-movement.md) |
| **4672** | Special privileges assigned to new logon (admin-level logon) | Privileged account use |
| **4720** | User account created | [Persistence](../post-exploitation/persistence.md) |
| **4722 / 4725 / 4726** | Account enabled / disabled / deleted | Account tampering |
| **4724** | Attempt to reset a password | Takeover of another account |
| **4728 / 4732 / 4756** | Member added to a global / local / universal security group | Someone added to Domain Admins or local Administrators |
| **4738** | User account changed | e.g. "password never expires" set, pre-auth disabled |
| **4740** | Account locked out | Brute force |
| **4767** | Account unlocked | |
| **4778 / 4779** | RDP session reconnected / disconnected | RDP activity |

## Logon types (on 4624 / 4625)

| Type | Name | Meaning |
|---|---|---|
| **2** | Interactive | At the keyboard (or console) |
| **3** | Network | Accessing a share, SMB, many remote tools. **Most common for lateral movement** |
| **4** | Batch | Scheduled task |
| **5** | Service | A service started under an account |
| **7** | Unlock | Workstation unlocked |
| **8** | NetworkCleartext | Password sent in clear text (e.g. IIS basic auth) |
| **9** | NewCredentials | `runas /netonly`. Also seen with some pass-the-hash tools |
| **10** | RemoteInteractive | **RDP** |
| **11** | CachedInteractive | Logged on with cached credentials (no DC reachable) |

## 4625 failure codes (Status / SubStatus)

| Code | Meaning | What it tells you |
|---|---|---|
| `0xC000006A` | Wrong password | The username **exists** |
| `0xC0000064` | User name does not exist | Attacker is guessing usernames |
| `0xC0000234` | Account locked out | Brute force hit the lockout limit |
| `0xC0000072` | Account disabled | Someone is using an old account |
| `0xC000006F` | Outside allowed logon hours | |
| `0xC0000071` | Password expired | |
| `0xC0000193` | Account expired | |

## Kerberos and NTLM (on domain controllers)

| ID | Meaning | Used to detect |
|---|---|---|
| **4768** | Kerberos TGT requested | [AS-REP roasting](../credential-attacks/as-rep-roasting.md) (pre-auth type 0) |
| **4769** | Kerberos service ticket requested | [Kerberoasting](../credential-attacks/kerberoasting.md) (encryption type `0x17`) |
| **4771** | Kerberos pre-authentication failed | Password spraying via Kerberos (failure code `0x18` = bad password) |
| **4776** | DC validated NTLM credentials | NTLM brute force, [pass-the-hash](../credential-attacks/pass-the-hash.md) |

**Kerberos encryption types:** `0x17` = RC4 (weak, attacker favourite), `0x11` = AES128, `0x12` = AES256.

**Common 4768 result codes:** `0x6` = username does not exist, `0x12` = account disabled / locked / expired, `0x17` = password expired. (Wrong passwords show up as **4771** with code `0x18`.)

## Processes, services, tasks

| ID | Log | Meaning |
|---|---|---|
| **4688** | Security | New process created. Enable "include command line" to see arguments |
| **4697** | Security | Service installed |
| **7045** | System | New service installed. Classic sign of PsExec and many malware droppers |
| **4698 / 4702** | Security | Scheduled task created / updated |
| **4657** | Security | Registry value modified (needs registry auditing / SACL) |

## Objects, shares and AD

| ID | Meaning | Used to detect |
|---|---|---|
| **4662** | Operation performed on an AD object | [DCSync](../credential-attacks/dcsync.md) (replication rights GUIDs) |
| **4663** | Attempt to access an object (file, registry) | Access to sensitive files |
| **5140** | Network share accessed | Access to `ADMIN$`, `C$` = [lateral movement](../post-exploitation/lateral-movement.md) |
| **5145** | Detailed share access check | Which file on a share was touched |

## Log tampering and policy

| ID | Log | Meaning |
|---|---|---|
| **1102** | Security | **Security log cleared**. Almost always investigate |
| **104** | System | System log cleared |
| **4719** | Security | Audit policy changed |

## PowerShell

| ID | Log | Meaning |
|---|---|---|
| **4104** | Microsoft-Windows-PowerShell/Operational | **Script block logging**: the actual code that ran, even after decoding. Best PowerShell source |
| **4103** | Microsoft-Windows-PowerShell/Operational | Module logging (pipeline execution details) |
| **400 / 600** | Windows PowerShell (classic) | Engine started. `EngineVersion=2.0` = downgrade attack to avoid logging |

## RDP-specific logs

| ID | Log | Meaning |
|---|---|---|
| **1149** | TerminalServices-RemoteConnectionManager/Operational | RDP authentication succeeded (shows source IP) |
| **21 / 24 / 25** | TerminalServices-LocalSessionManager/Operational | RDP session logon / disconnect / reconnect |

## Microsoft Defender

| ID | Meaning |
|---|---|
| **1116** | Malware detected |
| **1117** | Action taken on malware |
| **5001** | Real-time protection **disabled** |
