<sub>[Home](../README.md) › [Network attacks](README.md)</sub>

# Evil twin and rogue Wi-Fi

> **In one line:** The attacker sets up a **fake Wi-Fi access point** with the same name as a real one, gets devices to connect to it, and then sees or tampers with their traffic or captures their Wi-Fi credentials.

| **MITRE ATT&CK** | [T1557 Adversary-in-the-Middle](https://attack.mitre.org/techniques/T1557/); Mobile: [T1465 Rogue Wi-Fi Access Points](https://attack.mitre.org/techniques/T1465/) |
|---|---|
| **Tactics** | Credential Access, Collection |
| **Needs** | Physical proximity (parking lot, lobby, café, hotel, conference) |
| **Common tools** | WiFi Pineapple, hostapd / hostapd-wpe, airbase-ng, Wifiphisher, EAPHammer |

## Types

| Type | What it is |
|---|---|
| **Evil twin** | Fake AP using the **same SSID** as your real network (e.g. `Corp-WiFi`) |
| **Rogue AP** | Unauthorised AP plugged into your **wired** network (by an attacker or an employee), creating a backdoor into the LAN |
| **Karma / known-network attack** | AP answers to **any** network name a device asks for ("Home-WiFi", "Starbucks"), so devices auto-connect |
| **Captive portal phishing** | Fake "Wi-Fi login" page asking for corporate credentials |

## How it works

1. The attacker broadcasts `Corp-WiFi` with a **stronger signal** than the real AP.
2. Optionally sends **deauthentication** frames to kick devices off the real AP so they reconnect, often to the stronger fake one.
3. **On WPA2/WPA3-Enterprise:** the fake AP presents its own certificate. If devices **don't validate the RADIUS server certificate**, they send the user's challenge-response (MSCHAPv2), which can be cracked to get the **domain password**.
4. **On open / PSK networks:** victims get internet through the attacker, who can see unencrypted traffic, inject content, or show a fake login portal.

## What you see in logs

| Log source | What to look for |
|---|---|
| Wireless IDS / WIPS (controller) | **Same SSID** broadcast by an **unknown BSSID (MAC)**; AP on an unauthorised channel |
| Wireless controller | **Deauthentication floods**; many clients disconnecting at once |
| RADIUS / NPS logs | Authentication failures where clients reject or abort; users reporting certificate warnings |
| Endpoint / user reports | "Wi-Fi asked me to accept a new certificate" or "asked me to log in again" |
| Switch / NAC | Unknown device on a wired port with AP-like MAC vendor (rogue AP) |

## How to detect it

- Wireless intrusion prevention (WIPS) comparing broadcast SSIDs against your list of known AP MACs.
- Deauthentication frame counts.
- Periodic wireless surveys around your buildings.

## How to respond

1. Locate the device (WIPS signal triangulation, physical search; for rogue APs, the switch port).
2. Remove / shut down; involve physical security if it's in or near the building.
3. Reset passwords for users who connected and entered credentials.
4. **Fix:** WPA2/WPA3-Enterprise with **server certificate validation enforced** on all devices (via MDM/GPO), EAP-TLS (certificates instead of passwords), 802.11w protected management frames (blocks deauth), 802.1X / NAC on wired ports, always-on VPN on public Wi-Fi.

## References

- [MITRE ATT&CK T1557](https://attack.mitre.org/techniques/T1557/)

---

| [← Man-in-the-middle (MITM) and ARP spoofing](mitm-arp-spoofing.md) | [Home](../README.md) | [LLMNR / NBT-NS poisoning →](llmnr-nbtns-poisoning.md) |
|:---|:---:|---:|
