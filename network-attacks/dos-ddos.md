<sub>[Home](../README.md) › [Network attacks](README.md)</sub>

# DoS and DDoS

> **In one line:** The attacker **overloads a service** (bandwidth, server resources or the application) so real users can't use it. **DDoS** = from many machines at once (a botnet).

| **MITRE ATT&CK** | [T1498 Network Denial of Service](https://attack.mitre.org/techniques/T1498/) (.001 direct flood, .002 reflection amplification), [T1499 Endpoint Denial of Service](https://attack.mitre.org/techniques/T1499/) |
|---|---|
| **Tactic** | Impact |
| **Breaks** | **Availability** (the "A" in CIA) |

## Three kinds

| Kind | Target | Examples |
|---|---|---|
| **Volumetric** | Fill the internet link | UDP flood, ICMP flood, **amplification** attacks |
| **Protocol** | Exhaust connection tables in servers, firewalls, load balancers | **SYN flood**, fragmented packet attacks |
| **Application layer (L7)** | Exhaust the web app / database | HTTP GET/POST flood on expensive pages (search, login), **Slowloris**, HTTP/2 Rapid Reset (CVE-2023-44487) |

## How the main ones work

**SYN flood:** the attacker sends huge numbers of SYN packets (often from spoofed IPs) and never completes the handshake. The server keeps each half-open connection in memory waiting for the ACK, until the table is full and real users can't connect.

**Reflection / amplification:** the attacker sends small requests to public servers (DNS, NTP, memcached, CLDAP, SSDP) with the **victim's IP as the fake source**. The servers send **much larger** replies to the victim.

| Protocol | Port | Approx. amplification |
|---|---|---|
| DNS | 53 | up to ~50× or more |
| NTP (`monlist`) | 123 | ~500× |
| memcached | 11211 | up to ~50,000× |

**Slowloris:** opens many connections and sends HTTP headers very slowly, never finishing, so the web server's connection slots stay busy. Very little bandwidth needed.

## What you see in logs

| Log source | What to look for |
|---|---|
| NetFlow / firewall | Sudden traffic spike to one destination; huge packet rates |
| Zeek / firewall | Masses of `S0` connections (SYN with no completion) |
| Firewall / NetFlow | Large volumes **from** source ports 53, 123, 11211, 389, 1900 that you never requested (reflection) |
| Web server / CDN / WAF | Request rate far above normal, many IPs, same URL / User-Agent |
| Server monitoring | CPU, memory, connection count maxed; timeouts; 503 errors |

## How to detect it

- Compare traffic and request rates against a baseline.
- Uptime monitoring and user reports ("site is down") are often the first signal.

## How to respond

1. **Confirm it's an attack** and not a legitimate spike (sale, news coverage, broken client retrying).
2. Engage the **ISP / DDoS mitigation service** (Cloudflare, Akamai, AWS Shield, Azure DDoS Protection). Volumetric attacks must be absorbed upstream; your own firewall can't help once the link is full.
3. Apply filters: rate limiting, geo-blocking, blocking attack signatures (User-Agent, URL), CAPTCHA / JavaScript challenges for L7 floods.
4. **Watch for other alerts.** DDoS is sometimes a **smokescreen** to distract the SOC during a real intrusion.
5. Some attackers send a ransom demand ("pay or we continue") — escalate to management.
6. **Fix:** always-on DDoS protection / CDN for public services, SYN cookies, rate limits, don't run open resolvers or memcached on the internet.

## References

- [MITRE ATT&CK T1498](https://attack.mitre.org/techniques/T1498/)
- [CISA: Understanding and Responding to DDoS Attacks](https://www.cisa.gov/resources-tools/resources/understanding-and-responding-distributed-denial-service-attacks)

---

| [← Reconnaissance and scanning](reconnaissance-scanning.md) | [Home](../README.md) | [MITM and ARP spoofing →](mitm-arp-spoofing.md) |
|:---|:---:|---:|
