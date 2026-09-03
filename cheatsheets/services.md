# Cheatsheet — Services (systemd, cron, logs)

## systemctl
```bash
sudo systemctl start nginx          # start now
sudo systemctl stop nginx            # stop now
sudo systemctl restart nginx         # restart
sudo systemctl reload nginx          # reload config
sudo systemctl enable nginx          # start on boot
sudo systemctl disable nginx         # don't start on boot
sudo systemctl status nginx          # status + recent logs
sudo systemctl is-enabled nginx      # check enabled
sudo systemctl is-active nginx       # check running
sudo systemctl cat nginx.service     # show unit file
sudo systemctl daemon-reload         # reload unit files
sudo systemctl reset-failed nginx    # clear failed state
```

## journalctl
```bash
journalctl                          # all logs
journalctl -p err                   # errors and above
journalctl -n 50                    # last 50 lines
journalctl -f                       # follow live
journalctl --since today            # today
journalctl --since "1 hour ago"
journalctl -u nginx                 # service logs
journalctl -u nginx -p err          # only errors
journalctl -u nginx -n 50           # last 50 lines
journalctl -b                       # current boot
journalctl -b -1                    # previous boot
journalctl --disk-usage             # journal size
journalctl --vacuum-size=100M       # limit size
journalctl --vacuum-time=3days      # keep last 3 days
```

## cron
```bash
crontab -l                          # list jobs
crontab -e                          # edit
crontab -r                          # remove all
*/5 * * * * /path/to/script.sh      # every 5 min
0 2 * * * /path/to/script.sh        # daily at 2 AM
@reboot /path/to/script.sh          # at boot
```

## log files
```bash
/var/log/syslog               # general system logs
/var/log/auth.log             # authentication
/var/log/kern.log             # kernel messages
/var/log/dmesg                # boot messages
sudo tail -f /var/log/syslog  # follow live
grep "error" /var/log/syslog  # search errors
```

## Log rotation
```bash
cat /etc/logrotate.conf            # main config
ls /etc/logrotate.d/               # service configs
sudo logrotate -f /etc/logrotate.conf  # force rotation
```

## Common shortcuts
| Shortcut | Meaning | Equivalent |
|----------|---------|------------|
| @reboot | at boot | — |
| @hourly | every hour | 0 * * * * |
| @daily | midnight | 0 0 * * * |
| @weekly | Sunday midnight | 0 0 * * 0 |
| @monthly | first of month | 0 0 1 * * |
```

## Service types
- Type=simple — main process runs in foreground
- Type=forking — parent forks and exits
- Type=oneshot — runs and exits (for scripts)

## Debug flow
1. `systemctl status <service>` — what happened
2. `journalctl -u <service> -n 50` — recent logs
3. `journalctl -u <service> -p err` — only errors
4. Fix and `systemctl restart <service>`
5. `systemctl daemon-reload` after unit file changes
