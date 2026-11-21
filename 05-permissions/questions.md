# 05 — Permissions: Practice Questions

## Level 1 — Concepts

1. What does `x` mean on a directory?
2. You own a file with mode `044`. Can you read it? Why?
3. What permission do you need to delete a file?
4. What do SUID, SGID, and sticky bit do?
5. With umask `027`, what are the modes of new files and dirs?
6. What does `+` at the end of `ls -l` perms mean?
7. Why use capabilities instead of SUID root?

<details><summary>Answers</summary>

1. You can enter it and access entries by name.
2. No. You are the owner, so only owner bits (`0`) apply.
3. `w` and `x` on the parent directory.
4. SUID: run as file owner. SGID: run as file group / on dir, new files inherit group. Sticky on dir: only the file owner (or root/dir owner) can delete.
5. Files 640, dirs 750.
6. The file has an ACL.
7. Least privilege. The binary gets only one power (e.g., bind low port), not full root.
</details>

## Level 2 — Commands

1. Convert `rwxr-x---` to octal.
2. Make `deploy.sh` executable for the owner only.
3. Set dirs to 755 and files to 644 under `/var/www`.
4. Create `/srv/finance` where group `finance` members share files and new files get group `finance`.
5. Give user `auditor` read access to `/var/log/app.log` without changing owner or group.
6. Make all new files in `/srv/app` writable by group `devops` automatically.
7. List all SUID binaries on the root filesystem.
8. Make `/etc/resolv.conf` impossible to change, even by root tools.
9. Show permissions of every directory in `/srv/app/data/file.txt`.

<details><summary>Answers</summary>

1. `750`
2. `chmod u+x deploy.sh` (or `chmod 700`)
3. `find /var/www -type d -exec chmod 755 {} + ; find /var/www -type f -exec chmod 644 {} +`
4. `groupadd finance; mkdir /srv/finance; chgrp finance /srv/finance; chmod 2770 /srv/finance`
5. `setfacl -m u:auditor:r /var/log/app.log` (and `x` on parent dirs if needed)
6. `setfacl -d -m g:devops:rwX /srv/app` (plus `setfacl -m g:devops:rwX /srv/app`)
7. `find / -xdev -perm -4000 -type f 2>/dev/null`
8. `chattr +i /etc/resolv.conf`
9. `namei -l /srv/app/data/file.txt`
</details>

## Level 3 — Scenarios

1. nginx returns 403 for `/home/varun/site/index.html`. File is 644. Why?
2. Root cannot delete `/etc/app.conf`: "Operation not permitted". Why?
3. Users in `/tmp` delete each other's files. What is missing?
4. You set an ACL `u:anita:rwx` but anita only gets `r-x`. Why?
5. A Node app must listen on port 443 without root. Give two ways.

<details><summary>Answers</summary>

1. `/home/varun` is often 700. nginx user has no `x` on it. Fix: `chmod o+x /home/varun` or move the site to `/var/www`. Also check SELinux context.
2. Immutable attribute. `lsattr /etc/app.conf`, then `chattr -i`.
3. Sticky bit. `chmod 1777 /tmp`.
4. The ACL mask is `r-x`. Fix: `setfacl -m m::rwx file`.
5. `setcap cap_net_bind_service=+ep $(readlink -f $(which node))`, or systemd `AmbientCapabilities=CAP_NET_BIND_SERVICE`, or a reverse proxy on 443 forwarding to 3000.
</details>

## Lab

1. Build a 3-team shared tree (`dev`, `qa`, `ops`) with SGID and default ACLs. Test as each user.
2. Copy `/usr/bin/cat` to `/tmp/rcat`, set SUID root, read `/etc/shadow` as a normal user. Then remove it. Learn why SUID audits matter.
3. Set `umask 077` in a shell. Create files. Compare modes.
