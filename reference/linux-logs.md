<sub>[Home](../README.md)</sub>

# Linux logs

## Where the logs are

| File | Distro | What's in it |
|---|---|---|
| `/var/log/auth.log` | Debian / Ubuntu | SSH logins, `sudo`, `su`, user and group changes |
| `/var/log/secure` | RHEL / CentOS / Fedora | Same as `auth.log` |
| `/var/log/syslog` | Debian / Ubuntu | General system messages, cron |
| `/var/log/messages` | RHEL family | General system messages |
| `/var/log/audit/audit.log` | Any (with `auditd`) | Detailed syscalls, file access, commands run (if rules are set) |
| `/var/log/wtmp` | Any | Successful logins (read with `last`) |
| `/var/log/btmp` | Any | Failed logins (read with `lastb`) |
| `/var/log/lastlog` | Any | Last login per user (read with `lastlog`) |
| `/var/log/cron` or `syslog` | Varies | Cron job runs |
| `/var/log/apache2/access.log`, `/var/log/nginx/access.log` | Web servers | Every HTTP request. See [Web attacks](../web-attacks/README.md) |
| `~/.bash_history` | Per user | Commands typed (attackers often delete it) |
| `journalctl` | systemd systems | Binary journal. `journalctl -u ssh` shows one service |

## Key log lines

| Line | Meaning |
|---|---|
| `Failed password for root from 203.0.113.5 port 51234 ssh2` | Wrong password for an existing user |
| `Failed password for invalid user admin from 203.0.113.5` | Username does not exist (guessing usernames) |
| `Accepted password for root from 203.0.113.5` | Successful password login |
| `Accepted publickey for deploy from 10.0.0.4` | Successful SSH key login |
| `sudo: bob : TTY=pts/0 ; PWD=/home/bob ; USER=root ; COMMAND=/bin/bash` | `bob` became root with sudo |
| `bob is not in the sudoers file` | `bob` tried sudo without permission |
| `useradd[1234]: new user: name=backdoor` | New account created ([persistence](../post-exploitation/persistence.md)) |
| `session opened for user root by (uid=0)` | Session start |

## Where Linux persistence hides

| Location | What attackers put there |
|---|---|
| `crontab -l`, `/etc/crontab`, `/etc/cron.*/` | Scheduled reverse shells |
| `~/.ssh/authorized_keys` | Attacker's SSH public key |
| `/etc/systemd/system/*.service` | Malicious service |
| `~/.bashrc`, `/etc/profile` | Command that runs at every login |
| `/etc/passwd` | New user with UID 0 (a second root) |
| `/etc/ld.so.preload` | Library loaded into every process (rootkits) |

---

| [← Sysmon event IDs](sysmon-event-ids.md) | [Home](../README.md) | [Log sources →](log-sources.md) |
|:---|:---:|---:|
