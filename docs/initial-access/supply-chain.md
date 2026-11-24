# Supply chain attack

!!! abstract "In one line"
    The attacker compromises **software, an update or a provider you already trust**, so the malware arrives through a trusted channel.

| | |
|---|---|
| **MITRE ATT&CK** | [T1195 Supply Chain Compromise](https://attack.mitre.org/techniques/T1195/) (.001 dependencies and dev tools, .002 software supply chain, .003 hardware), [T1199 Trusted Relationship](https://attack.mitre.org/techniques/T1199/) |
| **Tactic** | Initial Access |

## Real examples

| Name | Year | What happened |
|---|---|---|
| **SolarWinds (SUNBURST)** | 2020 | Attackers inserted a backdoor into SolarWinds Orion updates; up to 18,000 customers downloaded it, and a much smaller number were actively targeted |
| **3CX** | 2023 | The 3CX desktop phone app installer was trojanized and signed by the vendor |
| **XZ Utils** | 2024 | A backdoor (CVE-2024-3094) was added to a Linux compression library by a long-term "maintainer"; found just before wide release |
| **Malicious packages** | Ongoing | Fake or typosquatted packages on npm / PyPI (e.g. `reqeusts` instead of `requests`) that steal credentials on install |
| **MSP compromise** | Ongoing | Attacker hacks an IT service provider, then uses its remote management tools to reach all of its customers (trusted relationship) |

## How it works

1. The attacker gets into the vendor's build system, code repository, update server or a package account.
2. They add malicious code to a legitimate product.
3. The product is **signed and distributed normally**. Customers install it as a routine update.
4. The backdoor activates on customer networks, often waiting days before doing anything.

## What you see in logs

| Log source | What to look for |
|---|---|
| EDR / Sysmon 1 | **Trusted, signed software** spawning unusual child processes (`cmd`, `powershell`) |
| Sysmon 3 / DNS / proxy | Trusted software connecting to **domains it never contacted before** |
| Software inventory | New versions installed right before the strange behaviour |
| Developer machines / CI | New dependencies pulled during builds; install scripts running network commands |
| Vendor / CISA advisories | Published IOCs for affected versions |

## How to detect it

Signature-based tools trust the software, so detection relies on **behaviour**:

- Baseline what each business application normally does (child processes, network destinations) and alert on changes.
- Watch threat intel / vendor advisories and **search your environment for affected versions** as soon as one is published.

## How to respond

1. Identify **every host** running the affected version (asset inventory, EDR).
2. Isolate or remove the affected software as the vendor advises.
3. Hunt for the published IOCs and for activity **after** installation (second-stage payloads, lateral movement).
4. Reset credentials the software had access to.
5. **Fix:** software inventory / SBOM (Software Bill of Materials), restrict outbound access for servers, pin and verify dependencies, least privilege for vendor and MSP access.

## References

- [MITRE ATT&CK T1195](https://attack.mitre.org/techniques/T1195/)
