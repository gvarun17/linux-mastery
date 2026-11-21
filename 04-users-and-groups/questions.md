# 04 — Users and Groups: Practice Questions

## Level 1 — Concepts

1. What are the 7 fields of `/etc/passwd`?
2. Why are password hashes in `/etc/shadow` and not `/etc/passwd`?
3. What does `!` at the start of a shadow hash mean?
4. What is the difference between `su -` and `sudo -i`?
5. What is the difference between `required` and `requisite` in PAM?
6. Why does `limits.conf` not affect a systemd service?

<details><summary>Answers</summary>

1. username, password placeholder, UID, GID, comment, home, shell.
2. `/etc/passwd` is world-readable. `/etc/shadow` is root-only, so hashes are safe from offline cracking by normal users.
3. The account password is locked.
4. `su -` needs the target (root) password. `sudo -i` needs your own password and a sudoers rule, and it is logged per user.
5. Both must pass. `requisite` stops at once on failure. `required` continues, then fails at the end.
6. `pam_limits` applies to PAM sessions (logins). systemd starts services without a PAM login. Use `LimitNOFILE=` in the unit.
</details>

## Level 2 — Commands

1. Create user `deploy` with home, bash shell, and supplementary group `docker`.
2. Add existing user `anita` to group `wheel` without removing her other groups.
3. Force `anita` to change her password at next login.
4. Lock user `contractor` and set the account to expire on 2026-12-31.
5. Create a system user `prometheus` with no login shell and home `/var/lib/prometheus`.
6. Allow group `devops` to restart only `nginx` without a password.
7. Show password aging info for `deploy`.
8. Look up user `varun` even if it comes from LDAP/SSSD.

<details><summary>Answers</summary>

1. `useradd -m -s /bin/bash -G docker deploy`
2. `usermod -aG wheel anita`
3. `chage -d 0 anita`
4. `usermod -L -e 2026-12-31 contractor`
5. `useradd -r -s /usr/sbin/nologin -d /var/lib/prometheus -m prometheus`
6. In `visudo -f /etc/sudoers.d/devops`: `%devops ALL=(root) NOPASSWD: /usr/bin/systemctl restart nginx`
7. `chage -l deploy`
8. `getent passwd varun`
</details>

## Level 3 — Scenarios

1. You added a user to `docker` group. They still get "permission denied" on `/var/run/docker.sock`. Why?
2. A sudoers rule lets `ops` run `/usr/bin/vim /etc/hosts` as root. What is the risk?
3. After a bad edit, sudo says "syntax error" and nobody has root. How do you recover?
4. Lock accounts after 5 failed logins for 15 minutes. Which module?
5. An app fails with "Too many open files". How do you check and raise the limit for a systemd service?

<details><summary>Answers</summary>

1. Group membership loads at login. User must log out/in or run `newgrp docker`.
2. vim can run `:!bash` and give a root shell. Use `sudoedit /etc/hosts` instead.
3. Use `pkexec visudo` if polkit works, or boot to rescue mode / single user (GRUB `rd.break` or `systemd.unit=rescue.target`), or attach the disk to another VM (cloud) and fix the file.
4. `pam_faillock` with `deny=5 unlock_time=900`.
5. Check `cat /proc/<PID>/limits`. Run `systemctl edit app`, add `[Service]` `LimitNOFILE=65535`, then `systemctl daemon-reload && systemctl restart app`.
</details>

## Lab

1. Create users `dev1`, `dev2` in group `devteam`. Create `/srv/devteam` that only that group can write.
2. Write a sudoers drop-in for a CI user. Test with `sudo -l -U ciuser`.
3. Configure `pam_pwquality` for minimum length 14. Test with `passwd`.
