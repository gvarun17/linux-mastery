# 17 — Kernel, /proc, /sys: Practice Questions

## Level 1 — Concepts

1. What is the difference between `/proc` and `/sys`?
2. How do you make a sysctl change persistent?
3. Why must `br_netfilter` be loaded for Kubernetes?
4. What does PSI (`/proc/pressure`) show?
5. Why keep an older kernel installed?

<details><summary>Answers</summary>

1. `/proc` shows processes and kernel state/tunables (`/proc/sys`). `/sys` shows the device/driver model, cgroups, and module parameters.
2. Put it in `/etc/sysctl.d/NN-name.conf` and run `sysctl --system`.
3. So bridged pod traffic passes through iptables (kube-proxy rules). The sysctl key does not exist until the module loads.
4. Percentage of time tasks stall waiting for CPU, memory, or I/O. A direct saturation metric.
5. If the new kernel fails to boot or breaks drivers, you select the old one in GRUB.
</details>

## Level 2 — Commands

1. Show the kernel boot parameters.
2. Enable IP forwarding now and permanently.
3. Load `overlay` and `br_netfilter` now and at every boot.
4. Show info and parameters of the `nf_conntrack` module.
5. Set `vm.max_map_count=262144` for OpenSearch permanently.
6. Show the default kernel on RHEL.
7. Prevent the `usb-storage` module from loading.
8. Show memory pressure stats.

<details><summary>Answers</summary>

1. `cat /proc/cmdline`
2. `sysctl -w net.ipv4.ip_forward=1` and `echo "net.ipv4.ip_forward=1" > /etc/sysctl.d/90-forward.conf`
3. `modprobe overlay br_netfilter` (or one at a time) and list both in `/etc/modules-load.d/k8s.conf`
4. `modinfo nf_conntrack`
5. `echo "vm.max_map_count=262144" > /etc/sysctl.d/99-opensearch.conf && sysctl --system`
6. `grubby --default-kernel`
7. `/etc/modprobe.d/block-usb.conf`: `blacklist usb-storage` and `install usb-storage /bin/false`
8. `cat /proc/pressure/memory`
</details>

## Level 3 — Scenarios

1. `kubeadm init` fails preflight: `/proc/sys/net/bridge/bridge-nf-call-iptables does not exist`. Fix.
2. A busy NAT gateway VM logs `nf_conntrack: table full, dropping packet`. Fix.
3. A new kernel after patching does not boot. Recover.
4. A load test shows connection resets under burst load. `ss -lnt` shows Recv-Q equal to backlog. What do you tune?

<details><summary>Answers</summary>

1. `modprobe br_netfilter`, persist in `/etc/modules-load.d/`, set the sysctl, `sysctl --system`.
2. Raise `net.netfilter.nf_conntrack_max` (and hashsize), shorten timeouts if suitable, check for connection floods.
3. Choose the old kernel in the GRUB menu, boot, set it as default (`grubby --set-default`), investigate (drivers, initramfs), remove or fix the bad kernel.
4. Listen backlog is full. Raise `net.core.somaxconn` and `net.ipv4.tcp_max_syn_backlog`, and the app's own backlog setting (e.g., nginx `listen ... backlog=`).
</details>

## Lab

1. Prepare a VM as a Kubernetes node (modules + sysctl). Verify all values.
2. Write a kernel hardening sysctl file from the notes. Apply and verify each key.
3. Enable kdump on a lab VM, trigger a crash with sysrq, and find the vmcore.
