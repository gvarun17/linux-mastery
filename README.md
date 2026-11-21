# Linux Mastery — From Fundamentals to Production Admin

A complete Linux system administration course. Each module has:

- `notes.md` — concepts, diagrams, commands, examples, common mistakes.
- `questions.md` — practice questions in 3 levels, hands-on labs, and answers.

The notes use a simplified technical English style (ASD-STE100 inspired): short sentences, active voice, one instruction per step.

## Learning Path

```mermaid
flowchart LR
    A[01 Fundamentals] --> B[02 Filesystem]
    B --> C[03 Files & Text]
    C --> D[04 Users]
    D --> E[05 Permissions]
    E --> F[06 Processes]
    F --> G[07 Packages]
    G --> H[08 Boot & systemd]
    H --> I[09 Storage & LVM]
    I --> J[10 Networking]
    J --> K[11 SSH]
    K --> L[12 Bash Scripting]
    L --> M[13 Logging]
    M --> N[14 Scheduling]
    N --> O[15 Security]
    O --> P[16 Performance]
    P --> Q[17 Kernel /proc /sys]
    Q --> R[18 Namespaces & cgroups]
    R --> S[19 Backup]
    S --> T[20 Capstone]
```

## Modules

| # | Module | Key topics |
|---|--------|-----------|
| 01 | [Fundamentals](01-fundamentals/notes.md) | Kernel, shell, distros, architecture, help system |
| 02 | [Filesystem Hierarchy](02-filesystem-hierarchy/notes.md) | FHS, paths, inodes, links, navigation |
| 03 | [Files & Text Processing](03-files-and-text-processing/notes.md) | find, grep, sed, awk, pipes, redirection, vim |
| 04 | [Users & Groups](04-users-and-groups/notes.md) | /etc/passwd, shadow, useradd, sudo, PAM |
| 05 | [Permissions](05-permissions/notes.md) | rwx, chmod, umask, SUID/SGID/sticky, ACLs, attributes |
| 06 | [Processes & Signals](06-processes-and-signals/notes.md) | ps, top, kill, nice, jobs, process states |
| 07 | [Package Management](07-package-management/notes.md) | apt, dnf, rpm, dpkg, repos |
| 08 | [Boot & systemd](08-boot-and-systemd/notes.md) | BIOS/UEFI, GRUB, initramfs, units, targets |
| 09 | [Storage, LVM & Filesystems](09-storage-lvm-filesystems/notes.md) | Partitions, mkfs, mount, fstab, LVM, swap, RAID |
| 10 | [Networking](10-networking/notes.md) | ip, ss, DNS, routing, NetworkManager, tcpdump |
| 11 | [SSH & Remote Access](11-ssh-remote-access/notes.md) | Keys, config, tunnels, scp, rsync |
| 12 | [Shell Scripting](12-shell-scripting/notes.md) | Variables, conditions, loops, functions, traps |
| 13 | [Logging](13-logging/notes.md) | journald, rsyslog, logrotate |
| 14 | [Scheduling](14-scheduling/notes.md) | cron, at, systemd timers |
| 15 | [Security & Hardening](15-security-hardening/notes.md) | SELinux, AppArmor, firewalld, ufw, auditd |
| 16 | [Performance & Troubleshooting](16-performance-troubleshooting/notes.md) | Load, CPU, memory, I/O, strace, USE method |
| 17 | [Kernel, /proc, /sys](17-kernel-proc-sys/notes.md) | Modules, sysctl, kernel parameters |
| 18 | [Namespaces & cgroups](18-containers-namespaces-cgroups/notes.md) | Container internals, unshare, cgroup v2 |
| 19 | [Backup & Archiving](19-backup-archiving/notes.md) | tar, gzip, rsync, dd, snapshots |
| 20 | [Capstone Scenarios](20-capstone-scenarios/notes.md) | Real incidents, interview drills, final labs |

## How to Study

1. Read `notes.md` for one module.
2. Type every command in a lab VM. Do not copy-paste.
3. Answer `questions.md` without looking at the answers.
4. Open the `<details>` blocks to check your answers.
5. Do the lab section. Break things on purpose, then fix them.

## Lab Setup

Use a disposable environment. Do not practice on production hosts.

| Option | Command / Notes |
|--------|----------------|
| Multipass (Ubuntu) | `multipass launch --name lab --disk 20G` |
| Vagrant (Rocky/Alma) | `vagrant init rockylinux/9 && vagrant up` |
| Docker (limited, no systemd) | `docker run -it --rm ubuntu:24.04 bash` |
| Cloud VM | AWS EC2 t3.micro with Amazon Linux 2023 or Ubuntu 24.04 |

Practice on **two families**: Debian/Ubuntu (`apt`) and RHEL/Rocky/Alma (`dnf`).

## Certification Mapping

| Cert | Most relevant modules |
|------|----------------------|
| RHCSA (EX200) | 02–09, 13–15 |
| LFCS | 01–14 |
| CKA (Linux side) | 06, 08, 10, 16–18 |
| Linux+ (XK0-005) | All |
