# Lesson 07 — systemd: Service Management

Date: 2026-09-04 | User: mahad | Lab: WSL2 Ubuntu 22.04

## Why this matters
systemd is the init system on modern Linux. Managing services, understanding unit files, and debugging failures are daily admin tasks. Interviews ask: How do you enable a service on boot? What's the difference between systemctl start and enable? How do you debug a failed service?

## Unit types
- **service** — daemons (most common)
- **socket** — IPC sockets
- **timer** — scheduled jobs (replaces cron)
- **target** — grouping (like runlevels)
- **mount/automount** — filesystems

## Common commands

### Service lifecycle
```bash
sudo systemctl start nginx        # start now
sudo systemctl stop nginx         # stop now
sudo systemctl restart nginx      # restart
sudo systemctl reload nginx       # reload config (no downtime)
sudo systemctl enable nginx       # start on boot
sudo systemctl disable nginx      # don't start on boot
sudo systemctl status nginx       # show status + recent logs
```

### Inspect units
```bash
systemctl list-units --type=service --state=running
systemctl list-unit-files --state=enabled
systemctl cat nginx.service       # show unit file
systemctl show nginx.service      # show all properties
systemctl is-enabled nginx        # check if enabled
systemctl is-active nginx         # check if running
```

### Logs
```bash
journalctl -u nginx               # logs for this service
journalctl -u nginx -f            # follow live
journalctl -u nginx --since today
journalctl -u nginx -p err        # only errors
```

### Debugging
```bash
systemctl status nginx            # shows last 10 log lines + status
journalctl -u nginx -n 50         # last 50 lines
systemctl daemon-reload           # reload unit files after edits
systemctl reset-failed nginx      # clear failed state
```

## Unit file structure
```ini
[Unit]
Description=My Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/myapp
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## Common mistakes
1. Forgetting `systemctl daemon-reload` after editing unit files
2. Not checking `journalctl -u` when a service fails
3. Using `systemctl start` instead of `enable` for boot persistence
4. Not understanding `Type=simple` vs `forking` vs `oneshot`

## What I learned (fill after the lab)
- (write 2-3 sentences: what surprised you, what you broke, how you fixed it)
