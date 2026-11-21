# 19 — Backup and Archiving: Practice Questions

## Level 1 — Concepts

1. Define RPO and RTO.
2. What is the 3-2-1 rule?
3. Compare full, incremental, and differential backups for restore effort.
4. Why is a same-disk snapshot not a backup?
5. Why not copy `/var/lib/mysql` while MySQL runs?
6. How does `rsync --link-dest` save space?

<details><summary>Answers</summary>

1. RPO: max acceptable data loss (time). RTO: max acceptable time to restore service.
2. 3 copies of data, on 2 different media/types, 1 copy off-site.
3. Full: one set. Incremental: full + every incremental since. Differential: full + latest differential.
4. Disk failure or account compromise destroys both data and snapshot.
5. Files change during the copy; the result is inconsistent/corrupt. Use dump tools or a frozen snapshot.
6. Unchanged files are hard links to the previous backup; only changed files use new space.
</details>

## Level 2 — Commands

1. Archive `/etc` into `etc-<date>.tar.gz` with relative paths.
2. List the content of an archive without extracting.
3. Extract only `etc/ssh/sshd_config` into `/tmp/restore`.
4. Mirror `/data` to `/mnt/backup/data` and delete removed files. Do a dry run first.
5. Stream a tar of `/srv` to a remote host over SSH.
6. Search for "ERROR" inside `app.log.3.gz` without extracting.
7. Create a disk image of `/dev/sdb` with progress.
8. Dump a PostgreSQL database in custom format.

<details><summary>Answers</summary>

1. `tar -czpf etc-$(date +%F).tar.gz -C / etc`
2. `tar -tzvf archive.tar.gz`
3. `tar -xzf etc-*.tar.gz -C /tmp/restore etc/ssh/sshd_config`
4. `rsync -aHAXn --delete /data/ /mnt/backup/data/` then without `-n`
5. `tar -czf - /srv | ssh backup 'cat > /backups/srv.tar.gz'`
6. `zgrep ERROR app.log.3.gz`
7. `dd if=/dev/sdb of=/backup/sdb.img bs=4M status=progress conv=fsync`
8. `pg_dump -Fc mydb > mydb.dump`
</details>

## Level 3 — Scenarios

1. Design backups for a single EC2 web + PostgreSQL server with RPO 1 h and RTO 4 h.
2. Ransomware encrypted the server and the backup NFS share mounted on it. What design would have prevented loss?
3. A restore test shows the tar backup lost file ownership and SELinux labels. Fix the backup command.
4. Write a cron-ready script that keeps 7 daily and 4 weekly rsync snapshots.

<details><summary>Answers</summary>

1. Hourly WAL archiving or `pg_dump`/pgBackRest to S3 (RPO), daily EBS snapshots via AWS Backup/DLM copied to another region/account, `/etc` and app config in git/AMI, documented + tested restore runbook to meet 4 h.
2. Immutable/off-host backups: S3 Object Lock, separate AWS account, pull-based backups (backup server pulls; production cannot delete), versioning, MFA delete.
3. Run as root and add `-p --acls --xattrs --selinux` (and `--numeric-owner` when restoring on another host).
4. Use `rsync -a --delete --link-dest=$LATEST src/ $DEST/$(date +%F)`, update a `latest` symlink, delete dailies older than 7 days, and copy Sunday snapshots into a `weekly/` set with 4 kept.
</details>

## Lab

1. Install restic. Back up `/etc` to a local repo. Delete a file. Restore it.
2. Build `--link-dest` snapshots for 3 days. Compare `du -sh` of each dir vs total.
3. Take an LVM snapshot with `fsfreeze`, back it up with tar, restore to a new LV.
