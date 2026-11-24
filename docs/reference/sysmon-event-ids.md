# Sysmon event IDs

**Sysmon** (System Monitor) is a free Microsoft Sysinternals tool. It logs much more detail than the default Windows logs: full command lines, parent processes, hashes, network connections per process. Logs go to `Microsoft-Windows-Sysmon/Operational`.

| ID | Event | Why it matters |
|---|---|---|
| **1** | Process creation | Command line, parent process, hash. **The most useful event**. Detects most execution |
| **2** | File creation time changed | Timestomping (hiding when a file was created) |
| **3** | Network connection | Which **process** connected to which IP and port. [C2](../network-attacks/c2-beaconing.md), lateral movement |
| **5** | Process terminated | |
| **6** | Driver loaded | Malicious / vulnerable drivers (used to kill EDR) |
| **7** | Image (DLL) loaded | DLL side-loading, suspicious DLLs |
| **8** | CreateRemoteThread | One process injecting code into another |
| **10** | Process access | One process opening another. **LSASS access = [credential dumping](../credential-attacks/credential-dumping.md)** |
| **11** | File created | Dropped malware, ransom notes |
| **12 / 13 / 14** | Registry key created-deleted / value set / renamed | Run keys and other [persistence](../post-exploitation/persistence.md) |
| **15** | File stream created | Alternate data streams, Mark-of-the-Web (downloaded files) |
| **17 / 18** | Named pipe created / connected | Cobalt Strike and other C2 frameworks use named pipes |
| **19 / 20 / 21** | WMI filter / consumer / binding | WMI event subscription persistence |
| **22** | DNS query | Which **process** looked up which domain. [DNS tunneling](../network-attacks/dns-tunneling.md), C2 |
| **23** | File delete (archived) | Keeps a copy of deleted files |
| **25** | Process tampering | Process hollowing / herpaderping |
| **26** | File delete logged | Deletion recorded without archiving |

## Most useful patterns

| Pattern | Sysmon event | Likely meaning |
|---|---|---|
| `winword.exe` / `excel.exe` → `powershell.exe` or `cmd.exe` | 1 | Malicious Office macro ([phishing](../initial-access/phishing.md)) |
| Any process opens `lsass.exe` with access `0x1010` or `0x1410` | 10 | Mimikatz-style credential dumping |
| `rundll32.exe` / `regsvr32.exe` / `mshta.exe` making network connections | 3 | [LOLBin](../malware/lolbins.md) abuse |
| Value set in `...\CurrentVersion\Run` | 13 | Persistence |
| `powershell.exe` with `-enc` / `-EncodedCommand` | 1 | Hidden command ([PowerShell abuse](../malware/powershell-abuse.md)) |
| `vssadmin.exe delete shadows` | 1 | [Ransomware](../malware/ransomware.md) deleting backups |

!!! tip "Config"
    Sysmon logs nothing useful without a config file. Common starting points are the **SwiftOnSecurity** sysmon-config and **Olaf Hartong's sysmon-modular**.
