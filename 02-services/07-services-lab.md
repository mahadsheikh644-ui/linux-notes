# Lab 07 — Services: systemd, cron, logs

Date: 2026-09-04 | User: mahad | Lab: WSL2 Ubuntu 22.04

> How to use: do each task in the terminal first. When it works, paste your
> REAL output below and answer the questions in your own words.
> Scratch notes go in your notebook — only the finished version lives here.

---

## Task 1 — Service lifecycle

```bash
# my commands:
# sudo systemctl start nginx
# sudo systemctl status nginx
# sudo systemctl enable nginx
# sudo systemctl is-enabled nginx
# sudo systemctl is-active nginx
# sudo systemctl stop nginx
# real output:

```

### 1.1 What's the difference between `start` and `enable`?
```
# (answer):

```

---

## Task 2 — Inspect a service

```bash
# my commands:
# sudo systemctl cat nginx.service
# systemctl list-units --type=service --state=running
# sudo systemctl list-unit-files --state=enabled
# real output:

```

### 2.1 What does `Type=simple` mean in a unit file?
```
# (answer):

```

---

## Task 3 — Debug a failed service

```bash
# my commands:
# sudo systemctl restart nginx
# sudo systemctl status nginx
# journalctl -u nginx -n 20
# real output:

```

### 3.1 What's the fastest way to find why a service failed?
```
# (answer):

```

---

## Task 4 — Cron job

```bash
# my commands:
# crontab -l
# (add a daily task)
# crontab -l
# real output:

```

### 4.1 How would you schedule a task to run every day at 3 AM?
```
# (answer):

```

---

## Task 5 — Journalctl

```bash
# my commands:
# journalctl -u nginx --since today
# journalctl -b -1
# journalctl --disk-usage
# real output:

```

### 5.1 What's the difference between `journalctl` and `/var/log/syslog`?
```
# (answer):

```

---

## Task 6 — Cleanup

```bash
# my commands:
# sudo systemctl disable nginx
# real output:

```

---

## What I learned (2-3 sentences)
- What surprised me:
- What I broke and how I fixed it:
- One thing I will never do again:
