<sub>[Home](../README.md)</sub>

# AI-enabled attacks

Attacks **against** AI systems that companies now run (chatbots, copilots, AI agents), and attacks that **use** AI to make old tricks more convincing.

| Attack | What it abuses | Reference |
|---|---|---|
| [Prompt injection](prompt-injection.md) | An AI model follows instructions hidden in user input or in content it reads | OWASP LLM01, MITRE ATLAS AML.T0051 |
| [Deepfake fraud](deepfake-fraud.md) | AI-generated voice or video of a real executive asks staff to pay or share access | MITRE T1656 Impersonation |

## Frameworks for AI attacks

| Framework | What it is |
|---|---|
| [OWASP Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/) | The most common risks in apps built on large language models |
| [MITRE ATLAS](https://atlas.mitre.org/) | ATT&CK-style matrix of real attacks on AI / ML systems |

## What changes for the SOC

- **New log sources:** prompts and responses of internal AI apps, AI agent tool calls (what the agent did, with which permissions), AI gateway logs.
- **Old attacks, better quality:** phishing emails with no spelling mistakes in any language, voice cloning from a 30-second clip, fake video calls. "Does this look professional?" is no longer a useful phishing test. **Verify through a second channel** instead.

---

| [← Cloud misconfiguration and key abuse](../cloud-identity/cloud-misconfiguration.md) | [Home](../README.md) | [Prompt injection →](prompt-injection.md) |
|:---|:---:|---:|
