# 02 — Filesystem Hierarchy: Practice Questions

## Level 1 — Concepts

1. Where do you find system config files? Logs? App state data?
2. What is the difference between `/tmp` and `/var/tmp`?
3. What does an inode store? What does it not store?
4. Give three differences between hard links and symlinks.
5. What do mtime and ctime track?
6. Why do `/proc` and `/sys` use no disk space?

<details><summary>Answers</summary>

1. `/etc`, `/var/log`, `/var/lib`.
2. `/tmp` is often cleared on reboot (may be tmpfs). `/var/tmp` survives reboots.
3. Metadata: owner, perms, size, times, link count, block pointers. It does not store the file name.
4. Hard links share an inode, cannot cross filesystems, cannot target dirs. Symlinks have their own inode, can cross filesystems, break when the target is deleted.
5. mtime: content change. ctime: metadata change (perms, owner, link count).
6. They are virtual filesystems. The kernel generates content in memory.
</details>

## Level 2 — Commands

1. List `/var/log` with the oldest files first and human sizes.
2. Show the inode number and all timestamps of `/etc/hosts`.
3. Show inode usage for all filesystems.
4. Find the 5 largest dirs under `/var`.
5. Create a symlink `/opt/app` pointing to `/opt/app-1.4`. Then switch it to `/opt/app-1.5`.
6. Copy `/etc/nginx` to `/backup/` and keep perms and timestamps.
7. Show the real type of a file named `data` with no extension.

<details><summary>Answers</summary>

1. `ls -ltrh /var/log`
2. `stat /etc/hosts` (or `ls -i /etc/hosts`)
3. `df -i`
4. `du -sh /var/* 2>/dev/null | sort -rh | head -5`
5. `ln -s /opt/app-1.4 /opt/app` then `ln -sfn /opt/app-1.5 /opt/app`
6. `cp -a /etc/nginx /backup/`
7. `file data`
</details>

## Level 3 — Scenarios

1. `df -h` shows `/` at 100%. You delete a 20 GB log file. `df` still shows 100%. Why? Fix it.
2. `df -h` shows 40% used, but you cannot create new files ("No space left on device"). Why?
3. A deploy script uses `rm /opt/app && ln -s new /opt/app`. Why is this risky? Improve it.
4. `du -sh /` shows 30 GB but `df` shows 50 GB used. Name two causes.

<details><summary>Answers</summary>

1. A process still holds the file open. Find it: `lsof +L1` or `lsof | grep deleted`. Restart the process or truncate via `/proc/<pid>/fd/<n>`: `: > /proc/PID/fd/N`.
2. Inodes are full (many small files). Check `df -i`. Delete small files (session dirs, cache).
3. There is a moment with no `/opt/app`. Use `ln -sfn new /opt/app` or create a temp link and `mv -T tmp /opt/app` (atomic rename).
4. Deleted-but-open files; files hidden under a mount point; reserved blocks for root (ext4 5%).
</details>

## Lab

1. Create a file, make a hard link and a symlink. Delete the original. Check which link still works.
2. Create 100 000 empty files on a small loop filesystem until inodes run out. Observe `df -i`.
3. Mount a tmpfs over a non-empty dir. Observe hidden files. Unmount and see them return.
