<sub>[Home](../README.md) › [Credential attacks](README.md)</sub>

# Pass-the-ticket

> **In one line:** The attacker **steals a Kerberos ticket from memory** on one machine and loads it into their own session to act as that user.

| **MITRE ATT&CK** | [T1550.003 Pass the Ticket](https://attack.mitre.org/techniques/T1550/003/) |
|---|---|
| **Tactics** | Defense Evasion, Lateral Movement |
| **Needs** | Admin on a machine where the victim has a Kerberos session |
| **Common tools** | Mimikatz `sekurlsa::tickets /export` + `kerberos::ptt`, Rubeus `dump` + `ptt` |

## How it works

1. A user (ideally an admin) is logged on to a machine. Their **TGT** and service tickets sit in LSASS memory.
2. The attacker, with admin on that machine, **exports the tickets** (`.kirbi` files).
3. On another machine they **inject the ticket** into their own logon session.
4. Windows now uses the stolen ticket automatically. The attacker accesses file shares, servers, etc. as the victim.
5. No password and no hash needed. The ticket works until it expires (default TGT lifetime: **10 hours**, renewable up to **7 days**).

## What you see in logs

| Log source | What to look for |
|---|---|
| DC **4768** | TGT issued to the user from **host A's IP** |
| DC **4769** | Service tickets requested by the same user from **host B's IP**, where no TGT was issued to them |
| Target server 4624 | Logon type 3, Kerberos, from a host where the user is not logged on |
| Sysmon 10 | Access to `lsass.exe` on the machine the ticket was stolen from |
| Sysmon 1 / 4688 | `Rubeus.exe`, `kerberos::ptt`, `.kirbi` files on disk |

## How to detect it

- **Same account, tickets used from two different hosts** at the same time.
- Service ticket requests (4769) from an IP that never got a TGT (4768) for that user.
- EDR alerts on ticket export / injection.

## How to respond

1. **Isolate** both the host the ticket was stolen from and the host using it.
2. **Disable the account and reset its password.** Already-issued tickets can keep working until they expire, so monitor the account for at least the TGT lifetime.
3. Run `klist purge` on affected hosts to clear cached tickets.
4. If a **krbtgt-signed** ticket looks forged rather than stolen, see [Golden ticket](golden-silver-ticket.md).
5. **Fix:** limit where privileged accounts log on (they should not log on to workstations), Credential Guard, Protected Users group.

## Don't confuse with

- [Pass-the-hash](pass-the-hash.md): reuses an **NTLM hash**.
- [Golden ticket](golden-silver-ticket.md): the ticket is **forged**, not stolen.

## References

- [MITRE ATT&CK T1550.003](https://attack.mitre.org/techniques/T1550/003/)

---

| [← Pass-the-hash](pass-the-hash.md) | [Home](../README.md) | [NTLM relay →](ntlm-relay.md) |
|:---|:---:|---:|
