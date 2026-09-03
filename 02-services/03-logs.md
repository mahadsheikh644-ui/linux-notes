# Lesson 09 — Logs: Viewing and Managing

Date: 2026-09-04 | User: mahad | Lab: WSL2 Ubuntu 22.04

## Why this matters
Logs are the first place you look when something breaks. systemd's journalctl has replaced the old /var/log files for most distros. Interviewers ask: How do you find error logs? What's the difference between journalctl and /var/log? How do you rotate logs to prevent disk fill?

## Log locations
```bash
/var/log/syslog           # general system messages
/var/log/auth.log         # authentication events
/var/log/kern.log         # kernel messages
/var/log/dmesg            # boot messages
/var/log/journal/         # systemd journal files
```

## journalctl commands
```bash
journalctl                    # show all logs
journalctl -p err             # only errors and above
journalctl -n 50              # last 50 lines
journalctl -f                 # follow live (like tail -f)
journalctl --since today      # today's logs
journalctl --since "1 hour ago"
journalctl --until "10 min ago"
journalctl -u nginx           # logs for a service
journalctl -u nginx -p err    # only errors for nginx
journalctl -u nginx -n 100    # last 100 lines for nginx
journalctl -u nginx -o cat    # output without timestamps
journalctl -b                 # logs from current boot
journalctl -b -1              # logs from previous boot
journalctl --disk-usage         # how much space journal takes
journalctl --vacuum-size=100M   # limit journal size
journalctl --vacuum-time=3days  # keep only last 3 days
```

## Log levels (priority)
```
0 emerg   Emergency
1 alert   Alert
2 crit    Critical
3 err     Error
4 warning Warning
5 notice  Notice
6 info    Informational
7 debug   Debug
```

## Traditional log commands
```bash
tail -f /var/log/syslog           # follow live
grep "error" /var/log/syslog      # search for errors
dmesg | tail -20                  # last kernel messages
last                                # last logged-in users
```

## Log rotation
```bash
# Check logrotate config
cat /etc/logrotate.conf
ls /etc/logrotate.d/

# Force rotation
sudo logrotate -f /etc/logrotate.conf
```

## Common mistakes
1. Never delete log files with rm — use logrotate
2. Not checking journal disk usage (can fill /var)
3. Not setting up persistent journals (they're volatile by default)
4. Ignoring /var/log/auth.log for security events

## Make journals persistent
```bash
sudo mkdir -p /var/log/journal
sudo systemd-tmpfiles --create --prefix /var/log/journal
sudo systemctl restart systemd-journald
```

## What I learned (fill after the lab)
- (write 2-3 sentences: what surprised you, what you broke, how you fixed it)
