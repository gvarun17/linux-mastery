# 08 — Boot and systemd: Practice Questions

## Level 1 — Concepts

1. List the boot stages from power-on to login.
2. What is the job of the initramfs?
3. What is the difference between `Wants=` and `Requires=`? What does `After=` do?
4. Where do you put custom unit files? Why not in `/usr/lib/systemd/system`?
5. What is the difference between `disable` and `mask`?
6. Which service Type do you use for a script that runs and exits?

<details><summary>Answers</summary>

1. Firmware (BIOS/UEFI) → GRUB → kernel + initramfs → mount real root → systemd → targets/services → login.
2. Load drivers and tools (storage, LVM, RAID, LUKS) needed to find and mount the real root filesystem.
3. `Requires=` is hard: failure of the dependency stops this unit. `Wants=` is soft. `After=` only sets start order.
4. `/etc/systemd/system/`. Package updates overwrite `/usr/lib/systemd/system`.
5. `disable` removes boot-time start links. `mask` links the unit to `/dev/null`, so nothing can start it.
6. `Type=oneshot`.
</details>

## Level 2 — Commands

1. List failed services.
2. Show the full unit file of `sshd`, including overrides.
3. Raise `LimitNOFILE` for nginx without editing the vendor file.
4. Set the system to boot into text mode.
5. Show the 10 slowest units at boot.
6. Show errors from the previous boot.
7. Add `console=ttyS0` to all kernels on RHEL.
8. Rebuild the initramfs on RHEL and Ubuntu.

<details><summary>Answers</summary>

1. `systemctl --failed`
2. `systemctl cat sshd`
3. `systemctl edit nginx` → `[Service]` `LimitNOFILE=65535` → `systemctl daemon-reload && systemctl restart nginx`
4. `systemctl set-default multi-user.target`
5. `systemd-analyze blame | head`
6. `journalctl -b -1 -p err` (needs persistent journal)
7. `grubby --update-kernel=ALL --args="console=ttyS0"`
8. `dracut -f` / `update-initramfs -u`
</details>

## Level 3 — Scenarios

1. A server drops to emergency mode after reboot. What is the most common cause? How do you fix it?
2. You lost the root password on Rocky 9. Reset it.
3. Your app starts before the network has an IP and fails. Fix the unit.
4. A service restarts in a loop and systemd stops it: "start request repeated too quickly". Explain and fix.
5. Write a unit that runs `/opt/app/server` as user `app`, restarts on failure, and starts at boot.

<details><summary>Answers</summary>

1. Bad `/etc/fstab` entry (wrong UUID, missing disk). Log in with root password, `mount -o remount,rw /`, fix fstab (add `nofail` for non-critical), `mount -a`, `systemctl daemon-reload`, reboot. Check `journalctl -xb`.
2. GRUB `e`, add `rd.break`, Ctrl+X, `mount -o remount,rw /sysroot`, `chroot /sysroot`, `passwd`, `touch /.autorelabel`, exit twice.
3. Add `Wants=network-online.target` and `After=network-online.target`. Enable `NetworkManager-wait-online` or `systemd-networkd-wait-online`.
4. It hit `StartLimitBurst` within `StartLimitIntervalSec`. Find the root cause with `journalctl -u x`. Then `systemctl reset-failed x`. Optionally tune `RestartSec=`.
5. See notes section 6: `[Service] User=app ExecStart=/opt/app/server Restart=on-failure` and `[Install] WantedBy=multi-user.target`; then `daemon-reload` and `enable --now`.
</details>

## Lab

1. Write a service unit for a Python HTTP server (`python3 -m http.server 8000`). Add hardening options. Check with `systemd-analyze security myapp`.
2. Break `/etc/fstab` on a VM. Recover from emergency mode.
3. Reset the root password with `rd.break`.
4. Create a `.path` unit that runs a script when a file in `/srv/inbox` changes.
