# Key SOC terms

## Indicators

| Term | Meaning | Example |
|---|---|---|
| **IOC** (Indicator of Compromise) | Evidence that an attack **already happened** | A known malware hash, a known C2 IP |
| **IOA** (Indicator of Attack) | Behaviour that shows an attack **is happening**, regardless of the tool | Word starting PowerShell, a process reading LSASS memory |
| **TTP** (Tactics, Techniques, Procedures) | How an attacker group operates | "Gets in with phishing, moves with RDP, exfiltrates with rclone" |
| **Artifact** | Any trace left on a system | Prefetch file, registry key, log entry |

## Alert outcomes

| Outcome | Meaning |
|---|---|
| **True positive (TP)** | Alert fired and it **is** malicious |
| **False positive (FP)** | Alert fired but it is **not** malicious (for example, an admin running a legitimate script) |
| **True negative (TN)** | No alert, and nothing malicious happened |
| **False negative (FN)** | No alert, but something malicious **did** happen. This is the worst case: a missed attack |
| **Benign true positive** | The rule worked correctly (the behaviour really happened), but it was authorised, for example a pentest |

## Security basics

| Term | Meaning |
|---|---|
| **CIA triad** | **Confidentiality** (only the right people can read it), **Integrity** (nobody changed it without permission), **Availability** (it works when needed). Every attack breaks at least one |
| **Vulnerability** | A weakness, for example an unpatched bug |
| **Exploit** | Code or a method that uses a vulnerability |
| **Threat** | Someone or something that could use the vulnerability |
| **Risk** | Likelihood × impact |
| **Zero-day** | A vulnerability that has no patch available yet |
| **CVE** | Public ID for a known vulnerability, e.g. CVE-2021-44228 (Log4Shell) |
| **CVSS** | Score from 0 to 10 for how severe a CVE is |
| **Least privilege** | Give users only the access they need, nothing more |
| **Defense in depth** | Several layers of security, so one failure is not enough for the attacker |

## SOC tools and roles

| Term | Meaning |
|---|---|
| **SIEM** | Collects logs from everywhere and runs detection rules (Splunk, Microsoft Sentinel, QRadar, Elastic) |
| **EDR** | Agent on each endpoint that records process, file, registry and network activity and can isolate the host (CrowdStrike, Microsoft Defender for Endpoint, SentinelOne) |
| **XDR** | EDR plus other sources (email, identity, cloud) correlated together |
| **SOAR** | Automates response steps (block IP, disable user, open ticket) |
| **IDS / IPS** | Inspects network traffic. IDS alerts, IPS also blocks (Snort, Suricata) |
| **WAF** | Firewall for web applications. Blocks SQLi, XSS and similar requests |
| **Sandbox** | Isolated machine where a suspicious file is run safely to see what it does |
| **Threat intel** | Information about attackers, their IOCs and TTPs |
| **L1 / L2 / L3** | L1 triages alerts, L2 investigates deeper, L3 does threat hunting and complex incident response |

## Incident response phases (NIST)

1. **Preparation**: tools, logging, playbooks ready before anything happens.
2. **Detection and analysis**: find the attack and understand its scope.
3. **Containment, eradication and recovery**: stop the spread, remove the attacker, restore systems.
4. **Post-incident activity**: lessons learned, fix the root cause.

## Hashes and encoding

| Term | Meaning |
|---|---|
| **Hash** | Fixed-length fingerprint of a file (MD5, SHA1, SHA256). Same file = same hash |
| **NTLM hash** | How Windows stores a password. Can be used directly in [pass-the-hash](../credential-attacks/pass-the-hash.md) |
| **Base64** | Encoding, **not** encryption. Anyone can decode it. Attackers use it to hide commands (`powershell -enc ...`) |
| **Salt** | Random data added to a password before hashing so identical passwords give different hashes |
