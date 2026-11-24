# C2 beaconing

!!! abstract "In one line"
    Malware on an infected machine **regularly contacts the attacker's command-and-control (C2) server** to ask "any new commands?". These regular check-ins are called **beacons**.

| | |
|---|---|
| **MITRE ATT&CK** | [T1071 Application Layer Protocol](https://attack.mitre.org/techniques/T1071/), [T1573 Encrypted Channel](https://attack.mitre.org/techniques/T1573/), [T1571 Non-Standard Port](https://attack.mitre.org/techniques/T1571/), [T1568.002 Domain Generation Algorithms](https://attack.mitre.org/techniques/T1568/002/), [T1102 Web Service](https://attack.mitre.org/techniques/T1102/) |
| **Tactic** | Command and Control |
| **Common frameworks** | Cobalt Strike, Sliver, Brute Ratel, Havoc, Mythic, Metasploit |

## How it works

1. The implant connects out to the C2 server (outbound is usually allowed; inbound is blocked).
2. It sends a small check-in and receives any queued tasks.
3. It **sleeps** for a set interval (e.g. 60 seconds), plus random **jitter** (e.g. ±20%) to look less regular.
4. It repeats, for days or weeks.

**Hiding tricks:**

| Trick | Example |
|---|---|
| Blend into HTTPS | C2 over port 443 with a normal-looking User-Agent |
| Legit services as C2 | Commands via Telegram, Discord, Slack, GitHub, Google Drive, Dropbox |
| Domain fronting / CDNs | Traffic appears to go to a big CDN domain |
| DGA | Malware generates hundreds of new domains per day; attacker registers one |
| DNS | See [DNS tunneling](dns-tunneling.md) |
| Long sleep | Check in every few hours instead of every minute |

## What you see in logs

| Log source | What to look for |
|---|---|
| Firewall / proxy / Zeek | Connections from one host to one destination at **regular intervals** (low variation in time between them) |
| Same | **Similar request and response sizes** every time |
| Proxy / DNS | **Rare destination**: only one or two hosts in the whole company talk to it |
| DNS / threat intel | **Newly registered** domain, dynamic DNS, domain on a threat feed |
| Zeek `ssl.log` | Self-signed or unusual certificates; JA3/JA4 fingerprints matching known C2 tools |
| Proxy | Connections direct to an **IP address** rather than a domain; odd or outdated User-Agent |
| Sysmon 3 / EDR | The **process** making the connection is unusual (`rundll32`, `regsvr32`, an unsigned exe in AppData) |
| Zeek `conn.log` | Very **long-lived** connections |

## How to detect it

Look at many connections together, not one at a time:

```spl title="Example Splunk search: regular intervals to one destination"
index=proxy
| sort 0 src_ip, dest, _time
| streamstats current=f last(_time) as prev_time by src_ip, dest
| eval interval=_time - prev_time
| stats count, avg(interval) as avg_int, stdev(interval) as sd_int by src_ip, dest
| where count > 50 AND sd_int < (avg_int * 0.2)
```

Low standard deviation relative to the average interval = regular = beacon-like. Exclude known regular traffic (updates, telemetry, NTP). **RITA** (Real Intelligence Threat Analytics) scores Zeek logs for beaconing automatically.

## How to respond

1. Identify the **host and process** making the connections (Sysmon 3 / EDR).
2. **Isolate** the host; **block** the domain/IP.
3. **Search every host** for connections to the same destination, same JA3/JA4, same User-Agent.
4. Analyse the implant: persistence, what commands were run, what else the attacker touched.
5. **Fix:** egress filtering (servers shouldn't browse the internet), web proxy with TLS inspection where allowed, block newly registered and uncategorised domains.

## References

- [MITRE ATT&CK T1071](https://attack.mitre.org/techniques/T1071/)
- [RITA by Active Countermeasures](https://github.com/activecm/rita)
