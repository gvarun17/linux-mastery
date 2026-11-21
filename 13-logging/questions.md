# 13 — Logging: Practice Questions

## Level 1 — Concepts

1. How do journald and rsyslog work together?
2. Where are auth logs on RHEL and on Ubuntu?
3. What is the difference between `copytruncate` and `create` + `postrotate`?
4. What does syslog priority 3 mean?
5. Why do you lose old boot logs on some systems?

<details><summary>Answers</summary>

1. journald collects logs from kernel, services, and syslog API. rsyslog reads from journald and writes text files or forwards to remote servers.
2. `/var/log/secure` (RHEL), `/var/log/auth.log` (Ubuntu).
3. `copytruncate` copies the file then truncates it in place (app keeps its file handle; a few lines can be lost). `create` + `postrotate` renames the file, creates a new one, and signals the app to reopen.
4. `err` (error).
5. The journal is volatile (`/run/log/journal`). Enable persistent storage.
</details>

## Level 2 — Commands

1. Show nginx logs from the last 30 minutes and follow.
2. Show only errors and worse from the current boot.
3. Show kernel messages with readable timestamps, only warnings and errors.
4. Limit the journal to 1 GB on disk permanently.
5. Send a test message with tag `deploy` and priority `local0.notice`.
6. Test a logrotate config without changing files.
7. Find OOM killer events.
8. Show the last 10 successful and failed logins.

<details><summary>Answers</summary>

1. `journalctl -u nginx --since "30 min ago" -f`
2. `journalctl -b -p err`
3. `dmesg -T --level=err,warn`
4. Set `SystemMaxUse=1G` in `/etc/systemd/journald.conf`; `systemctl restart systemd-journald`
5. `logger -p local0.notice -t deploy "test"`
6. `logrotate -d /etc/logrotate.d/<file>`
7. `journalctl -k | grep -i "out of memory"` or `dmesg -T | grep -i oom`
8. `last -n 10; sudo lastb -n 10`
</details>

## Level 3 — Scenarios

1. `/var` is at 100%. `du` shows `/var/log/journal` at 12 GB. Fix now and prevent repeat.
2. After logrotate, the app log stays empty and `app.log.1` keeps growing. Why? Fix.
3. A server rebooted unexpectedly overnight. How do you find why?
4. Send all logs from 50 servers to one central server over TCP.
5. A Java service was killed. `systemctl status` shows `status=9/KILL`. Where do you look?

<details><summary>Answers</summary>

1. `journalctl --vacuum-size=1G`; set `SystemMaxUse=1G` in journald.conf; restart journald.
2. The app keeps its open handle on the renamed file. Add `postrotate` that sends HUP/reopens, or use `copytruncate`.
3. `last -x reboot shutdown`, `journalctl -b -1 -e`, `journalctl --list-boots`, `dmesg` from previous boot, check for kernel panic, OOM, power events, hypervisor/cloud events (AWS scheduled maintenance, status checks).
4. On server: rsyslog `imtcp` on 514 + template per host. On clients: `*.* @@logserver:514` in `/etc/rsyslog.d/`. Open firewall. Consider TLS (`gtls`) and a queue for reliability.
5. `journalctl -k | grep -i oom` (OOM killer), cgroup memory limit (`MemoryMax=`), `journalctl -u svc`, and whether someone ran `kill -9`.
</details>

## Lab

1. Enable persistent journal. Reboot. Read the previous boot with `journalctl -b -1`.
2. Write a logrotate rule for a test app that writes with `logger`. Force rotate.
3. Build a 2-VM rsyslog client/server setup.
