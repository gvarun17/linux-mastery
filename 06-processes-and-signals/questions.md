# 06 — Processes and Signals: Practice Questions

## Level 1 — Concepts

1. What happens during `fork()` and `execve()`?
2. What is a zombie? What is an orphan?
3. What does state `D` mean? Why can you not kill it?
4. Which signals cannot be caught or ignored?
5. What does load average 8.0 mean on a 4-CPU host?
6. What does `st` in top mean in a cloud VM?

<details><summary>Answers</summary>

1. `fork()` copies the parent into a new child process. `execve()` replaces the child's memory with a new program.
2. Zombie: finished process whose parent has not read its exit status. Orphan: running process whose parent died; PID 1 adopts it.
3. Uninterruptible sleep, usually waiting on disk or NFS. The kernel does not deliver signals until the I/O returns.
4. SIGKILL (9) and SIGSTOP (19).
5. On average 8 tasks want CPU or wait on I/O. That is 2× capacity, so there is a queue.
6. Steal time: CPU time the hypervisor gave to other VMs.
</details>

## Level 2 — Commands

1. Show the top 10 processes by memory.
2. Show the process tree with PIDs.
3. Send a reload signal to nginx master.
4. Stop all processes matching `celery worker`.
5. Start a backup in the background with lowest CPU and I/O priority.
6. List open files of PID 4321.
7. Show the full command line of PID 4321.
8. Run a script that survives logout and logs output.

<details><summary>Answers</summary>

1. `ps aux --sort=-%mem | head -11`
2. `pstree -p` or `ps -ef --forest`
3. `kill -HUP $(cat /run/nginx.pid)` or `nginx -s reload` or `systemctl reload nginx`
4. `pkill -f "celery worker"`
5. `nice -n 19 ionice -c3 ./backup.sh &`
6. `lsof -p 4321` or `ls -l /proc/4321/fd`
7. `tr '\0' ' ' < /proc/4321/cmdline` or `ps -o cmd -p 4321`
8. `nohup ./script.sh > script.log 2>&1 &`
</details>

## Level 3 — Scenarios

1. `ps` shows 500 zombie processes of an app. How do you clear them?
2. Load is 30 on 8 CPUs but CPU idle is 70%. What do you check?
3. `kill -9` does not remove a process. Name two reasons.
4. A container takes 30 seconds to stop every time. Why?
5. You ran a long job in SSH without tmux. You must log out. Save the job.

<details><summary>Answers</summary>

1. Find the parent: `ps -o ppid= -p <zombie>`. Signal it with SIGCHLD or restart it. If the parent dies, PID 1 reaps the zombies.
2. Many tasks in D state waiting on I/O. Check `vmstat 1` (`b` column, `wa`), `iostat -x 1`, `ps -eo stat,pid,cmd | grep ^D`, NFS mounts.
3. It is a zombie (already dead), or it is in D state (stuck in kernel I/O).
4. The app runs as PID 1 in the container and ignores SIGTERM (no handler, or shell wrapper not forwarding signals). After the grace period, SIGKILL is sent. Use `exec` in entrypoint, `tini`, or handle SIGTERM.
5. `Ctrl+Z`, `bg`, `disown -h %1`. Then log out.
</details>

## Lab

1. Write a C or Python program that creates a zombie. Observe it in `ps`. Kill the parent.
2. Write a bash script with `trap 'echo got TERM; exit' TERM`. Send SIGTERM and SIGKILL. Compare.
3. Run `stress --cpu 4` and observe load and `us` in top.
