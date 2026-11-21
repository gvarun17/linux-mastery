# 03 — Files and Text Processing: Practice Questions

## Level 1 — Concepts

1. Name the three standard streams and their file descriptor numbers.
2. What is the difference between `> f 2>&1` and `2>&1 > f`?
3. What does `set -o pipefail` change?
4. Why does `uniq` need sorted input?
5. What is the difference between `find -exec ... \;` and `-exec ... +`?
6. Why does `sudo echo text > /etc/file` fail?

<details><summary>Answers</summary>

1. stdin 0, stdout 1, stderr 2.
2. The first sends both streams to `f`. The second points stderr at the terminal (old stdout), then sends only stdout to `f`.
3. The pipeline returns the exit code of the last failing command, not only the last command.
4. `uniq` only removes **adjacent** duplicate lines.
5. `\;` runs the command once per file. `+` passes many files to one command (faster).
6. The shell does the redirection as your user before `sudo` runs. Use `echo text | sudo tee /etc/file`.
</details>

## Level 2 — Commands

1. Print usernames and shells from `/etc/passwd`.
2. List users with UID ≥ 1000.
3. Find files over 200 MB under `/var` modified more than 30 days ago.
4. Count lines with "ERROR" or "FATAL" (any case) in `app.log`.
5. Show 3 lines before and after each "Out of memory" in `/var/log/messages`.
6. Replace `http://` with `https://` in all `.conf` files in `/etc/myapp`, keep backups.
7. Show the top 5 HTTP status codes in an nginx access log (field 9).
8. Delete comment lines and blank lines from `nginx.conf` output.
9. Find all world-writable files on `/`, excluding `/proc`.
10. Print lines 100 to 120 of a big file.

<details><summary>Answers</summary>

1. `awk -F: '{print $1, $7}' /etc/passwd` or `cut -d: -f1,7 /etc/passwd`
2. `awk -F: '$3>=1000 {print $1}' /etc/passwd`
3. `find /var -type f -size +200M -mtime +30`
4. `grep -ciE "error|fatal" app.log`
5. `grep -B3 -A3 "Out of memory" /var/log/messages`
6. `sed -i.bak 's|http://|https://|g' /etc/myapp/*.conf`
7. `awk '{print $9}' access.log | sort | uniq -c | sort -rn | head -5`
8. `grep -vE '^\s*(#|$)' nginx.conf`
9. `find / -xdev -type f -perm -0002 2>/dev/null` (`-xdev` skips `/proc` and other mounts)
10. `sed -n '100,120p' file`
</details>

## Level 3 — Scenarios

1. A script works on your laptop but fails on Linux with `/bin/bash^M: bad interpreter`. Why? Fix it.
2. You must find which config file under `/etc` contains the IP `10.0.4.12`. Write the command.
3. Write a one-liner that alerts if any mount is above 85% use.
4. A file list has spaces in names. `for f in $(cat list)` breaks. Fix it.
5. Extract all unique email addresses from a file.

<details><summary>Answers</summary>

1. Windows CRLF line endings. Fix: `sed -i 's/\r$//' script.sh` or `dos2unix script.sh`.
2. `grep -rlF "10.0.4.12" /etc 2>/dev/null`
3. `df -hP | awk 'NR>1 && $5+0>85 {print "ALERT", $6, $5}'`
4. `while IFS= read -r f; do echo "$f"; done < list`
5. `grep -oE '[[:alnum:]._%+-]+@[[:alnum:].-]+\.[a-z]{2,}' file | sort -u`
</details>

## Lab

1. Download a sample nginx log. Build a report: top IPs, top URLs, count of 5xx per hour.
2. Use `jq` on `aws ec2 describe-instances` output (or a sample) to list instance IDs and states.
3. Do the vim tutorial: run `vimtutor` once.
