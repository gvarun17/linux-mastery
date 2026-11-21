# 20 — Capstone: Final Labs and Challenge Questions

## Part A — Mixed Challenge Questions

Answer without notes. Then check the relevant module.

1. A file is 644, owned by root. A normal user can delete it. How?
2. You run `chmod 000 /usr/bin/chmod` as root. How do you fix it?
3. `ls` works but `ll` and `cd` do not appear in `which`. Why?
4. Explain why `echo $((2**63))` prints a negative number.
5. A process shows `STAT = Ds`. What is it doing?
6. Why might `sudo` take 20 seconds to prompt?
7. `ping` works by IP but a service on port 443 times out. Server side shows nothing in logs. Where is traffic dropped?
8. A cron job uses `$HOME/bin/tool` and fails. Why?
9. You resized an EBS volume. `lsblk` shows 50G, `df` shows 20G. Two missing steps?
10. A systemd service works with `systemctl start` but not after reboot. Why?
11. How do you find which package installed `/usr/sbin/ss`?
12. What is the first thing you check if a K8s node is `NotReady`?

<details><summary>Answers</summary>

1. The user has `w`+`x` on the parent directory (and no sticky bit).
2. Use another tool: `setfacl -m u::rwx /usr/bin/chmod`, or `/lib64/ld-linux-x86-64.so.2 /usr/bin/chmod 755 /usr/bin/chmod`, or `install -m 755`, or reinstall coreutils (`rpm --setperms coreutils`).
3. `ll` is an alias, `cd` is a builtin. `which` only searches `$PATH`. Use `type`.
4. Bash uses signed 64-bit integers; 2^63 overflows.
5. Uninterruptible sleep (I/O wait); `s` = session leader.
6. Hostname does not resolve (sudo looks up the host). Fix `/etc/hosts`.
7. A firewall (host firewalld/ufw, SG, NACL) drops it before the app, or the app listens on another interface. Use `tcpdump` on the server to see if SYN arrives.
8. Cron env differs; `$HOME` may be set but `PATH`/working dir differ, or the job runs as another user. Use absolute paths and set env.
9. `growpart` the partition, then `xfs_growfs`/`resize2fs`.
10. It is not enabled: `systemctl enable app`. (Or a dependency like network/mount is not ready.)
11. `rpm -qf /usr/sbin/ss` or `dpkg -S /usr/sbin/ss`.
12. `kubectl describe node` conditions; on the node: `systemctl status kubelet containerd`, `journalctl -u kubelet`, disk/memory pressure, CNI status, certificates, time sync.
</details>

## Part B — Capstone Labs (Do on a VM)

### Lab 1 — Build a Production Web Server

Requirements:
1. Rocky 9 VM. Hostname `web01.lab.local`.
2. Users `alice` (admin, sudo) and `bob` (deploy, may only restart the app).
3. SSH: keys only, no root login, only group `sshusers`.
4. LVM: new disk → VG `vgapp` → LV `lvweb` 5G xfs on `/srv/web` (fstab by UUID).
5. nginx serves `/srv/web` on port 8080 with SELinux enforcing.
6. firewalld allows only 22 and 8080.
7. A Python app as a systemd service on 127.0.0.1:3000, nginx proxies `/api`.
8. logrotate for app logs, daily, keep 14.
9. systemd timer backs up `/srv/web` and `/etc` nightly to `/backup`, keep 7.
10. Persistent journal limited to 500 MB.

### Lab 2 — Break and Fix (with a partner)

Partner breaks one item at a time; you fix it within 15 minutes:

| # | Break | Skill |
|---|-------|-------|
| 1 | Wrong UUID in fstab | Emergency mode |
| 2 | `chmod 644 ~/.ssh/authorized_keys` → `666` | SSH debug |
| 3 | `mv` site files into web root (bad SELinux label) | SELinux |
| 4 | Fill `/` with a file held open | Disk / lsof |
| 5 | Remove default route | Networking |
| 6 | Typo in nginx config | Service debug |
| 7 | Bad DNS server in resolv.conf | DNS |
| 8 | Root password changed | rd.break |
| 9 | `ulimit` too low for app | Limits |
| 10 | Stop chronyd and shift clock | Time |

### Lab 3 — Kubernetes Node from Scratch (Linux side)

1. Disable swap, load `overlay` and `br_netfilter`, set sysctls.
2. Install containerd, set `SystemdCgroup = true`.
3. Install kubeadm/kubelet/kubectl, pin versions.
4. `kubeadm init`. Inspect cgroups for a pod, `nsenter` into its netns, read iptables rules from kube-proxy.

### Lab 4 — Scripting Capstone

Write `server-audit.sh` that prints a report:
- OS, kernel, uptime, load, CPU/memory usage
- Filesystems over 80%
- Top 5 processes by CPU and memory
- Listening ports with processes
- Failed systemd units
- Users with UID 0 and users with empty passwords
- SUID files not in a known allowlist
- Last 5 failed logins
- Pending security updates

Requirements: `set -Eeuo pipefail`, functions, works on Ubuntu and Rocky, passes `shellcheck`, exit code 1 if any check fails.

## Part C — Self-Assessment

Rate yourself 1–5 per module. Re-study anything under 4.

| Module | Score |
|--------|-------|
| 01 Fundamentals | |
| 02 Filesystem | |
| 03 Text processing | |
| 04 Users | |
| 05 Permissions | |
| 06 Processes | |
| 07 Packages | |
| 08 Boot/systemd | |
| 09 Storage/LVM | |
| 10 Networking | |
| 11 SSH | |
| 12 Scripting | |
| 13 Logging | |
| 14 Scheduling | |
| 15 Security | |
| 16 Performance | |
| 17 Kernel | |
| 18 Containers | |
| 19 Backup | |
