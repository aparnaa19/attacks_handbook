# Brute force

!!! abstract "In one line"
    The attacker tries many passwords against **one account** until one works.

| | |
|---|---|
| **MITRE ATT&CK** | [T1110.001 Password Guessing](https://attack.mitre.org/techniques/T1110/001/) (parent: [T1110 Brute Force](https://attack.mitre.org/techniques/T1110/)) |
| **Tactic** | Credential Access |
| **Common targets** | RDP (3389), SSH (22), VPN portals, Microsoft 365 / Entra ID sign-in, web login pages |
| **Common tools** | Hydra, Medusa, NetExec (formerly CrackMapExec), Burp Intruder |

## How it works

1. The attacker picks a target account, for example `administrator` or `root`.
2. They find a login service exposed to them, such as RDP or SSH open to the internet.
3. A tool sends login attempts with passwords from a wordlist (for example `rockyou.txt`), very fast.
4. Every wrong password creates a **failed logon** event.
5. If one password is right, the log shows a **successful logon** from the same source, usually right after the failures.

!!! note "Online vs offline"
    The attack above is **online**: every guess hits a live login service, so it leaves logs.
    **Offline** cracking ([T1110.002](https://attack.mitre.org/techniques/T1110/002/)) happens after the attacker has stolen password hashes. They guess on their own machine with Hashcat or John the Ripper, so **no logon logs are created**. You detect the hash theft instead, see [Credential dumping](credential-dumping.md).

## What you see in logs

| Log source | What to look for |
|---|---|
| Windows Security log | Many **4625** (failed logon) for one `TargetUserName` from one `IpAddress`, then a **4624** (successful logon) |
| Windows 4625 sub-status | `0xC000006A` = wrong password (the user exists). `0xC0000064` = the user does not exist |
| Windows lockout | **4740** account locked out, after the lockout threshold is reached |
| Domain controller | **4771** Kerberos pre-auth failed (failure code `0x18` = bad password), **4776** NTLM validation failures |
| Linux `auth.log` / `secure` | Repeated `Failed password for root from <IP>` then `Accepted password for root from <IP>` |
| Firewall / VPN | Many connections to 22 / 3389 / VPN from one source IP in a short time |
| Entra ID sign-in logs | Many failures with error `50126` (invalid username or password) for one user |

**Logon type matters** on 4624/4625: type **10** = RDP, type **3** = network (SMB, etc.), type **2** = someone at the keyboard.

## How to detect it

- **Many failures, one user, one source, short time.** For example, more than 20 failures for the same account from the same IP within 5 minutes.
- **Failure followed by success** from the same source. This is the high-priority case: the attacker got in.
- **Lockouts** (4740) on privileged accounts.
- **External IPs** hitting RDP or SSH directly. These services should not be internet-facing.

```spl title="Example Splunk search (field names depend on your add-on)"
index=wineventlog EventCode=4625
| bin _time span=5m
| stats count by _time, src_ip, user
| where count > 20
```

```spl title="Failures followed by a success from the same IP"
index=wineventlog (EventCode=4625 OR EventCode=4624)
| stats count(eval(EventCode=4625)) as failures,
        count(eval(EventCode=4624)) as successes by src_ip, user
| where failures > 20 AND successes > 0
```

## How to respond

1. **Check if any attempt succeeded.** Search for 4624 / `Accepted password` from the attacking IP. If yes, treat the account as compromised.
2. **Block the source IP** at the firewall or VPN.
3. If compromised: **reset the password**, revoke sessions, and review what the account did after the successful logon (processes started, files touched, other hosts it logged in to).
4. **Fix the cause:** take RDP/SSH off the internet (put it behind a VPN), enable MFA, set an account lockout policy, use SSH keys instead of passwords.

## Don't confuse with

| Attack | Pattern |
|---|---|
| **Brute force** | **Many passwords → one account** |
| [Password spraying](password-spraying.md) | **One password → many accounts** |
| [Credential stuffing](credential-stuffing.md) | **Leaked username + password pairs** from other breaches, one try each |

## References

- [MITRE ATT&CK T1110 Brute Force](https://attack.mitre.org/techniques/T1110/)
- [Microsoft: Event 4625 audit failure reference](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625)
