# 07 — Package Management: Practice Questions

## Level 1 — Concepts

1. What is the difference between `apt` and `dpkg`? Between `dnf` and `rpm`?
2. Why do repos use GPG signatures?
3. What is the difference between `apt remove` and `apt purge`?
4. What does `apt full-upgrade` do that `apt upgrade` does not?
5. Why should you pin kubelet/kubeadm versions?

<details><summary>Answers</summary>

1. `apt`/`dnf` are high level: repos and dependency resolution. `dpkg`/`rpm` install single package files and manage the local DB.
2. To prove packages come from the vendor and were not changed.
3. `remove` keeps config files. `purge` deletes them too.
4. It can remove or install packages to finish an upgrade (dependency changes).
5. Kubernetes upgrades must follow a controlled order (control plane first, one minor version at a time). Auto upgrades can break the cluster.
</details>

## Level 2 — Commands

1. Find which package owns `/usr/bin/dig` on Ubuntu and on Rocky.
2. List all files installed by `openssh-server` (both families).
3. Show available versions of `docker-ce` and which repo gives each (Ubuntu).
4. Undo the last dnf transaction.
5. Install only security updates on RHEL.
6. Prevent `kubelet` upgrades on Ubuntu and on RHEL.
7. Check which files of `nginx` were changed since install (RHEL).
8. Check if a reboot is needed after patching (both families).

<details><summary>Answers</summary>

1. `dpkg -S /usr/bin/dig` / `rpm -qf /usr/bin/dig` (or `dnf provides /usr/bin/dig`)
2. `dpkg -L openssh-server` / `rpm -ql openssh-server`
3. `apt-cache policy docker-ce`
4. `dnf history` then `dnf history undo <ID>` (or `dnf history undo last`)
5. `dnf upgrade --security`
6. `apt-mark hold kubelet` / `dnf versionlock add kubelet`
7. `rpm -V nginx`
8. `needs-restarting -r` / check `/var/run/reboot-required`
</details>

## Level 3 — Scenarios

1. `apt install` fails: "dpkg was interrupted". Fix it.
2. `dnf install` fails with a GPG key error for a new vendor repo. What do you do?
3. A patch broke an app on 1 of 20 servers. Describe a safe rollback.
4. You need the same package versions on 50 servers. How do you guarantee it?
5. `apt update` shows "NO_PUBKEY". Fix it the modern way.

<details><summary>Answers</summary>

1. `sudo dpkg --configure -a`, then `sudo apt -f install`.
2. Get the vendor key from the official site over HTTPS, verify its fingerprint, `rpm --import`, or set `gpgkey=` in the .repo. Do not set `gpgcheck=0`.
3. Remove it from the load balancer. `dnf history undo <ID>` or `apt install pkg=<oldversion>`. Or restore from snapshot/AMI. Pin the version until fixed. Find the root cause on staging.
4. Use a local mirror/snapshot repo (Pulp, Satellite, Artifactory, aptly), pin versions in config management (Ansible), or bake golden images (Packer/AMI).
5. Download the key, `gpg --dearmor` into `/etc/apt/keyrings/`, and use `signed-by=` in the sources entry. `apt-key` is deprecated.
</details>

## Lab

1. Add the Docker CE repo on Ubuntu and Rocky. Install a specific version. Pin it.
2. Build a tiny `.deb` with `dpkg-deb --build` that installs a script to `/usr/local/bin`.
3. Break a config owned by a package and detect it with `rpm -V` or `debsums`.
