<sub>[Home](../README.md) › [Initial access](README.md)</sub>

# HTML smuggling

> **In one line:** An innocent-looking `.html` attachment **builds the malware file inside the browser** using JavaScript, so the email gateway and web proxy never see the malicious file pass through.

| **MITRE ATT&CK** | [T1027.006 Obfuscated Files or Information: HTML Smuggling](https://attack.mitre.org/techniques/T1027/006/) |
|---|---|
| **Tactics** | Defense Evasion, Initial Access |
| **Delivers** | ZIP / ISO / IMG containing a loader, `.lnk`, or a fake login page |
| **Known use** | Nobelium (2021), QakBot, many malware loaders |

## How it works

1. The user receives `Invoice_7731.html` as an attachment (or a link to such a page).
2. The HTML file contains the malware **encoded as a long JavaScript string** (base64, often also encrypted / split into pieces).
3. When opened, the script **decodes the string in the browser** and creates a file with `Blob` + `URL.createObjectURL`.
4. The browser "downloads" the file, but it was created **locally**: no network download for the proxy to scan, and the email gateway only saw an HTML file.
5. The user opens the dropped ZIP / ISO and runs what's inside.

```html
<script>
  const b64 = "UEsDBBQAAAAI...";          // the payload, encoded
  const bytes = Uint8Array.from(atob(b64), c => c.charCodeAt(0));
  const url = URL.createObjectURL(new Blob([bytes]));
  const a = document.createElement("a");
  a.href = url; a.download = "Invoice_7731.zip"; a.click();
</script>
```

## What you see in logs

| Log source | What to look for |
|---|---|
| Email gateway | **`.html` / `.htm` / `.shtml` / `.svg` attachments** from external senders |
| Sysmon 11 / EDR | Browser (`chrome.exe`, `msedge.exe`) writing `.zip` / `.iso` / `.img` / `.js` to `Downloads` **without a matching network download** in proxy logs |
| Sysmon 15 | Mark-of-the-Web `Zone.Identifier` showing a **`blob:`** or **`file://`** origin instead of a normal URL |
| Sysmon 1 | Then the usual chain: `explorer.exe` → mounted ISO → `.lnk` → `cmd` / `powershell` / `rundll32` |

## How to detect it

- HTML attachments containing large base64 blobs and `atob(`, `Blob(`, `createObjectURL`, `msSaveOrOpenBlob`.
- File downloads with `blob:` origins.
- Archive/disk-image files opened shortly after an HTML attachment was opened.

## How to respond

1. Purge the email from all mailboxes; get the attachment hash.
2. Open the HTML **in a sandbox** or read its source to extract the payload.
3. Check which users opened it and whether the dropped file ran (EDR); isolate those hosts.
4. **Fix:** block or quarantine HTML attachments from external senders, set `.js` / `.hta` / `.wsf` to open with Notepad, block auto-mount of ISO / IMG for normal users.

## References

- [MITRE ATT&CK T1027.006](https://attack.mitre.org/techniques/T1027/006/)

---

| [← QR code phishing (quishing)](qr-code-phishing.md) | [Home](../README.md) | [Business email compromise (BEC) →](business-email-compromise.md) |
|:---|:---:|---:|
