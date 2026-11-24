# Reconnaissance and scanning

!!! abstract "In one line"
    Before attacking, the attacker **collects information**: which hosts are alive, which ports are open, which software and versions are running, who works there.

| | |
|---|---|
| **MITRE ATT&CK** | [T1595 Active Scanning](https://attack.mitre.org/techniques/T1595/) (from outside), [T1046 Network Service Discovery](https://attack.mitre.org/techniques/T1046/) (from inside), [T1589 Gather Victim Identity Information](https://attack.mitre.org/techniques/T1589/) |
| **Tactics** | Reconnaissance, Discovery |
| **Common tools** | Nmap, Masscan, Shodan, Nuclei, Nessus, Gobuster / ffuf, BloodHound (AD) |

## Passive vs active

| | Passive recon | Active recon |
|---|---|---|
| **Touches your systems?** | No | Yes |
| **Leaves logs?** | **No** | Yes |
| **Examples** | WHOIS, DNS records, certificate transparency (crt.sh), Shodan, LinkedIn, GitHub, Google dorks, breach data | Port scans, ping sweeps, vulnerability scans, directory brute forcing |

## Nmap scan types

| Scan | Flag | How it works | Log footprint |
|---|---|---|---|
| **TCP connect** | `-sT` | Full handshake on each port | Many complete connections; app logs may record them |
| **SYN (half-open)** | `-sS` | Sends SYN; on SYN-ACK replies RST, never completes | Many `S0` / `RSTO` connections, often missed by app logs |
| **UDP** | `-sU` | Sends UDP packets; ICMP "port unreachable" = closed | Many ICMP unreachable replies |
| **FIN / NULL / Xmas** | `-sF` / `-sN` / `-sX` | Odd flag combinations to sneak past simple filters | Abnormal flag combinations; IDS alerts |
| **Ping sweep** | `-sn` | Finds which hosts are alive | ICMP / ARP to every IP in a range |
| **Version / OS detection** | `-sV` / `-O` | Probes services and TCP behaviour to identify software | Unusual probe payloads to open ports |

**Shapes of a scan:**

- **Vertical:** one source → **one host, many ports**
- **Horizontal:** one source → **many hosts, one port** (looking for one vulnerable service, e.g. 445 or 3389)
- **Block:** many hosts × many ports

## Internal discovery (after a compromise)

Once inside, attackers map the environment:

- `net user /domain`, `net group "domain admins" /domain`, `nltest /dclist:`, `whoami /all`, `ipconfig /all`, `arp -a`
- **BloodHound / SharpHound**: queries AD over LDAP to find the shortest path to Domain Admin
- Port scans from a workstation (Advanced IP Scanner, Nmap, SoftPerfect)

## What you see in logs

| Log source | What to look for |
|---|---|
| Firewall | One source IP hitting **many ports** or **many hosts**; lots of denies |
| Zeek `conn.log` | Bursts of `S0`, `REJ`, `RSTO` states from one source |
| IDS | `ET SCAN` signatures (Nmap, Masscan, Nikto, etc.) |
| Web server | Thousands of **404s** from one IP (directory brute forcing); scanner User-Agents (Nikto, Nuclei, sqlmap) |
| Sysmon 1 / 4688 | Discovery commands in a burst; `nmap.exe`, `SharpHound.exe`, `AdFind.exe` |
| DC / LDAP logs | One user querying huge numbers of AD objects (BloodHound) |

## How to detect it

```spl title="Example Splunk search: vertical scan"
index=firewall
| bin _time span=1m
| stats dc(dest_port) as ports by _time, src_ip, dest_ip
| where ports > 50
```

```spl title="Example Splunk search: horizontal scan"
index=firewall
| bin _time span=5m
| stats dc(dest_ip) as hosts by _time, src_ip, dest_port
| where hosts > 50
```

## How to respond

- **External scanning of your internet-facing IPs is constant background noise.** Don't escalate every scan. Escalate when it's **followed by exploitation attempts** or targets a newly announced vulnerability on your systems.
- **Internal scanning from a normal workstation is serious**: it usually means a compromised host (or a worm). Check if it's an authorised vulnerability scanner first; if not, isolate and investigate.
- Use scan data to **fix exposure**: why is that port open to the internet?

## References

- [MITRE ATT&CK T1595](https://attack.mitre.org/techniques/T1595/)
- [MITRE ATT&CK T1046](https://attack.mitre.org/techniques/T1046/)
