# 14 — Scheduling: Practice Questions

## Level 1 — Concepts

1. What are the 5 crontab time fields?
2. What is the difference between `crontab -e` and `/etc/cron.d/` file formats?
3. Why do scripts often fail in cron but work in a terminal?
4. What does `Persistent=true` do in a timer?
5. What problem does anacron solve?

<details><summary>Answers</summary>

1. minute, hour, day of month, month, day of week.
2. `/etc/cron.d/` and `/etc/crontab` have an extra **user** field before the command.
3. Minimal environment: short `PATH`, no shell rc files, different working dir, no TTY, `%` handling.
4. If the system was off at the scheduled time, the job runs at next boot.
5. Runs periodic jobs missed while the machine was powered off.
</details>

## Level 2 — Write the Schedule

1. Every 15 minutes.
2. 23:45 on weekdays.
3. 04:00 on the 1st and 15th of each month.
4. Every 2 hours between 08:00 and 20:00.
5. A system cron file that runs `/opt/cleanup.sh` as user `app` daily at 03:30.
6. A one-time job that restarts nginx in 20 minutes.
7. A systemd timer that runs `report.service` every weekday at 09:00 with up to 5 minutes random delay.

<details><summary>Answers</summary>

1. `*/15 * * * *`
2. `45 23 * * 1-5`
3. `0 4 1,15 * *`
4. `0 8-20/2 * * *`
5. `/etc/cron.d/cleanup`: `30 3 * * * app /opt/cleanup.sh >> /var/log/cleanup.log 2>&1`
6. `echo "systemctl restart nginx" | at now + 20 minutes` or `systemd-run --on-active=20m systemctl restart nginx`
7. `[Timer]` `OnCalendar=Mon..Fri 09:00` `RandomizedDelaySec=5min` `Persistent=true`, `[Install] WantedBy=timers.target`
</details>

## Level 3 — Scenarios

1. A cron job runs `date +%F` in a filename and the job breaks. Why?
2. A sync job every 5 min sometimes runs longer than 5 min and corrupts data. Fix.
3. Convert `0 2 * * * /usr/local/bin/backup.sh` into a systemd timer with logs and a 2 GB memory limit.
4. How do you verify that a cron job actually ran yesterday?

<details><summary>Answers</summary>

1. `%` is a newline in crontab. Escape: `date +\%F`.
2. `*/5 * * * * flock -n /var/lock/sync.lock /opt/sync.sh`, or use a systemd timer (it will not start the service if already running).
3. `backup.service` (`Type=oneshot`, `ExecStart=/usr/local/bin/backup.sh`, `MemoryMax=2G`) + `backup.timer` (`OnCalendar=*-*-* 02:00`, `Persistent=true`), enable the timer, read `journalctl -u backup`.
4. Check cron logs (`/var/log/cron`, `journalctl -u crond --since yesterday`), the job's own log, and output artifacts (timestamps).
</details>

## Lab

1. Create a cron job that appends `date` to a file every minute. Watch it. Remove it.
2. Build the backup timer from the notes. Run `systemctl list-timers`.
3. Use `systemd-analyze calendar` to test 5 expressions.
