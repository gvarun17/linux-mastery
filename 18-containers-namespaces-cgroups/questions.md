# 18 — Namespaces and cgroups: Practice Questions

## Level 1 — Concepts

1. What are the 4 building blocks of a container?
2. Name 6 namespace types and what each isolates.
3. What happens when a container hits its CPU limit? Its memory limit?
4. How does overlayfs give each container its own writable root?
5. What is the role of runc vs containerd?
6. Why is a container not a VM?

<details><summary>Answers</summary>

1. Namespaces, cgroups, a layered root filesystem, and security controls (capabilities, seccomp, MAC).
2. pid (PIDs), net (network stack), mnt (mounts), uts (hostname), ipc (IPC objects), user (UID/GID mapping), cgroup (cgroup view).
3. CPU: throttled. Memory: OOM killed (exit 137).
4. Read-only image layers (lowerdirs) + a per-container writable upperdir merged into one view; writes use copy-on-write.
5. runc is the low-level OCI runtime that creates the process with namespaces/cgroups. containerd manages images, snapshots, and container lifecycle and calls runc via a shim.
6. All containers share the host kernel. A VM runs its own kernel on virtual hardware.
</details>

## Level 2 — Commands

1. List all namespaces on the host.
2. Run `ip addr` inside the network namespace of a container with host PID 4321.
3. Start a shell in new PID and UTS namespaces with its own `/proc`.
4. Run a script limited to 300 MB memory and half a CPU with systemd.
5. Check if a cgroup had OOM kills.
6. Check if a pod/container is CPU throttled.
7. Confirm the host uses cgroup v2.

<details><summary>Answers</summary>

1. `lsns`
2. `nsenter -t 4321 -n ip addr`
3. `unshare --pid --uts --fork --mount-proc bash`
4. `systemd-run --scope -p MemoryMax=300M -p CPUQuota=50% ./script.sh`
5. `cat /sys/fs/cgroup/<path>/memory.events` (look at `oom_kill`)
6. `cat /sys/fs/cgroup/<path>/cpu.stat` (look at `nr_throttled`, `throttled_usec`)
7. `stat -fc %T /sys/fs/cgroup` → `cgroup2fs`
</details>

## Level 3 — Scenarios

1. A pod is slow but never above 50% of its node CPU. Its limit is 500m. What do you check?
2. A pod has no `curl` or `ip` tools. How do you debug its network from the node?
3. A container restarts with exit code 137. List 2 possible causes and how to tell them apart.
4. Developers ask for `--privileged` to run a tool that needs to bind port 80. Better option?
5. Explain what happens at the kernel level when you run `docker run -m 256m nginx`.

<details><summary>Answers</summary>

1. CPU throttling: `cpu.stat` `nr_throttled`, `container_cpu_cfs_throttled_seconds_total` metric. Raise or remove the CPU limit, or tune the app (thread count).
2. Find the container PID (`crictl inspect` → pid) and `nsenter -t PID -n` to use host tools; or `kubectl debug` with an ephemeral container.
3. OOM kill (check `kubectl describe pod` → OOMKilled, `memory.events`, dmesg) vs external SIGKILL after failed graceful stop (grace period exceeded; check events/logs).
4. `--cap-add NET_BIND_SERVICE` (with `--cap-drop ALL`), or bind a high port and map it.
5. dockerd → containerd → shim → runc: pull/unpack layers into overlayfs, create namespaces via `clone()`, create a cgroup with `memory.max=256M`, apply seccomp/capabilities/AppArmor-SELinux, set up veth into the bridge network + iptables NAT, pivot_root into merged rootfs, `execve` nginx as PID 1.
</details>

## Lab

1. Build the red/blue namespace bridge from the notes. Add NAT so `red` can reach the internet.
2. Build an overlayfs mount by hand. Change and delete files and look at upper (whiteout files).
3. Run a container, find its PID on the host, read its cgroup files and `/proc/PID/ns`.
