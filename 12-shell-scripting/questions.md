# 12 — Shell Scripting: Practice Questions

## Level 1 — Concepts

1. What do `set -e`, `set -u`, and `set -o pipefail` do?
2. What is the difference between `"$@"` and `$*`?
3. Why should you quote variables?
4. What is the difference between `[ ]` and `[[ ]]`?
5. What does `trap cleanup EXIT` do?
6. Why does `cat file | while read l; do count=$((count+1)); done; echo $count` print 0?

<details><summary>Answers</summary>

1. Exit on error; error on unset variable; fail a pipeline if any command fails.
2. `"$@"` keeps each arg as a separate word. `$*` joins them into one string (or splits unquoted).
3. To stop word splitting and glob expansion on values with spaces or `*`.
4. `[[ ]]` is a bash keyword: no word splitting, supports `&&`, `||`, `=~`, and patterns. `[ ]` is the POSIX test command.
5. It runs the `cleanup` function when the script exits, for any reason (normal, error, or most signals).
6. The pipe runs the loop in a subshell. Changes to `count` are lost. Use `done < file`.
</details>

## Level 2 — Write the Code

1. Print "usage" and exit 2 if fewer than 2 args are given.
2. Get the file name without dir and without `.tar.gz` from `/backup/db-2026-11-21.tar.gz`.
3. Loop over `hosts.txt` and print hosts that do not answer ping.
4. Exit with error if `$DB_URL` is not set.
5. Retry `curl -f http://localhost/health` 5 times with 3 s sleep.
6. Stop a script if another copy is running.
7. Check if the script runs as root.

<details><summary>Answers</summary>

1. `(( $# >= 2 )) || { echo "usage: $0 a b"; exit 2; }`
2. `f=/backup/db-2026-11-21.tar.gz; b="${f##*/}"; echo "${b%.tar.gz}"`
3. `while IFS= read -r h; do ping -c1 -W1 "$h" >/dev/null || echo "$h down"; done < hosts.txt`
4. `: "${DB_URL:?DB_URL is required}"`
5. `for i in {1..5}; do curl -fsS http://localhost/health && break; sleep 3; done`
6. `exec 9>/var/lock/x.lock; flock -n 9 || exit 1`
7. `[[ $EUID -eq 0 ]] || { echo "need root"; exit 1; }`
</details>

## Level 3 — Mini Projects

Write each as a full script with `set -Eeuo pipefail`, logging, and usage. Run `shellcheck`.

1. **disk-alert.sh** — Check all mounts. Print a warning for each over a threshold (default 80, set by `-t`). Exit 1 if any are over.
2. **backup.sh** — Tar + gzip a dir to `/backup/<name>-<date>.tar.gz`. Keep the newest 7. Use `flock`.
3. **user-bulk.sh** — Read `users.csv` (`username,group,shell`). Create missing groups and users. Skip existing users.
4. **health-check.sh** — Read URLs from a file. Check each in parallel. Print status code and time. Summarize failures.
5. **log-report.sh** — For an nginx log, print top 10 IPs, top 10 URLs, and count of 4xx/5xx.

<details><summary>Sample solution: disk-alert.sh</summary>

```bash
#!/usr/bin/env bash
set -Eeuo pipefail
threshold=80
while getopts ":t:h" o; do
  case "$o" in
    t) threshold="$OPTARG" ;;
    *) echo "usage: $0 [-t percent]"; exit 2 ;;
  esac
done
over=0
while read -r use mount; do
  use="${use%\%}"
  if (( use > threshold )); then
    echo "WARN: $mount at ${use}%"
    over=1
  fi
done < <(df -P -x tmpfs -x devtmpfs | awk 'NR>1 {print $5, $6}')
exit "$over"
```
</details>

<details><summary>Sample solution: backup.sh</summary>

```bash
#!/usr/bin/env bash
set -Eeuo pipefail
src="${1:?usage: $0 <dir>}"
dest=/backup
name="$(basename "$src")"
exec 9>"/var/lock/backup-$name.lock"
flock -n 9 || { echo "already running"; exit 1; }
mkdir -p "$dest"
out="$dest/$name-$(date +%F_%H%M).tar.gz"
tar -czf "$out" -C "$(dirname "$src")" "$name"
echo "created $out"
ls -1t "$dest/$name-"*.tar.gz | tail -n +8 | xargs -r rm -f
```
</details>
