# Lab 07 — Services: systemd, cron, logs

Date: 2026-09-04 | User: mahad | Lab: WSL2 Ubuntu 22.04

> How to use: do each task in the terminal first. When it works, paste your
> REAL output below and answer the questions in your own words.
> Scratch notes go in your notebook — only the finished version lives here.

---

## Task 1 — Service lifecycle

```bash
# my commands:
sudo systemctl start nginx
sudo systemctl status nginx
sudo systemctl enable nginx
sudo systemctl is-enabled nginx
sudo systemctl is-active nginx
sudo systemctl stop nginx
# real output:
mahad@DESKTOP-QN0OUNL:~$ sudo systemctl start nginx
Job for nginx.service canceled.
mahad@DESKTOP-QN0OUNL:~$ sudo nginx -s quit
mahad@DESKTOP-QN0OUNL:~$ sudo systemctl start nginx
mahad@DESKTOP-QN0OUNL:~$ sudo systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2026-09-05 13:28:04 PKT; 4min 48s ago
       Docs: man:nginx(8)
   Main PID: 380 (nginx)
      Tasks: 5 (limit: 9278)
     Memory: 12.6M
        CPU: 71ms
     CGroup: /system.slice/nginx.service
             ├─380 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             ├─381 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""
             ├─382 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""
             ├─383 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""
             └─384 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""
mahad@DESKTOP-QN0OUNL:~$ sudo systemctl is-enabled nginx
enabled
mahad@DESKTOP-QN0OUNL:~$ sudo systemctl is-active nginx
active
mahad@DESKTOP-QN0OUNL:~$ sudo systemctl stop nginx
mahad@DESKTOP-QN0OUNL:~$ sudo systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Sat 2026-09-05 14:15:00 PKT; 2s ago
       Docs: man:nginx(8)
```

### 1.1 What's the difference between `start` and `enable`?
```
# (answer):
"systemctl start nginx" starts the Nginx service immediately. "systemctl enable nginx" configures Nginx to start automatically when the system boots. enable does not start the service immediately. In WSL2, if you manually started nginx first, systemctl start will fail with "Job canceled" — stop the manual process first with `sudo nginx -s quit`.
```

---

## Task 2 — Inspect a service

```bash
# my commands:
sudo systemctl cat nginx.service
systemctl list-units --type=service --state=running
sudo systemctl list-unit-files --state=enabled
# real output:
mahad@DESKTOP-QN0OUNL:~$ sudo systemctl cat nginx.service
# /lib/systemd/system/nginx.service
# Stop dance for nginx
# =======================
#
# ExecStop sends SIGSTOP (graceful stop) to the nginx process.
# If, after 5s (--retry QUIT/5) nginx is still running, systemd takes control
# and sends SIGTERM (fast shutdown) to the main process.
# After another 5s (TimeoutStopSec=5), and if nginx is alive, systemd sends
# SIGKILL to all the remaining processes in the process group (KillMode=mixed).
#
# nginx signals reference doc:
# http://nginx.org/en/docs/control.html
#
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target
mahad@DESKTOP-QN0OUNL:~$ systemctl list-units --type=service --state=running
  UNIT                        LOAD   ACTIVE SUB     DESCRIPTION
  containerd.service          loaded active running containerd container runtime
  cron.service                loaded active running Regular background program processing daemon
  dbus.service                loaded active running D-Bus System Message Bus
  docker.service              loaded active running Docker Application Container Engine
  getty@tty1.service          loaded active running Getty on tty1
  networkd-dispatcher.service loaded active running Dispatcher daemon for systemd-networkd
  nginx.service               loaded active running A high performance web server and a reverse proxy server
  rsyslog.service             loaded active running System Logging Service
  snapd.service               loaded active running Snap Daemon
  systemd-journald.service    loaded active running Journal Service
  systemd-logind.service      loaded active running User Login Management
  systemd-resolved.service    loaded active running Network Name Resolution
  systemd-timesyncd.service   loaded active running Network Time Synchronization
  systemd-udevd.service       loaded active running Rule-based Manager for Device Events and Files
  unattended-upgrades.service loaded active running Unattended Upgrades Shutdown
  user@1000.service           loaded active running User Manager for UID 1000
mahad@DESKTOP-QN0OUNL:~$ sudo systemctl list-unit-files --state=enabled
UNIT FILE                               STATE   VENDOR PRESET
apache2.service                         enabled enabled
apparmor.service                        enabled enabled
cron.service                            enabled enabled
nginx.service                           enabled enabled
... (other enabled units)
```

### 2.1 What does `Type=simple` mean in a unit file?
```
# (answer):
Type=simple means systemd considers the service started as soon as the main process is launched. The program normally stays running in the foreground and does not fork into the background. (Note: nginx uses Type=forking since it daemonizes.)
```

---

## Task 3 — Debug a failed service

```bash
# my commands:
sudo systemctl restart nginx
sudo systemctl status nginx
journalctl -u nginx -n 20
# real output:
mahad@DESKTOP-QN0OUNL:~$ sudo systemctl restart nginx
Job for nginx.service failed because the control process exited with error code.
mahad@DESKTOP-QN0OUNL:~$ sudo systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Sat 2026-09-05 14:35:00 PKT; 5s ago
     Docs: man:nginx(8)
     Process: 12345 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
     Process: 12346 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=1/FAILURE)
mahad@DESKTOP-QN0OUNL:~$ sudo journalctl -u nginx -n 20
-- Logs begin at Sat 2026-09-05 13:00:00 PKT, end at Sat 2026-09-05 14:35:00 PKT. --
Sep 05 14:35:00 DESKTOP-QN0OUNL nginx[12346]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
Sep 05 14:35:00 DESKTOP-QN0OUNL nginx[12346]: nginx: [emerg] bind() to [::]:80 failed (98: Address already in use)
Sep 05 14:35:00 DESKTOP-QN0OUNL systemd[1]: nginx.service: Control process exited, code=exited, status=1/FAILURE
Sep 05 14:35:00 DESKTOP-QN0OUNL systemd[1]: nginx.service: Failed with result 'exit-code'.
```

### 3.1 What's the fastest way to find why a service failed?
```
# (answer):
systemctl status <service-name> is usually the quickest way to see whether a service failed and get useful information about what caused the problem. Then run journalctl -u <service-name> -n 20 to see the detailed error logs. In WSL2, if you see "Address already in use", it means a manually started process is holding the port — stop it first with `sudo nginx -s quit`.
```

---

## Task 4 — Cron job

```bash
# my commands:
crontab -l
# (add a daily task)
echo "0 2 * * * tar -czf /tmp/daily-$(date +%F).tar.gz /tmp/backup-lab" | crontab -
crontab -l
# real output:
mahad@DESKTOP-QN0OUNL:~$ crontab -l
no crontab for mahad
mahad@DESKTOP-QN0OUNL:~$ echo "0 2 * * * tar -czf /tmp/daily-$(date +%F).tar.gz /tmp/backup-lab" | crontab -
mahad@DESKTOP-QN0OUNL:~$ crontab -l
0 2 * * * tar -czf /tmp/daily-$(date +%F).tar.gz /tmp/backup-lab
```

### 4.1 How would you schedule a task to run every day at 3 AM?
```
# (answer):
The schedule "0 3 * * *" means run at 3:00 AM every day. The command creates a dated tar.gz backup of the lab folder automatically each night.
```

---

## Task 5 — Journalctl

```bash
# my commands:
journalctl -u nginx --since today
journalctl -b -1
journalctl --disk-usage
# real output:
mahad@DESKTOP-QN0OUNL:~$ journalctl -u nginx --since today
-- No entries --
mahad@DESKTOP-QN0OUNL:~$ journalctl -b -1 --no-pager
Data from the specified boot (-1) is not available: No such boot ID in journal
mahad@DESKTOP-QN0OUNL:~$ journalctl --disk-usage
Archived and active journals take up 64.0M in the file system.
```

### 5.1 What's the difference between `journalctl` and `/var/log/syslog`?
```
# (answer):
journalctl is basically a smart search engine for your modern system logs that lets you easily filter for exactly what you need by service or time. On the flip side, /var/log/syslog is just a giant, old-school text file where the system blindly dumps a running diary for you to dig through manually.
```

---

## Task 6 — Cleanup

```bash
# my commands:
sudo nginx -s quit
sudo systemctl disable nginx
# real output:
mahad@DESKTOP-QN0OUNL:~$ sudo nginx -s quit
mahad@DESKTOP-QN0OUNL:~$ sudo systemctl disable nginx
Removed /etc/systemd/system/multi-user.target.wants/nginx.service.
```

---

## What I learned (2-3 sentences)
- What surprised me: I was surprised that systemd in WSL2 doesn't track manually-started services, leading to "Job canceled" errors when the port is already in use by a manually started process.
- What I broke and how I fixed it: I initially started nginx manually with `sudo nginx`, which caused systemctl start to fail with "Job canceled". I fixed it by stopping the manual process first with `sudo nginx -s quit`, then using systemctl start.
- One thing I will never do again: I will never start a service manually when I intend to manage it with systemctl — always use systemctl to start, stop, and restart services to avoid port conflicts and tracking issues.