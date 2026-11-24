# Drive-by compromise

!!! abstract "In one line"
    The user **visits a website** (hacked, fake or malicious ad) and gets infected, either by an exploit or by being tricked into downloading a "fake update" or "fake installer".

| | |
|---|---|
| **MITRE ATT&CK** | [T1189 Drive-by Compromise](https://attack.mitre.org/techniques/T1189/) |
| **Tactic** | Initial Access |
| **Known families** | SocGholish (fake browser update), malvertising for fake Zoom / PuTTY / AnyDesk installers |

## Variants

| Variant | What happens |
|---|---|
| **Fake update** | A hacked legitimate website shows "Your browser is out of date. Update now." The download is a `.js` or `.zip` with malware |
| **Malvertising / SEO poisoning** | A search ad or top search result leads to a fake download site for popular software. The installer contains malware |
| **Watering hole** | Attackers hack a site that their specific target group visits (e.g. an industry news site) |
| **Exploit kit** | The page silently exploits an unpatched browser or plugin. Rarer today because browsers auto-update |

## How it works (fake update example)

1. The user browses a normal news site that has been hacked.
2. Injected JavaScript shows a full-page "Chrome update required" banner.
3. The user downloads `Chrome.Update.js` and double-clicks it.
4. Windows runs it with `wscript.exe`, which profiles the machine and downloads the next stage.
5. Often the next stage is a RAT or Cobalt Strike, leading to ransomware.

## What you see in logs

| Log source | What to look for |
|---|---|
| Proxy | Visit to a site followed by download of `.js`, `.zip`, `.msi`, `.exe` from an unusual domain |
| Sysmon 1 / EDR | `chrome.exe` / `msedge.exe` → (user opens) `wscript.exe` running a `.js` file from `Downloads` |
| Sysmon 15 | `Zone.Identifier` stream (Mark-of-the-Web) showing the file came from the internet and from which URL |
| DNS | Lookups for newly registered domains right after browsing |
| Sysmon 3 | `wscript.exe` or the new installer making outbound connections |

## How to detect it

- `wscript.exe` / `cscript.exe` executing a script from the user's `Downloads` folder.
- Installers for common tools downloaded from domains that are not the vendor's.
- Browser download followed within seconds by a new process making network connections.

```spl title="Example Splunk search"
index=sysmon EventCode=1 Image IN ("*\\wscript.exe", "*\\cscript.exe")
  CommandLine="*\\Downloads\\*"
| table _time, host, user, ParentImage, CommandLine
```

## How to respond

1. **Isolate the host.**
2. Find the **downloaded file** and the **URL** it came from (proxy + Mark-of-the-Web).
3. Block the domain(s). Search proxy logs for **other users** who visited the same site or downloaded the same file.
4. Investigate what the second stage did (persistence, C2, credential theft).
5. **Fix:** set `.js`, `.vbs`, `.hta` to open in Notepad by default, app allowlisting, ad blocking, software only from an internal portal.

## References

- [MITRE ATT&CK T1189](https://attack.mitre.org/techniques/T1189/)
