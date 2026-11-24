<sub>[Home](../README.md)</sub>

# Credential attacks

Attacks that **guess, steal or reuse** passwords, password hashes and Kerberos tickets. Credentials are the attacker's main goal after getting in: with a valid admin account, they look like a normal user.

## Two groups

**Guessing attacks** (create failed-logon logs, usually from outside):

| Attack | Pattern | MITRE |
|---|---|---|
| [Brute force](brute-force.md) | Many passwords → one account | T1110.001 |
| [Password spraying](password-spraying.md) | One password → many accounts | T1110.003 |
| [Credential stuffing](credential-stuffing.md) | Leaked user+password pairs → your login | T1110.004 |
| [MFA fatigue](mfa-fatigue.md) | Correct password + spam MFA pushes until the user approves | T1621 |

**Stealing and reusing** (attacker is already inside, usually with admin rights):

| Attack | What is stolen / abused | MITRE |
|---|---|---|
| [Credential dumping](credential-dumping.md) | Hashes and passwords from LSASS memory, SAM, NTDS.dit | T1003 |
| [DCSync](dcsync.md) | All domain hashes, by pretending to be a domain controller | T1003.006 |
| [Pass-the-hash](pass-the-hash.md) | NTLM hash used directly to log in | T1550.002 |
| [Pass-the-ticket](pass-the-ticket.md) | Kerberos ticket stolen and reused | T1550.003 |
| [Kerberoasting](kerberoasting.md) | Service ticket cracked offline to get a service account password | T1558.003 |
| [AS-REP roasting](as-rep-roasting.md) | Accounts without Kerberos pre-auth cracked offline | T1558.004 |
| [Golden / silver ticket](golden-silver-ticket.md) | Forged Kerberos tickets | T1558.001 / .002 |

## Kerberos in 5 steps (needed for the Kerberos attacks)

1. **AS-REQ:** the user asks the domain controller (KDC) for a **TGT** (Ticket Granting Ticket), proving their identity with a timestamp encrypted with their password hash (this proof is called **pre-authentication**).
2. **AS-REP:** the DC returns a TGT, encrypted with the **krbtgt** account's hash. → Event **4768**.
3. **TGS-REQ:** when the user wants to use a service (e.g. a file server), they send the TGT and ask for a **service ticket**.
4. **TGS-REP:** the DC returns a service ticket, encrypted with the **service account's** password hash. → Event **4769**.
5. The user gives the service ticket to the service, which decrypts it with its own key and lets the user in. → Event **4624** on the server.

Where each attack hits:

- Step 1 without pre-auth → [AS-REP roasting](as-rep-roasting.md)
- Step 4 ticket cracked offline → [Kerberoasting](kerberoasting.md)
- krbtgt hash stolen → forge step 2 → [Golden ticket](golden-silver-ticket.md)
- Service hash stolen → forge step 4 → [Silver ticket](golden-silver-ticket.md)
- Ticket stolen from memory and reused → [Pass-the-ticket](pass-the-ticket.md)

## NTLM in 3 steps

1. The client says "I want to log in as `bob`".
2. The server sends a random **challenge**.
3. The client encrypts the challenge with **bob's NT hash** and sends it back. The server (or the DC, event **4776**) checks it.

The plain-text password is never needed, only the hash. That is why a stolen hash works for [pass-the-hash](pass-the-hash.md).

---

| [← Supply chain attack](../initial-access/supply-chain.md) | [Home](../README.md) | [Brute force →](brute-force.md) |
|:---|:---:|---:|
