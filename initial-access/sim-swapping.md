<sub>[Home](../README.md) › [Initial access](README.md)</sub>

# SIM swapping

> **In one line:** The attacker convinces (or bribes) a **mobile carrier** to move the victim's phone number to the attacker's SIM card, then receives the victim's **SMS codes and calls**, including MFA and password resets.

| **MITRE ATT&CK** | [T1451 SIM Card Swap](https://attack.mitre.org/techniques/T1451/) (Mobile matrix), followed by [T1078 Valid Accounts](https://attack.mitre.org/techniques/T1078/) |
|---|---|
| **Tactics** | Initial Access, Credential Access |
| **Known use** | Scattered Spider / 0ktapus, Lapsus$, cryptocurrency thefts |
| **Breaks** | **SMS and voice-call MFA**, SMS-based password reset |

## How it works

1. The attacker gathers the victim's personal details (name, address, last 4 of SSN, account PIN) from breaches, social media or phishing.
2. They contact the carrier pretending to be the victim ("I lost my phone"), or use a **bribed / phished carrier employee**, or port the number to a new carrier.
3. The number moves to the attacker's SIM. The victim's phone suddenly shows **"No service"**.
4. The attacker clicks **"Forgot password"** on email, bank or corporate accounts. The SMS reset codes go to them.
5. They log in, change recovery details and MFA, and lock the victim out.

## What you see in logs

| Log source | What to look for |
|---|---|
| User report | **Phone suddenly has no signal** / "SIM not provisioned", followed by account alerts |
| IdP / app logs | **Password reset via SMS** followed by login from a new device / IP |
| IdP audit logs | **MFA methods changed**: phone number replaced, new authenticator app added, other methods removed |
| IdP sign-in logs | MFA completed with **SMS** for a user who normally uses an app |
| Help desk | Calls asking to change the phone number on the account |

## How to detect it

- SMS-based reset or SMS MFA followed by MFA-method changes within minutes.
- Many accounts of the same person changing at once.
- A user calling in about losing mobile service (treat as a possible security event).

## How to respond

1. Contact the user through **another channel** (work laptop chat, manager, in person).
2. **Lock the account**, revoke sessions, restore the correct MFA methods and phone number.
3. The user must contact their **carrier** to recover the number and add a port-out PIN / SIM lock.
4. Review everything done during the takeover.
5. **Fix:** remove **SMS and voice** as MFA / reset options for corporate accounts; use authenticator apps or FIDO2; require strong identity verification for help desk resets.

## References

- [MITRE ATT&CK Mobile T1451](https://attack.mitre.org/techniques/T1451/)

---

| [← Social engineering](social-engineering.md) | [Home](../README.md) | [Drive-by compromise →](drive-by-compromise.md) |
|:---|:---:|---:|
