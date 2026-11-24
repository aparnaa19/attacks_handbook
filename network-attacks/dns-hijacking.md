<sub>[Home](../README.md) › [Network attacks](README.md)</sub>

# DNS hijacking and DNS spoofing

> **In one line:** The attacker makes a domain name **resolve to the attacker's IP**, so users who type the correct address still land on the attacker's server.

| **MITRE ATT&CK** | [T1584.002 Compromise Infrastructure: DNS Server](https://attack.mitre.org/techniques/T1584/002/), [T1584.001 Domains](https://attack.mitre.org/techniques/T1584/001/), [T1557 Adversary-in-the-Middle](https://attack.mitre.org/techniques/T1557/) |
|---|---|
| **Tactics** | Resource Development, Credential Access, Collection |
| **Known use** | Sea Turtle and DNSpionage campaigns (changed government and telecom DNS records), router DNS-changer malware |

## Ways to do it

| Method | Where the change happens | Scope |
|---|---|---|
| **Registrar / DNS account takeover** | Attacker logs in to the domain's registrar or DNS provider (phished password, no MFA) and changes records | **Everyone** on the internet |
| **Compromised DNS server** | Attacker edits zone records on the company's DNS server | Everyone using that server |
| **DNS cache poisoning** | Fake answers injected into a resolver's cache | Users of that resolver |
| **Local DNS spoofing** | On the same LAN, attacker answers DNS queries first (after [ARP spoofing](mitm-arp-spoofing.md) or a rogue DHCP server) | Local network |
| **Router / endpoint change** | Malware changes DNS settings on a home router or PC to a rogue resolver | That router's or host's users |
| **Hosts file** | `C:\Windows\System32\drivers\etc\hosts` or `/etc/hosts` edited | One host |
| **Dangling DNS / subdomain takeover** | A `CNAME` still points to a deleted cloud resource; attacker claims that resource name | That subdomain |

## How it works (registrar takeover example)

1. The attacker phishes the credentials of the account managing `company.com` at the registrar.
2. They change the **MX** or **A** records (or the name servers) to point to their servers.
3. They get a valid **TLS certificate** for the domain (they now control DNS, so validation passes).
4. Users visiting `mail.company.com` reach the attacker's proxy, which steals credentials and forwards traffic to the real server.

## What you see in logs

| Log source | What to look for |
|---|---|
| Registrar / DNS provider audit logs | Logins from new IPs; changes to **NS, A, MX** records; contact email changes |
| Certificate transparency | **New certificates** for your domains that you didn't request |
| DNS monitoring | Your domain resolving to unexpected IPs from external resolvers |
| Sysmon 11 / EDR | Writes to the **hosts** file |
| Endpoint / router config | DNS servers changed to unknown IPs |
| Cloud DNS | `CNAME` records pointing to resources (S3, Azure, Heroku) that no longer exist |

## How to detect it

- Monitor your public DNS records and alert on any change.
- Monitor certificate transparency logs for your domains.
- Alert on hosts-file modification and on DNS server changes on endpoints.

## How to respond

1. **Revert the DNS records** and lock the registrar account (reset password, enable MFA, revoke API keys).
2. **Revoke certificates** issued to the attacker.
3. Assume credentials sent to the hijacked service were stolen: reset them, revoke sessions.
4. **Fix:** MFA + **registry lock** on registrar accounts, DNSSEC, CAA records (limit which CAs may issue certificates), clean up dangling DNS records.

## References

- [MITRE ATT&CK T1584.002](https://attack.mitre.org/techniques/T1584/002/)

---

| [← LLMNR / NBT-NS poisoning](llmnr-nbtns-poisoning.md) | [Home](../README.md) | [DNS tunneling →](dns-tunneling.md) |
|:---|:---:|---:|
