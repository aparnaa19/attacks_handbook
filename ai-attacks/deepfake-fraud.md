<sub>[Home](../README.md) › [AI-enabled attacks](README.md)</sub>

# Deepfake fraud

> **In one line:** The attacker uses **AI-generated voice or video** of a real executive or colleague to convince an employee to **send money, share credentials or grant access**.

| **MITRE ATT&CK** | [T1656 Impersonation](https://attack.mitre.org/techniques/T1656/), [T1566.004 Spearphishing Voice](https://attack.mitre.org/techniques/T1566/004/) |
|---|---|
| **Tactics** | Defense Evasion, Initial Access |
| **Known case** | Arup (2024): an employee in Hong Kong transferred about US$25 million after a video call in which the CFO and colleagues were all deepfakes |
| **Related** | [Business email compromise](../initial-access/business-email-compromise.md), [Social engineering](../initial-access/social-engineering.md) |

## Forms

| Form | Example |
|---|---|
| **Voice clone phone call** | "Hi, it's the CEO. I'm about to board a flight, I need you to approve this wire now." Cloned from a few seconds of public audio (earnings calls, interviews, social media) |
| **Voice note / voicemail** | Same, as a WhatsApp or Teams voice message |
| **Live video call** | Teams / Zoom call where the "executive" is a real-time face swap |
| **Fake job candidates** | Remote interview with a deepfaked face; the "hire" is a fraudulent worker who then gets corporate access |
| **Help desk** | Voice clone of an employee asking for a password or MFA reset |

## How it works

1. The attacker researches the company: who approves payments, who reports to whom, current deals (LinkedIn, press releases, breached email).
2. They collect audio/video of the executive and train a voice or face model.
3. They make contact, usually **after** a setup email ("Expect a call from me about a confidential acquisition").
4. On the call, they create **urgency and secrecy**: confidential deal, regulator deadline, "don't tell anyone yet".
5. The employee sends money, or shares access.

## What you see in logs

Deepfake calls usually leave **few technical traces**. Look at the surrounding activity:

| Log source | What to look for |
|---|---|
| Email | A setup email from a lookalike domain or new external address preceding the call |
| Teams / Zoom logs | Meeting organised by an **external** account using an executive's display name |
| Phone records | Calls from unknown numbers claiming to be executives |
| Finance systems | Urgent, unusual payments to **new beneficiaries**, especially just under approval thresholds |
| Help desk | Password / MFA reset requests by voice, followed by new-device sign-in |
| HR / recruiting | Candidates whose video looks off (lip sync, lighting, refuses to turn head or wave a hand in front of the face) |

## How to detect it

- Process controls detect it better than technology: any **payment or access request that arrives by call or video** must be verified through a **separate, known channel**.
- Flag external Teams/Zoom accounts using internal people's names.
- Watch for the classic pressure signals: urgency, secrecy, unusual channel, request to bypass process.

## How to respond

1. If money was sent: **finance contacts the bank immediately** to try to recall it (hours matter), then law enforcement.
2. Preserve call recordings, meeting logs, phone numbers and setup emails.
3. Check for related compromise: was the setup email from a hacked mailbox? See [BEC](../initial-access/business-email-compromise.md).
4. Warn staff, especially finance, HR and help desk, about the specific lure.
5. **Fix:** call-back verification to numbers on file, multi-person approval for payments, code words for high-risk requests, help desk identity verification that doesn't rely on voice, training with real deepfake examples.

## References

- [MITRE ATT&CK T1656](https://attack.mitre.org/techniques/T1656/)

---

| [← Prompt injection](prompt-injection.md) | [Home](../README.md) | [Insider threat →](../insider-threat/README.md) |
|:---|:---:|---:|
