# 15 — Security and Hardening: Practice Questions

## Level 1 — Concepts

1. What is the difference between DAC and MAC?
2. Which part of an SELinux context matters most for most policies?
3. Why does `mv` cause SELinux issues but `cp` usually does not?
4. What is the difference between `--permanent` and runtime rules in firewalld?
5. In which iptables chain do you filter traffic to a local service? Traffic routed through the host?
6. Name 5 items of a Linux hardening baseline.

<details><summary>Answers</summary>

1. DAC (discretionary): owner sets permissions (rwx). MAC (mandatory): system policy decides, even root cannot bypass (SELinux, AppArmor).
2. The type (e.g., `httpd_t`, `httpd_sys_content_t`).
3. `mv` keeps the original label. `cp` creates a new file that inherits the label of the destination dir.
4. Runtime rules apply now and vanish on reload/reboot. Permanent rules are saved but need `--reload` to apply.
5. INPUT. FORWARD.
6. Patching, SSH keys only/no root, firewall default deny, SELinux enforcing, remove unused services, secure mount options, auditd, central logs, NTP, SUID audit.
</details>

## Level 2 — Commands

1. Allow HTTPS permanently in firewalld and apply.
2. Allow Postgres (5432) only from `10.0.0.0/16` with ufw.
3. Show SELinux mode and the label of `/var/www/html`.
4. Label `/data/web` so httpd can serve it, permanently.
5. Allow httpd to listen on TCP 8443.
6. Allow nginx (as reverse proxy) to connect to backends.
7. Watch `/etc/shadow` for writes and find events later.
8. List all SUID/SGID files.

<details><summary>Answers</summary>

1. `firewall-cmd --add-service=https --permanent && firewall-cmd --reload`
2. `ufw allow from 10.0.0.0/16 to any port 5432 proto tcp`
3. `getenforce; ls -Zd /var/www/html`
4. `semanage fcontext -a -t httpd_sys_content_t "/data/web(/.*)?" && restorecon -Rv /data/web`
5. `semanage port -a -t http_port_t -p tcp 8443`
6. `setsebool -P httpd_can_network_connect on`
7. `auditctl -w /etc/shadow -p wa -k shadow; ausearch -k shadow`
8. `find / -xdev \( -perm -4000 -o -perm -2000 \) -type f 2>/dev/null`
</details>

## Level 3 — Scenarios

1. nginx returns 403 for files you moved from `/home/varun/site` to `/usr/share/nginx/html`. Perms are 644. Why?
2. nginx proxy_pass to `127.0.0.1:3000` returns 502. Logs show "Permission denied" connecting upstream. Rocky 9. Fix.
3. You enabled ufw remotely and lost SSH. How to avoid next time? How to recover now?
4. Audit asks: who changed `/etc/sudoers` last week? How would you answer, and what should be set up in advance?
5. A security scan says the server has 40 listening ports. How do you reduce attack surface?

<details><summary>Answers</summary>

1. SELinux label from home dir (`user_home_t`). Fix: `restorecon -Rv /usr/share/nginx/html`.
2. SELinux blocks network connect. `setsebool -P httpd_can_network_connect on`. Confirm with `ausearch -m AVC -ts recent`.
3. Avoid: `ufw allow OpenSSH` before `ufw enable`. Recover: cloud serial console / SSM / VM console, then `ufw allow OpenSSH` or `ufw disable`.
4. auditd watch `-w /etc/sudoers -p wa -k sudoers` (and `/etc/sudoers.d`), logs shipped off-host; then `ausearch -k sudoers -ts lastweek`. Also config management/git history.
5. `ss -tulpn` → map to services → disable/mask unused, bind internal services to localhost, firewall default deny, remove packages.
</details>

## Lab

1. On Rocky: serve a site from `/srv/site` on port 8081 with SELinux enforcing. Fix every denial the right way.
2. On Ubuntu: set ufw default deny, allow SSH and 443, test from another host.
3. Run `lynis audit system`. Fix the top 5 warnings.
