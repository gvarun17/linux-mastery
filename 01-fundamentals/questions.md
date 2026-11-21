# 01 — Fundamentals: Practice Questions

## Level 1 — Concepts

1. What is the difference between the Linux kernel and a Linux distribution?
2. What is a system call? Give three examples.
3. Name the two main distro families used in servers and their package tools.
4. In what order does bash look up a command name?
5. What is the difference between `'$HOME'` and `"$HOME"`?
6. What does "everything is a file" mean? Give four examples.
7. Which man section holds file formats? Which holds admin commands?

<details><summary>Answers</summary>

1. The kernel manages hardware, processes, and memory. A distro is the kernel plus tools, a package manager, configs, and support.
2. A request from a user program to the kernel. Examples: `open()`, `read()`, `fork()`, `execve()`, `write()`.
3. Debian family (`apt`/`dpkg`) and Red Hat family (`dnf`/`rpm`).
4. Alias → keyword → function → builtin → `$PATH` executable.
5. Single quotes print the literal text `$HOME`. Double quotes expand it to the home path.
6. Linux exposes resources as files: disks (`/dev/sda`), processes (`/proc/1`), kernel settings (`/proc/sys`), terminals (`/dev/pts/0`).
7. Section 5 for file formats. Section 8 for admin commands.
</details>

## Level 2 — Commands

1. Show the kernel release and the distro name.
2. Find all man pages related to "partition".
3. Show how bash resolves `ls` and `cd`.
4. Create `project/{src,bin,docs}` in one command.
5. Rerun the previous command with `sudo`.
6. Print today's date as `YYYY-MM-DD` inside a sentence.
7. Make variable `REGION=ap-south-1` available to child processes.

<details><summary>Answers</summary>

1. `uname -r; cat /etc/os-release`
2. `man -k partition` or `apropos partition`
3. `type ls cd`
4. `mkdir -p project/{src,bin,docs}`
5. `sudo !!`
6. `echo "Today is $(date +%F)"`
7. `export REGION=ap-south-1`
</details>

## Level 3 — Scenarios

1. A user says `ll` works in their terminal but fails in a script. Why?
2. You added `export PATH=$PATH:/opt/tools/bin` to `~/.bashrc`. A cron job still cannot find the tool. Why?
3. `rm -rf $DIR/` ran with `$DIR` empty. What happened? How do you prevent it?
4. A file is called `-f`. How do you delete it?

<details><summary>Answers</summary>

1. `ll` is an alias. Non-interactive scripts do not load aliases. Use `ls -l` in scripts.
2. Cron does not read `~/.bashrc`. Set `PATH` in the crontab or use full paths.
3. It ran `rm -rf /`. Use `set -u` and `rm -rf "${DIR:?DIR is empty}/"`.
4. `rm -- -f` or `rm ./-f`.
</details>

## Lab

1. Open `man bash`. Search for `PROMPTING`. Change your `PS1` to show `user@host:dir $`.
2. Create an alias `k=kubectl` that survives logout.
3. Run `strace -c ls` and list the top 3 system calls.
4. Compare `/etc/os-release` on an Ubuntu and a Rocky VM.
