# 16 — Performance and Troubleshooting: Practice Questions

## Level 1 — Concepts

1. What does the USE method check?
2. `free` shows 300 MB free and 9 GB buff/cache. Is the host low on memory?
3. What do the `r` and `b` columns in `vmstat` mean?
4. What does exit code 137 mean for a container?
5. Why is `%util` less useful on NVMe and SSD?
6. Why should you collect data before restarting a service?

<details><summary>Answers</summary>

1. Utilization, saturation, and errors for every resource.
2. Probably not. Check `available`. Cache is freed when apps need memory.
3. `r`: runnable tasks waiting for CPU. `b`: tasks blocked on I/O (D state).
4. 128 + 9 = killed by SIGKILL, usually the OOM killer (memory limit).
5. These devices serve many requests in parallel. 100% busy time does not mean saturated. Use latency (await) and queue size.
6. A restart destroys evidence (memory state, open files, stuck threads), so the root cause stays unknown.
</details>

## Level 2 — Commands

1. Show per-CPU usage every second, 5 times.
2. Show disk latency and queue per device.
3. Show which processes do the most disk I/O.
4. Show the system calls a running process (PID 2345) makes, summarized.
5. Find which process listens on port 8080.
6. Show TCP retransmit counters.
7. Show CPU history from yesterday with sar.
8. Find deleted files that still use disk space.

<details><summary>Answers</summary>

1. `mpstat -P ALL 1 5`
2. `iostat -xz 1`
3. `iotop -oPa` or `pidstat -d 1`
4. `strace -c -p 2345` (Ctrl+C to print)
5. `ss -tlpn 'sport = :8080'` or `lsof -i :8080`
6. `nstat -az | grep -i retrans` or `sar -n ETCP 1`
7. `sar -u -f /var/log/sa/saDD` (DD = day) or `sar -u -1` on newer sysstat
8. `lsof +L1`
</details>

## Level 3 — Incidents

1. API latency jumped from 50 ms to 2 s. CPU 20%, load 25 on 8 cores, `wa` 60%. Next steps?
2. A t3.medium was fast in the morning and slow in the afternoon. CPU shows 20% busy, `st` low. What do you suspect?
3. A Java app is OOM-killed every 2 days. How do you investigate?
4. A service fails with "Too many open files" under load. Diagnose and fix.
5. A process uses 100% CPU. How do you find which code path?
6. Users report slow SSH login (10 s delay before password prompt). Common causes?

<details><summary>Answers</summary>

1. I/O bound. `iostat -xz 1` (which disk, await), `iotop` (which process), `lsof` (which files), check EBS limits/burst balance, noisy batch jobs, log flood, swap activity.
2. CPU credit exhaustion: T3 throttled to baseline (20% for t3.medium). Check `CPUCreditBalance`. Use unlimited mode or a non-burst type (m6i).
3. `dmesg` for OOM details, track RSS over time, check JVM heap vs container/cgroup limit (`-Xmx` vs `MemoryMax`), off-heap/native memory, heap dumps (`jcmd GC.heap_dump`), fix leak or right-size.
4. `cat /proc/PID/limits`, `ls /proc/PID/fd | wc -l`, check for fd leaks (`lsof -p PID` growing, CLOSE_WAIT). Raise `LimitNOFILE` in unit; fix leaks.
5. `top -H -p PID` (hot thread), `perf top -p PID` / `perf record -g`, language tools (py-spy, jstack, async-profiler), `strace -c` for syscall heavy loops.
6. Reverse DNS lookup of client (`UseDNS no`), GSSAPI auth (`GSSAPIAuthentication no`), slow PAM/LDAP/SSSD, DNS resolver timeouts.
</details>

## Lab

1. Run `stress-ng --cpu 4`, `--vm 2 --vm-bytes 80%`, `--io 4` one at a time. Watch each in `vmstat 1` and label the pattern.
2. Fill a disk with a file held open by `tail -f`. Delete it. Recover space without killing tail (truncate via `/proc/PID/fd`).
3. Trigger the OOM killer in a cgroup-limited systemd-run scope: `systemd-run --scope -p MemoryMax=100M stress-ng --vm 1 --vm-bytes 200M`.
