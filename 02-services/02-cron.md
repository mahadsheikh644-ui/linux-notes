# Lesson 08 — Cron: Scheduled Tasks

Date: 2026-09-04 | User: mahad | Lab: WSL2 Ubuntu 22.04

## Why this matters
Cron automates repetitive tasks — backups, cleanup, reports. systemd timers are the modern replacement, but cron is still everywhere. Interviewers ask: How do you schedule a daily task? What's the difference between cron and systemd timers? How do you troubleshoot a missed cron job?

## Crontab basics
```bash
crontab -l                    # list current jobs
crontab -e                    # edit crontab
crontab -r                    # remove all jobs
```

## Cron format
```
* * * * * command_to_run
│ │ │ │ │
│ │ │ │ └─── day of week (0-7, 0 and 7 are Sunday)
│ │ │ └───── month (1-12)
│ │ └─────── day of month (1-31)
│ └───────── hour (0-23)
└─────────── minute (0-59)
```

## Examples
```bash
# Run every day at 2 AM
0 2 * * * /path/to/script.sh

# Run every Monday at 3:30 AM
30 3 * * 1 /path/to/script.sh

# Run every 5 minutes
*/5 * * * * /path/to/script.sh

# Run at boot
@reboot /path/to/script.sh
```

## Useful shortcuts
| Shortcut | Meaning | Equivalent |
|----------|---------|------------|
| @reboot | at boot | — |
| @yearly | once a year | 0 0 1 1 * |
| @annually | same as @yearly | 0 0 1 1 * |
| @monthly | first of month | 0 0 1 * * |
| @weekly | Sunday at midnight | 0 0 * * 0 |
| @daily | midnight every day | 0 0 * * * |
| @midnight | same as @daily | 0 0 * * * |
| @hourly | every hour | 0 * * * * |

## Logging
```bash
# Redirect output to a log file
0 2 * * * /path/to/script.sh >> /var/log/mytask.log 2>&1

# View cron logs
grep CRON /var/log/syslog
journalctl -u cron.service
```

## systemd timers (modern alternative)
```bash
# Create a timer unit
[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target

# List timers
systemctl list-timers
systemctl list-timers --all
```

## Common mistakes
1. Using absolute paths in cron commands (PATH is limited)
2. Forgetting to make scripts executable
3. Not redirecting output (cron mails you by default)
4. Not specifying the shell interpreter in scripts

## What I learned (fill after the lab)
- (write 2-3 sentences: what surprised you, what you broke, how you fixed it)
