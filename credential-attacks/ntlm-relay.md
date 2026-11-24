<sub>[Home](../README.md) › [Credential attacks](README.md)</sub>

# NTLM relay

> **In one line:** The attacker tricks a machine into authenticating to them, then **forwards that authentication in real time** to another server and logs in there as the victim. No password or hash cracking needed.

| **MITRE ATT&CK** | [T1557.001 LLMNR/NBT-NS Poisoning and SMB Relay](https://attack.mitre.org/techniques/T1557/001/), [T1187 Forced Authentication](https://attack.mitre.org/techniques/T1187/) |
|---|---|
| **Tactics** | Credential Access, Lateral Movement |
| **Needs** | A way to make a victim authenticate (poisoning or coercion) and a target that doesn't require signing |
| **Common tools** | Impacket `ntlmrelayx.py`, Responder, PetitPotam, Coercer, PrinterBug |

## How it works

1. **Get an authentication to come to the attacker.** Either wait for one ([LLMNR poisoning](../network-attacks/llmnr-nbtns-poisoning.md)) or **force** one: coercion bugs like **PetitPotam** (MS-EFSRPC) or **PrinterBug** (MS-RPRN) make a server, even a **domain controller**, connect to the attacker and authenticate with its computer account.
2. The attacker does **not** crack anything. They pass each NTLM message straight through to a **target** server.
3. The target thinks the victim is logging in and grants access.
4. What the attacker gets depends on the target:

| Relay target | Result |
|---|---|
| SMB on a server without SMB signing | Run commands / dump SAM as the victim (if victim is admin there) |
| LDAP / LDAPS on a DC without signing / channel binding | Create computer accounts, change ACLs, set up resource-based constrained delegation (RBCD) to take over a machine |
| **AD CS web enrollment** (`/certsrv`) | Get a **certificate for a DC**, then become the DC → full domain compromise (ESC8). See [AD CS abuse](ad-cs-abuse.md) |
| MSSQL, HTTP apps | Log in as the victim |

## What you see in logs

| Log source | What to look for |
|---|---|
| Target server **4624** | Logon type 3, **NTLM**, where the `IpAddress` is the **attacker's machine** but the account belongs to someone else (or a DC's computer account `DC01$`) |
| DC 4624 / 4776 | A **computer account** (`DC01$`) authenticating from an IP that is **not** that computer |
| Network | SMB / HTTP connections **from a DC to a workstation** (coercion makes the DC connect out) |
| Network | EFSRPC / Print Spooler RPC calls to a DC from a workstation (coercion trigger) |
| AD CS | **4886 / 4887** certificate requested and issued for a DC account via web enrollment |
| LDAP / DC | New computer account created (4741), `msDS-AllowedToActOnBehalfOfOtherIdentity` changed (5136) |

## How to detect it

- **Account/IP mismatch:** a machine account logging in from an IP that isn't that machine.
- Domain controllers making outbound SMB/HTTP connections to workstations.
- Certificates issued to DC accounts through web enrollment.

**Example Splunk search: machine account used from the wrong IP**

```spl
index=wineventlog EventCode=4624 Authentication_Package=NTLM user="*$"
| lookup asset_ip_lookup host AS user OUTPUT ip AS expected_ip
| where src_ip != expected_ip
| table _time, dest, user, src_ip, expected_ip
```

(`asset_ip_lookup` is a placeholder for your asset inventory lookup.)

## How to respond

1. **Isolate** the attacker machine.
2. Check what the relayed sessions did on each target; revoke any certificates issued; remove computer accounts / delegation changes.
3. If a DC's identity was relayed: treat as **domain compromise**.
4. **Fix:** require **SMB signing** everywhere, **LDAP signing + channel binding**, **EPA** (Extended Protection) on AD CS web enrollment or disable it, disable Print Spooler on DCs, patch coercion bugs, disable LLMNR/NBT-NS, reduce NTLM use.

## Don't confuse with

- [LLMNR poisoning](../network-attacks/llmnr-nbtns-poisoning.md): **captures** the hash to crack offline. Relay **uses it live**. Often the same attack chain.
- [Pass-the-hash](pass-the-hash.md): attacker already **has** the hash.

## References

- [MITRE ATT&CK T1557.001](https://attack.mitre.org/techniques/T1557/001/)
- [MITRE ATT&CK T1187](https://attack.mitre.org/techniques/T1187/)

---

| [← Pass-the-ticket](pass-the-ticket.md) | [Home](../README.md) | [Kerberoasting →](kerberoasting.md) |
|:---|:---:|---:|
