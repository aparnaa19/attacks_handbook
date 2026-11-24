<sub>[Home](../README.md) › [Network attacks](README.md)</sub>

# DNS tunneling

> **In one line:** The attacker hides **commands and stolen data inside DNS queries and answers**, because almost every network allows DNS out.

| **MITRE ATT&CK** | [T1071.004 Application Layer Protocol: DNS](https://attack.mitre.org/techniques/T1071/004/), [T1048 Exfiltration Over Alternative Protocol](https://attack.mitre.org/techniques/T1048/), [T1572 Protocol Tunneling](https://attack.mitre.org/techniques/T1572/) |
|---|---|
| **Tactics** | Command and Control, Exfiltration |
| **Common tools** | iodine, dnscat2, DNSExfiltrator; C2 frameworks with DNS beacons (e.g. Cobalt Strike) |

## How it works

1. The attacker registers a domain, e.g. `t.attacker.example`, and runs **their own authoritative DNS server** for it.
2. Malware on the victim encodes data into a **subdomain** and looks it up:

    ```text
    MFRGGZDFMZTWQ2LK.ONSWG4TFOQ.t.attacker.example
    ```

3. The company's DNS resolver can't answer, so it forwards the query to the internet → it reaches **the attacker's DNS server**, which reads the data from the name.
4. The attacker's server replies with commands hidden in **TXT**, **NULL**, **CNAME** or even A records.
5. Repeat thousands of times: a slow but working two-way channel that never directly connects to the attacker's IP.

## What you see in logs

| Log source | What to look for |
|---|---|
| DNS server / Zeek `dns.log` | **Very long query names** (e.g. more than 50 characters), random-looking (high-entropy) labels |
| DNS logs | **Huge number of unique subdomains** under one parent domain from one host |
| DNS logs | High volume of **TXT** or **NULL** record queries |
| DNS logs | Many **NXDOMAIN** answers, or a domain with no website |
| Firewall | Workstations sending DNS **directly to external servers** (port 53 to `8.8.8.8`) instead of internal resolvers |
| Sysmon 22 | **Which process** made the queries (should be a browser or known app, not `rundll32` or an unknown exe) |

## How to detect it

**Example Splunk search: many unique subdomains per domain**

```spl
index=dns
| rex field=query "(?<parent_domain>[^.]+\.[^.]+)$"
| eval qlen=len(query)
| stats dc(query) as unique_queries, avg(qlen) as avg_len by src_ip, parent_domain
| where unique_queries > 200 AND avg_len > 40
```

> [!NOTE]
> **DNS over HTTPS (DoH)**
>
> If malware uses DoH, its queries travel inside HTTPS to providers like Cloudflare or Google and **don't appear in your DNS logs**. Block unapproved DoH resolvers and force clients to use internal DNS.

## How to respond

1. Identify the **host and process** making the queries.
2. **Sinkhole / block** the domain on your resolvers. Isolate the host.
3. Estimate **how much data** left: count and size of queries over time.
4. Investigate how the malware got there.
5. **Fix:** only internal resolvers may send DNS to the internet; block outbound 53 from everything else; DNS filtering / Response Policy Zones; block newly registered domains.

## References

- [MITRE ATT&CK T1071.004](https://attack.mitre.org/techniques/T1071/004/)

---

| [← LLMNR / NBT-NS poisoning](llmnr-nbtns-poisoning.md) | [Home](../README.md) | [C2 beaconing →](c2-beaconing.md) |
|:---|:---:|---:|
