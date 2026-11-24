<sub>[Home](../README.md) › [Network attacks](README.md)</sub>

# LLMNR / NBT-NS poisoning

> **In one line:** When a Windows machine can't resolve a name through DNS, it **asks the whole local network**. The attacker answers "that's me", and the victim sends them its **NTLM password hash**.

| **MITRE ATT&CK** | [T1557.001 LLMNR/NBT-NS Poisoning and SMB Relay](https://attack.mitre.org/techniques/T1557/001/) |
|---|---|
| **Tactics** | Credential Access, Collection |
| **Needs** | Attacker on the same network segment |
| **Common tools** | Responder, Inveigh, Impacket `ntlmrelayx.py` |

## How it works

1. A user types a wrong share name: `\\filesrvr` instead of `\\fileserver`.
2. DNS says "no such name".
3. Windows falls back to **LLMNR** (UDP 5355, multicast) and **NBT-NS** (UDP 137, broadcast): "Does anyone know `filesrvr`?"
4. The attacker's **Responder** replies: "Yes, it's me."
5. The victim connects to the attacker and **automatically authenticates** with NTLM, sending a **NetNTLMv2** challenge-response.
6. The attacker either:
    - **cracks it offline** (Hashcat mode 5600) to get the password, or
    - **relays it** in real time to another server (`ntlmrelayx`) to log in as the victim. Works where **SMB signing** is not required.

Users don't even need to type a wrong name: misconfigured apps, old drive mappings and WPAD lookups trigger it constantly.

## What you see in logs

| Log source | What to look for |
|---|---|
| Network (Zeek, IDS, packet capture) | LLMNR (UDP 5355) / NBT-NS (UDP 137) **responses coming from a workstation** |
| Network | One host answering name queries for **many different names** |
| Windows 4624 / 4776 | NTLM authentications **to a workstation IP** that is not a server |
| Relay | 4624 logon type 3 on a server from the attacker's IP using the victim's account |
| EDR | Responder / Inveigh detection |

## How to detect it

- **Honeypot query:** schedule a script that looks up a name that doesn't exist (e.g. `\\fileshare-backup01`). Nobody legitimate should answer. If someone does, it's a poisoner.
- Alert on LLMNR/NBT-NS responses from non-server hosts.

## How to respond

1. Find the responding host and **isolate** it.
2. Identify users whose hashes were sent to it (NTLM auth logs, network traffic) and **reset their passwords**, especially privileged ones.
3. If relaying happened, check what the relayed sessions did on the target servers.
4. **Fix:** **disable LLMNR** (Group Policy: *Turn off multicast name resolution*), **disable NBT-NS** (NetBIOS over TCP/IP off via DHCP or adapter settings), **require SMB signing** (stops relay), strong passwords (makes cracking slow).

## References

- [MITRE ATT&CK T1557.001](https://attack.mitre.org/techniques/T1557/001/)

---

| [← MITM and ARP spoofing](mitm-arp-spoofing.md) | [Home](../README.md) | [DNS tunneling →](dns-tunneling.md) |
|:---|:---:|---:|
