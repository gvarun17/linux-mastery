# 11 — SSH: Practice Questions

## Level 1 — Concepts

1. What is the difference between a host key and a user key?
2. Which permissions must `~/.ssh` and `authorized_keys` have?
3. Why is `ProxyJump` safer than agent forwarding?
4. What is the difference between `-L` and `-R`?
5. What does a trailing slash on the rsync source change?

<details><summary>Answers</summary>

1. Host key proves the server identity to the client. User key proves the user identity to the server.
2. `~/.ssh` 700, `authorized_keys` 600.
3. With agent forwarding, root on the jump host can use your agent socket. ProxyJump keeps keys and agent on your laptop; the bastion only forwards TCP.
4. `-L` opens a port on the client that forwards to a target via the server. `-R` opens a port on the server that forwards back to the client side.
5. `src/` copies the contents of src. `src` copies the dir itself into the destination.
</details>

## Level 2 — Commands

1. Create an ed25519 key and install it on `web01`.
2. Write a `~/.ssh/config` entry so `ssh db-01` goes through `bastion`.
3. Open local port 5433 to RDS `mydb.xyz.rds.amazonaws.com:5432` via bastion, in background.
4. Disable root login and password authentication. Test and apply.
5. Copy `/var/log/nginx/` from `web01` to local, keep perms, show progress.
6. Remove an old host key for `10.0.0.5`.
7. Run `df -h` on 3 hosts in a loop.

<details><summary>Answers</summary>

1. `ssh-keygen -t ed25519 && ssh-copy-id web01`
2. `Host db-01` / `HostName 10.0.3.4` / `User ubuntu` / `ProxyJump bastion`
3. `ssh -fN -L 5433:mydb.xyz.rds.amazonaws.com:5432 bastion`
4. Set `PermitRootLogin no` and `PasswordAuthentication no` in `/etc/ssh/sshd_config.d/10-hardening.conf`; `sshd -t && systemctl reload sshd`
5. `rsync -avz --progress web01:/var/log/nginx/ ./nginx-logs/`
6. `ssh-keygen -R 10.0.0.5`
7. `for h in web01 web02 web03; do ssh "$h" df -h; done`
</details>

## Level 3 — Scenarios

1. Key login fails, password works. List what you check on the server.
2. You changed sshd to port 2222 on Rocky. Restart works but connections fail. Why?
3. After rebuilding a server, SSH warns "REMOTE HOST IDENTIFICATION HAS CHANGED". Correct response?
4. A teammate left. How do you revoke access across 100 servers?
5. Security team wants no inbound port 22 on AWS. How do engineers still get shell access?

<details><summary>Answers</summary>

1. `journalctl -u sshd` / `/var/log/secure`; perms on home, `~/.ssh`, `authorized_keys`; ownership; key in the right user's file; `PubkeyAuthentication yes`; `AllowUsers/AllowGroups`; SELinux context (`restorecon -Rv ~/.ssh`).
2. SELinux does not allow sshd on 2222 (`semanage port -a -t ssh_port_t -p tcp 2222`) and firewalld/SG does not allow the port.
3. Confirm the rebuild with the owner. Compare the new fingerprint via a trusted channel (console output). Then `ssh-keygen -R host`.
4. Remove their key from config management (Ansible) and redeploy `authorized_keys`; disable their account / IdP; better: use SSH certificates with short TTL or SSM/IdP-based access so revocation is central.
5. AWS SSM Session Manager (agent + IAM), or EC2 Instance Connect Endpoint.
</details>

## Lab

1. Build bastion + private VM. Configure ProxyJump. Tunnel to a private Postgres.
2. Install fail2ban. Trigger a ban with wrong logins from another host.
3. Set up an SSH CA: sign a user key with `ssh-keygen -s` and trust it with `TrustedUserCAKeys`.
