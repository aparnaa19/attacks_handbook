# SOC Attack Handbook

Theory reference for the attacks a SOC analyst needs to know.

**Read it here: https://aparnaa19.github.io/attacks_handbook/**

Each attack page covers:

- what the attack is and how it works, step by step
- what it looks like in logs (Windows event IDs, Sysmon, Linux, network, cloud)
- how to detect it, with an example Splunk search
- how to respond
- which attacks it is often confused with

## Sections

| Section | Pages |
|---|---|
| Basics | Attack lifecycle (Kill Chain, MITRE ATT&CK), key SOC terms |
| Quick reference | Symptom → attack lookup, Windows and Sysmon event IDs, Linux logs, log sources, common ports |
| Initial access | Phishing, BEC, social engineering, drive-by, exploited public-facing apps, supply chain |
| Credential attacks | Brute force, password spraying, credential stuffing, MFA fatigue, credential dumping, DCSync, pass-the-hash, pass-the-ticket, Kerberoasting, AS-REP roasting, golden/silver tickets |
| Malware | Ransomware, trojans and RATs, worms, infostealers, fileless malware, PowerShell abuse, LOLBins |
| Web attacks | SQL injection, XSS, command injection, path traversal, SSRF, CSRF, IDOR, file upload and web shells |
| Network attacks | Scanning, DoS/DDoS, MITM and ARP spoofing, LLMNR/NBT-NS poisoning, DNS tunneling, C2 beaconing |
| Post-exploitation | Privilege escalation, persistence, lateral movement, defense evasion, data exfiltration |
| Cloud and identity | Session token theft (AiTM), OAuth consent phishing, cloud misconfiguration |
| Insider threat | Malicious, negligent and compromised insiders |

## Run it locally

The site is built with [Zensical](https://zensical.org/).

```bash
pip install zensical
zensical serve
```

Then open http://localhost:8000.

Pages are plain Markdown in `docs/`. The sidebar order is set in `zensical.toml`. Every push to `main` rebuilds and publishes the site through GitHub Actions (`.github/workflows/docs.yml`).
