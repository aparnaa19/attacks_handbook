<sub>[Home](../README.md) › [Network attacks](README.md)</sub>

# Man-in-the-middle (MITM) and ARP spoofing

> **In one line:** The attacker places themselves **between the victim and the network** so all traffic passes through them, letting them read or change it.

| **MITRE ATT&CK** | [T1557 Adversary-in-the-Middle](https://attack.mitre.org/techniques/T1557/): .002 ARP Cache Poisoning, .003 DHCP Spoofing |
|---|---|
| **Tactics** | Credential Access, Collection |
| **Needs** | Attacker on the **same local network** (same LAN / Wi-Fi) |
| **Common tools** | Bettercap, Ettercap, arpspoof |

## ARP in one paragraph

On a local network, devices send frames to **MAC addresses**, not IPs. **ARP** (Address Resolution Protocol) asks "Who has 192.168.1.1?" and the owner replies "192.168.1.1 is at `aa:bb:cc:dd:ee:ff`". Devices **cache** the answer. ARP has **no authentication**: any device can send a reply, even one nobody asked for, and the cache accepts it.

## How ARP spoofing works

1. The attacker sends fake ARP replies to the victim: "The gateway (192.168.1.1) is at **attacker's MAC**".
2. And to the gateway: "The victim (192.168.1.50) is at **attacker's MAC**".
3. Both update their caches. All traffic between victim and gateway now goes **through the attacker**.
4. The attacker forwards it (so nothing seems broken) while reading or changing it: clear-text passwords (HTTP, FTP, Telnet), session cookies, DNS replies.

## Other MITM methods

| Method | How |
|---|---|
| **Rogue DHCP** | Attacker's DHCP server answers first and gives victims the attacker as their gateway/DNS |
| **Evil twin Wi-Fi** | Fake access point with the same name as the real one |
| **DNS spoofing** | Fake DNS answers send victims to attacker-controlled IPs |
| **SSL stripping** | Downgrades HTTPS links to HTTP so traffic is readable (stopped by HSTS) |
| **AiTM phishing** | Web proxy between user and real login page. See [Session token theft](../cloud-identity/session-token-theft.md) |

## What you see in logs

| Log source | What to look for |
|---|---|
| Switch (Dynamic ARP Inspection) | DAI drops: ARP replies that don't match the DHCP snooping table |
| arpwatch / Zeek | "Flip flop": an IP address changes its MAC address; **one MAC claims several IPs** (including the gateway) |
| Endpoints | Windows "IP address conflict" warnings; `arp -a` shows the gateway with another host's MAC |
| IDS | Excessive gratuitous ARP replies |
| Network | Many unsolicited ARP replies from one MAC |

## How to detect it

- Gateway's MAC address changes.
- One MAC address associated with multiple IPs.
- Many ARP replies without requests.

## How to respond

1. Find the attacker's **MAC address**, then look it up in the switch **MAC address table** to find the physical port; shut the port.
2. Identify what traffic passed through the attacker and **reset any credentials sent in clear text**.
3. Clear ARP caches on affected hosts.
4. **Fix:** Dynamic ARP Inspection + DHCP snooping on switches, port security / 802.1X, encrypt everything (HTTPS with HSTS, SMB signing, LDAPS), WPA3-Enterprise on Wi-Fi.

## References

- [MITRE ATT&CK T1557.002](https://attack.mitre.org/techniques/T1557/002/)

---

| [← DoS and DDoS](dos-ddos.md) | [Home](../README.md) | [LLMNR / NBT-NS poisoning →](llmnr-nbtns-poisoning.md) |
|:---|:---:|---:|
